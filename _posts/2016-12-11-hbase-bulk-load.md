---
type: post
blog: true
layout: post
title: "Bulk Loading Data to HBase"
---

We are evaluating storage product for storing and serving HathiTrust digital library's corpus these days and one of the candidates is HBase. We are considering HBase mainly because it comes with built-in Hadoop MapReduce and Spark support. HathiTrust corpus consists of digitized (OCRed) books, journals and various other historical documents from libraries all over the world which we call volumes. The full corpus contains about 14 million volumes, and we keep them in a shared Lustre cluster. Our plan is to move the corpus to our cluster to support large-scale analysis and direct downloads.  

To understand the performance behaviors of HBase we loaded about 40,000 volumes from our corpus into HBase and performed some random read tests. We used a three node cluster and [HDP](http://hortonworks.com/products/data-center/hdp/) 2.5. Each node has a single Core-i7 processor, 8GB of RAM and a 500GB HDD. Cluster like this is not suitable for a production HBase deployment and we didn't have any other options because we didn't receive our hardware yet. Since we had to pack an HDFS cluster, a Zookeeper cluster, and an HBase cluster into three nodes, HBase performance was poor. We used HBase's bulk load feature, and I am going to discuss the MapReduce-based bulk loading process in the rest of the document. 

Apache [HBase](https://hbase.apache.org) is a non-relational database modeled after Google's [BigTable](http://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf) and uses HDFS for storage layer. One of the interesting properties of HBase is the ability to bulk load data. Since we already have our data and we will only see a small number of writes periodically, this is a handy feature for our use case. There are multiple ways to do this and HBase provide several CLI tools such as TSV bulk loader to facilitate this process. In addition to the built-in tools, you can use a MapReduce application to bulk load data as well. In this approach, MapReduce outputs [HFile](http://blog.cloudera.com/blog/2012/06/hbase-io-hfile-input-output/)s which is the internal storage format of HBase, and you can use ```org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles``` tools to load generated HFiles into an HBase table.

We start by creating a table in HBase with a single split. If you know your row key distribution, you can [pre-split](http://hortonworks.com/blog/apache-hbase-region-splitting-and-merging/) your data. I did the splitting manually via the HBase UI. We didn't load raw data to HDFS for bulk load. Mappers read data directly from the local filesystem. This was possible since our data is on a network filesystem. All we had to do is mount it to the nodes that YARN node managers run and make it accessible to Hadoop user.

HBase provides reducers to use with bulk load MapReduce application and calling ```HFileOutputFormat2.configureIncrementalLoad(job, table, regionLocator)``` in your MapReduce driver code will automatically configure the reducer for you. Your mapper can output two different types of values. You can output ```org.apache.hadoop.hbase.KeyValue``` objects or ```org.apache.hadoop.hbase.client.Put``` objects encapsulating a whole row. We use the latter. Mapper code for loading digitized volumes is shown below. Please note that digitized volumes were stored in the filesystem in special directory hierarchy called [PairTree](https://wiki.ucop.edu/display/Curation/PairTree).

```java
package edu.indiana.d2i.htrc.corpusmgt.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.KeyValue;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Enumeration;
import java.util.Random;
import java.util.zip.ZipEntry;
import java.util.zip.ZipFile;

public class BulkLoadMapper extends Mapper<LongWritable, Text, ImmutableBytesWritable, Put> {

  byte[] buf = new byte[1024];

  @Override
  protected void setup(Context context) throws IOException, InterruptedException {
    Configuration conf = context.getConfiguration();
  }

  @Override
  protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
    String[] volIdPathPair = value.toString().split("\\s*,\\s*");
    String volId = volIdPathPair[0];
    volId = volId.substring(1, volId.length() - 1);
    String volPath = volIdPathPair[1];
    volPath = volPath.substring(1, volPath.length() - 1);
    if(new File(volPath).isFile()) {
      Put p = new Put(volId.getBytes());
      p.addColumn(Constants.COMPRESSED_VOLUME_CF.getBytes(), Constants.ZIP.getBytes(), readVolumeContent(volPath));

      ZipFile zip = new ZipFile(volPath);

      Enumeration entries = zip.entries();
      while (entries.hasMoreElements()) {
        ZipEntry entry = (ZipEntry) entries.nextElement();
        if (!entry.isDirectory()) {
          InputStream in = zip.getInputStream(entry);
          ByteArrayOutputStream bo = new ByteArrayOutputStream();
          int n;

          while ((n = in.read(buf, 0, 1024)) > -1) {
            bo.write(buf, 0, n);
          }

          p.addColumn(Constants.PAGES_CF.getBytes(), entry.getName().getBytes(), bo.toByteArray());
          bo.close();
          in.close();
        }
      }

      context.write(new ImmutableBytesWritable(volId.getBytes()), p);
    } else {
      System.out.println(volPath + " is a directory.");
    }
  }

  private byte[] readVolumeContent(String volPath) throws IOException {
    return Files.readAllBytes(Paths.get(volPath));
  }
}
```

MapReduce driver code is shown below. 

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
import org.apache.hadoop.hbase.mapreduce.HFileOutputFormat2;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.NLineInputFormat;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class BulkLoadDriver {
  private static final Logger log = LoggerFactory.getLogger(BulkLoadDriver.class);

  public static void submitBulkLoadJob(String hbaseTable, String input, String output, String hadoopConfDir,
                                       String hadoopUser) throws Exception {
    System.setProperty("HADOOP_USER_NAME", hadoopUser);
    System.setProperty("mapreduce.job.maps", "2");
    Configuration conf = Utils.createHBaseMapRedConfiguration(hadoopConfDir);
    conf.set("hbase.fs.tmp.dir", "/tmp");

    Job job = Job.getInstance(conf, "HBase Bulk Importer for HTRC");
    job.setJarByClass(BulkLoadMapper.class);
    job.setMapperClass(BulkLoadMapper.class);
    job.setMapOutputKeyClass(ImmutableBytesWritable.class);
    job.setMapOutputValueClass(Put.class);

    job.setInputFormatClass(NLineInputFormat.class);

    job.getConfiguration().setInt("mapreduce.input.lineinputformat.linespermap", 100);


    Path tmpPath = new Path(output);
    Connection hbCon = ConnectionFactory.createConnection(conf);
    Table hTable = hbCon.getTable(TableName.valueOf(hbaseTable));
    RegionLocator regionLocator = hbCon.getRegionLocator(TableName.valueOf(hbaseTable));
    Admin admin = hbCon.getAdmin();

    try {
      HFileOutputFormat2.configureIncrementalLoad(job, hTable, regionLocator);
      FileInputFormat.addInputPath(job, new Path(input));
      HFileOutputFormat2.setOutputPath(job, tmpPath);
      job.waitForCompletion(true);
    } finally {
      hTable.close();
      regionLocator.close();
      admin.close();
      hbCon.close();
    }
  }
}
```

Most of the sample code in the internet including [this](https://sreejithrpillai.wordpress.com/2015/01/08/bulkloading-data-into-hbase-table-using-mapreduce/) uses ```LoadIncrementalHFiles.doBulkLoad``` after the original MapReduce job to load the HFiles created into the HBase table. Somehow that didn't work with HDP 2.5, and I used the command line version of the same tool with ```hbase``` tool to load the HFiles into HBase as follows:

```bash
hbase org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles <bulk_load_mapreduce_output_path> <table_name>
```

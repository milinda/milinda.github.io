---
type: post
layout: post
blog: true
title: Latex Tips and Tricks
categories: tech
tags: latex
---

## Horizontal Rule Above/Below Figure Caption

Source: [tex.stackexchange.com](http://tex.stackexchange.com/questions/14968/horizontal-line-below-figure-caption)

You can use [caption](http://www.ctan.org/pkg/caption) packge if you want to draw a horizontal rule above or below a figure caption. ```\DeclareCaptionFormat``` can be used to create a custom caption format with the horizontal rule and ```\DeclareCaptionLabelFormat``` can be used to customize the caption label format. Finally ```\captionsetup``` can be used to tell the Latex system to use newly defined caption format and label format.

```tex
\usepackage{caption}
\DeclareCaptionFormat{myformat}{\hrulefill\\#1#2#3}
\DeclareCaptionLabelFormat{bf-parens}{\textbf{#1~#2}}
\captionsetup[figure]{labelformat=bf-parens,format=myformat}
```

If you want the horizontal rule below the caption, all you have to do is move the ```\hrulefill``` to the end and remove ```\\``` like ```{#1#2#3\hrulefill}```. In caption format, **#1** get replaced with the label, **#2** get replaced with the separator and **#3** get separated with the caption text. In caption label format, **#1** get replaced with the name (e.g. Figure) and **#2** get replavced with reference number.

## Text With Background Color 

[framed](https://www.ctan.org/pkg/framed?lang=en) package can be used to create a colored box with text.

```tex
% In preamble
\usepackage{framed}
\definecolor{shadecolor}{RGB}{216,229,229}

% In document
\begin{snugshade*}
Text goes here.
\end{snugshade*}
```

## Figure Word Wrapping

[wrapfig](https://www.ctan.org/pkg/wrapfig?lang=en) can be used to have figures wrap text around them.

```tex
\begin{wrapfigure}{r}{0.5\textwidth}
  \begin{center}
    \includegraphics[keepaspectratio=true,width=0.4\textwidth]{fig-file}
  \end{center}
  \caption{Caption Goes Here}	
  \label{fig:1}
\end{wrapfigure}
```

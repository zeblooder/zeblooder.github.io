---
title: 在latex项目里用plotneuralnet
date: 2022-03-17 11:48:26
tags: [latex,工具]
---
plotneuralnet是一个绘制神经网络的工具，使用该工具可以将python脚本转换为latex脚本，编译得到pdf。需要以下几个步骤：

1. 下载plotneuralnet的github源码，https://github.com/HarisIqbal88/PlotNeuralNet
2. 找一个latex环境，可以是win/linux/overleaf，前两种需要安装latex的环境，比如texlive，可以参考https://github.com/luanshiyinyang/PlotNeuralNet，texlive在这里可以找到https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/Images/。
3. 写好脚本`my_arch.py`（名字可以随便起）放在源码的pyexamples文件夹下，编写的教程可以在其他地方搜到。
4. 直接用`python`运行`python my_arch.py`，可以看到目录下的`my_arch.tex`

这样就得到了一个能够直接编译的tex文件。但是实际上很多情况下我们想把这个放进自己的项目里。我以一个比较大的tex项目为例，文件夹结构大致如下：

<details>
  <summary>查看文件夹结构</summary>
  <pre><code>  
.
├── Makefile
├── README.md
├── bibtex-style
│   ├── gbt7714-2005.bst
│   ├── thesis.bst
│   └── thesis2.bst
├── code
│   └── demo.cpp
├── ctex-fontset-adobe2.def
├── docs
│   ├── abstract.tex
│   ├── ack.tex
│   ├── appendix1.tex
│   ├── chap01.tex
│   ├── chap02.tex
│   ├── chap03.tex
│   ├── chap04.tex
│   ├── chap05.tex
│   ├── chap06.tex
│   ├── disclaim.tex
│   ├── grading.tex
│   ├── info.tex
│   ├── progress.tex
│   └── proposal.tex
├── fonts
│   └── init_fonts.sh
├── gulpfile.js
├── image
│   ├── appendix1
│   ├── chap03
│   │   ├── overleaf-config.jpg
│   │   ├── overleaf-create-proj.jpg
│   │   ├── overleaf-example.jpg
│   │   ├── overleaf-upload-proj.jpg
│   │   └── vscode-example.png
│   ├── chap04
│   │   ├── example
│   │   │   ├── 2007_000799.jpg
│   │   └── result
│   │       ├── compare
│   │       │   └── zoom_dog.png
│   │       └── error
│   │           └── p_2008_001580.png
│   └── template
│       ├── readme.md
│       └── logo.pdf
├── main.bib
├── main.pdf
├── main.tex
├── package.json
├── packages
│   ├── algorithm2e.sty
│   ├── algorithm2e.tex
│   └── ctex-xecjk-adobefonts.def
│   └── thubeamer.sty
├── code.sty
└── thesis.cls
  </code></pre>
</details>

实际上只需要关心`docs`文件夹和`main.tex`，因为神经网络结构一般放在正文里。

以放入`docs`的`chap03.tex`为例，需要以下步骤：

1. 将源码的layers文件夹放进自己的项目文件夹。
2. 把`my_arch.tex`中这几行的内容（里面的配置可以调整）放入main.tex（最外层的文档）的序言（preamble）区，也就是放\usepackage的区域里。

```latex
\documentclass[border=8pt, multi, tikz]{standalone} 
\usepackage{import}
\subimport{../layers/}{init}
```

3. **调整subimport里面的layers的路径，按照第一步来做，则应该改成`\subimport{layers/}{init}`**。
4. 接下来将神经网络结构放入正文，把`my_arch.tex`里除上面这几行之外的内容完整地拷贝到`chap03.tex`里自己想放的位置。**要去掉`\begin{document}`和`\end{document}`。**
5. 如果代码里有`includegraphics`，要把对应的图片放在对应的文件夹下。
6. 编译就可以看到结果。

这样做会看到神经网络图片非常大，超出了页面。因为这个模块是`tikzpicture`的，所以可以用`TikZ`的方法来调整。

以整体缩放为例，参考[在 LaTeX 中同步缩放 TikZ 与其中的 node](https://liam.page/2017/07/23/global-scale-for-TikZ-picture/)，可以在`\begin{tikzpicture}`上面加入下面这段代码：

```latex
\tikzset{global scale/.style={
    scale=#1,
    every node/.append style={scale=#1}
  }
}
```

然后把`\begin{tikzpicture}`改成`\begin{tikzpicture}[scale=0.5]`，就能把整个大小缩放为原来的0.5倍。

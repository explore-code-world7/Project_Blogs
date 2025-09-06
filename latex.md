# paper structure content

## abstract

## background

## method

## experiment

## conclusion

# 矩阵/vector

```latex
\begin{equation}
\begin{pmatrix}
a_{11} & a_{12} & a_{13} \\
a_{21} & a_{22} & a_{23} \\
a_{31} & a_{32} & a_{33}
\end{pmatrix}
\begin{pmatrix}
x_1 \\
x_2 \\
x_3
\end{pmatrix}
=
\begin{pmatrix}
b_1 \\
b_2 \\
b_3
\end{pmatrix}
\end{equation}
```

# graph with 2 column

```latex
\begin{figure*}[t]
    \centering
    \includegraphics[width=0.9\textwidth]{HIM_process}
    \caption{Process of HIM predict implicit observation and current velocuty based on historic observation, which is concatenated with proprioceptive observation for policy to generate action and training of HIM with SwAV}
    \label{fig:himproces}
\end{figure*}
```

# 子图并排

```latex
\begin{figure}[htbp]
    \centering
    \begin{subfigure}[b]{0.2\textwidth}
        \centering
        \includegraphics[width=0.45\textwidth]{pic/rgbimg_with_shadow.png}
        \caption{RGB images with deep shadow}
        \label{fig:image1}
    \end{subfigure}
    \begin{subfigure}[b]{0.2\textwidth}
        \centering
        \includegraphics[width=0.45\textwidth]{pic/rgbimg_without_shadow.png}
        \caption{RGB images after removing shadow}
        \label{fig:image2}
    \end{subfigure}
    \caption{Shadow Schema}
    \label{fig:subfigures}
\end{figure}
```

# table

```latex
\begin{table}[htbp] % 定义表格的位置
    \centering % 将表格居中
    \caption{不同数据集下Bert、XLNet、RoBERTa的对比} % 添加标题
    \label{tab:example} % 创建标签
    \scalebox{1.0}{
        \begin{tabular}{|c|c|c|c|}
        \hline
        accuracy & Bert & XLNet & RoBERTa \\
        \hline
        imdb &  h & 90.17 & 81.50 \\
        \hline
        sentiment\_140 & h & 85.97 & 82.36 \\ 
        \hline
        \end{tabular}
    }
\end{table}
```

# bib

在 LaTeX 中使用 BibTeX 引用参考文献的基本步骤如下：

## 1. 创建 `.bib` 文件

首先，你需要创建一个 `.bib` 文件来存储你的参考文献。这个文件的扩展名通常是 `.bib`，例如 `references.bib`。在这个文件中，你可以按照 BibTeX 格式添加参考文献条目。以下是一个示例条目：

```bibtex
@article{smith2023,
  author = {John Smith},
  title = {An Interesting Article},
  journal = {Journal of Interesting Research},
  year = {2023},
  volume = {10},
  number = {2},
  pages = {100-110},
}
```

## 2. 在 LaTeX 文档中引用

在你的 LaTeX 文档中，你需要加载 `natbib` 或 `biblatex` 包，并指定你的 `.bib` 文件。以下是一个简单的示例：

```latex
\documentclass{article}
\usepackage{natbib}  % 或者使用 \usepackage{biblatex}

\begin{document}

这是一个引用示例 \citep{smith2023}。

\bibliographystyle{plain}  % 选择参考文献样式
\bibliography{references}   % 指定 .bib 文件名（不带扩展名）

\end{document}
```

## 3. 编译文档

编译 LaTeX 文档时，你需要按照以下顺序运行命令：

1. `pdflatex yourfile.tex`
2. `bibtex yourfile`
3. `pdflatex yourfile.tex`
4. `pdflatex yourfile.tex`

这将确保引用和参考文献正确生成。

## 4. 选择参考文献样式

在 `\bibliographystyle{}` 中，你可以选择不同的样式，例如：

- `plain`：按字母顺序排列，数字引用。
- `unsrt`：按引用顺序排列。
- `alpha`：使用作者名和年份的缩写。
- `apalike`：APA 风格。

### 使用 `biblatex`

如果你选择使用 `biblatex`，你可以这样做：

```latex
\documentclass{article}
\usepackage[backend=biber]{biblatex}

\addbibresource{references.bib}  % 指定 .bib 文件名

\begin{document}

这是一个引用示例 \cite{smith2023}。

\printbibliography  % 打印参考文献

\end{document}
```

然后，使用 `biber` 代替 `bibtex` 进行编译。

### 总结

通过以上步骤，你可以在 LaTeX 文档中使用 BibTeX 引用参考文献。确保正确设置 `.bib` 文件和引用命令，以便生成所需的参考文献列表。

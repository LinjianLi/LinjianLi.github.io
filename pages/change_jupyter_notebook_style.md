# Change Jupyter Notebook Style

## Change Fonts

我是在 Anaconda 环境下的，因此到 Anaconda 目录下，
`path-to-Anaconda3/lib/site-packages/notebook/static/custom`，找到 `custom.css` 文件，然后编辑。

原本的 Jupyter Notebook 的字体是类似 Times New Roman 的衬线字体，行距也比较小。
我想要简单地把字体换成代码用的等宽字体 DejaVu Sans Mono ，并且修改行距，于是我就在 `custom.css` 里面修改字体
```
/*
Placeholder for custom user CSS

mainly to be overridden in profile/static/custom/custom.css

This will always be an empty file in IPython
*/

* {font-family: DejaVu Sans Mono; line-height: 1.5em;}
```
以上代码块的最后一行就是我添加的内容。
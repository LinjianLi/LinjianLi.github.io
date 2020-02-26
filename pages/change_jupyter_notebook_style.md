# Change Jupyter Notebook Style

## Change Fonts

更改字体的主要原因是默认的代码块的字体是类似宋体英文的字体，非常难看，很多符号区分不开。例如英文字母 “l” 和阿拉伯数字 “1”。

我是在 Anaconda 环境下的，因此到 Anaconda 目录下，
`path-to-Anaconda3/lib/site-packages/notebook/static/custom`，找到 `custom.css` 文件，然后编辑。

***注意：不同环境的 CSS 是独立开来的，前面的路径指的是 `base` 环境，修改 `base` 环境的 CSS 不会影响其他环境的显示效果！***

有时候修改 `site-packages/notebook/static/custom` 里面的 `custom.css` 还是不能起效，这时候就要去用户目录里找到 `.jupyter` 目录，即 `~/.jupyter` ，在该目录下新建文件夹 `custom` ，然后在里面新建 `custom.css` 文件，即 `~/.jupyter/custom/custom.css` ，这时候在这个 CSS 文件里面修改样式即可。

我想要简单地把字体换成代码用的等宽字体 DejaVu Sans Mono ，于是我就在 `custom.css` 里面修改字体
```
/*
Placeholder for custom user CSS

mainly to be overridden in profile/static/custom/custom.css

This will always be an empty file in IPython
*/

.CodeMirror {
    font-family: "DejaVu Sans Mono", "Source Code Pro", source-code-pro,Consolas, Arial, monospace;
}

pre {
    font-family: "DejaVu Sans Mono", "Source Code Pro", source-code-pro,Consolas, Arial, monospace;
}
```
以上代码块的最后几行就是我添加的内容。
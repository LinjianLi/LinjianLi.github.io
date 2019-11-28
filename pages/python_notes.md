# Notes about Python

## Encoding

我在 Ubuntu 18.04 上运行 Python 程序的时候遇到过源文件的编码问题，会出现类似以下的情况

```shell
SyntaxError: Non-ASCII character '\xe6' in file my_python_program.py on line 5, but no encoding declared; see http://python.org/dev/peps/pep-0263/ for details
```

```shell
if not line.startswith('  File "<frozen importlib._bootstrap'))
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe5 in position 11: ordinal not in range(128)
```

这时需要在源代码的开头加上

```shell
# -*- coding: utf-8 -*-
import sys
reload(sys)
sys.setdefaultencoding("utf-8")
```

## 正则表达式

在参加大数据比赛的时候，我负责数据清洗的工作，但我在使用正则表达式去除特殊符号的时候，我发现数字全被去除了，而我本意没有要去除这些数字。原来是我在正则表达式中“不小心”使用了 character range 的表示方法。例子如下

```python
text = '+12-3ab+c_456d_ef'
text = re.sub(r'[+-_]+', '', text)
print(text)
```

我想去除加号、减号、下划线，但是会输出 `abcdef`。因为我写的 `[+-_]` 会被解析成类似 `[0-9]` 这种使用范围表示的字符。如果我把 `[+-_]` 替换成 `[a-_]`，运行就会报错 `bad character range a-_ at position 1`。
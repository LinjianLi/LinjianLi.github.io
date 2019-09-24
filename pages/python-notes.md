# Encoding

我在 Ubuntu 18.04 上运行 Python 程序的时候遇到过源文件的编码问题，会出现类似以下的情况
```
SyntaxError: Non-ASCII character '\xe6' in file my_python_program.py on line 5, but no encoding declared; see http://python.org/dev/peps/pep-0263/ for details
```
```
if not line.startswith('  File "<frozen importlib._bootstrap'))
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe5 in position 11: ordinal not in range(128)
```

这时需要在源代码的开头加上
```
# -*- coding: utf-8 -*-
import sys
reload(sys)
sys.setdefaultencoding("utf-8")
```
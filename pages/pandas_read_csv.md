# pandas.read_csv 函数读取 CSV 文件时的坑

如果 CSV 文件的字段被英文双引号包围，`pandas.read_csv` 函数**并不是**只会读取双引号内的内容，
而是会把双引号外的英文逗号当作分隔符，其他内容全部读取。

通过以下例子就能明白了

新建一个 CSV 文件（姑且命名为 `a_csv_file.csv`），然后写入以下内容，注意第三行和第四行是有一些空格的。
```
col1,col2,col3
abcd,efgh,ijkl
"abcd","efgh","ijkl"
"abcd" , "efgh" , "ijkl"  
"abcd" a bug ,  "efgh"  ,  "ijkl"  
```

我们在 Jupyter Notebook 里面进行测试
```Python
import pandas as pd
a_csv = pd.read_csv('a_csv_file.csv', header=0)
print(a_csv['col1'][0]==a_csv['col1'][1])
print(a_csv['col1'][0]==a_csv['col1'][2])
```

上面输出一个 True 和一个 False ，为什么第二个输出是 False ？输出一下第二个 Print 函数里面的两个字符串

```Python
print(a_csv['col1'][0])
print(a_csv['col1'][2])
print(a_csv['col1'][0]==a_csv['col1'][2])
```

我们看到输出应该是一模一样的两个字符串，但就是判断为 False。 于是我们用 `difflib` 来查看一下到底有什么区别

```Python
import difflib
print('\n'.join(difflib.ndiff([a_csv['col1'][0]], [a_csv['col1'][2]])))
```

能得到以下输出

```
- abcd
+ abcd 
?     +
```

可以看到第二个字符串底下有个“+”号，我们选中第二个字符串，可以发现第二个字符串后面好像有个空格。
我们再查看一下两个字符串的长度。并且把第二个字符串的末尾和空格符号比较一下。


```Python
print(len(a_csv['col1'][0]), '--', len(a_csv['col1'][2]))
print(a_csv['col1'][2][-1]==' ')
```

能得到以下输出

```
4 -- 5
True
```

Voilà

这就是开头所写的

> 如果 CSV 文件的字段被英文双引号包围，`pandas.read_csv` 函数**并不是**只会读取双引号内的内容，
而是会把双引号外的英文逗号当作分隔符，其他内容全部读取。

我们可以再输出其他字段验证一下
```Python
print(a_csv['col1'][3])
```

可以看到输出的字符串里面是包含了“ a bug ”的。
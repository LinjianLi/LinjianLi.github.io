# 用计数的方式表示编程中多个对象的取值组合

我在用 C++ 做 Bayesian network 的项目的时候，会遇到这样一种情况：有一堆有顺序的离散变量（用从 0 开始的数字给编号），不同变量的取值范围不同，我需要表示这一组变量的某一种 instantiation 的时候，我就只能用 `set< pair<int, int> >` 来表示。例如，`{<0,1>, <1,4>}` 就表示变量 0 取值为 1，变量 1 取值为4。可是用这种方式就感觉空间开销还是挺大的，将来要判断某两个 instantiation 是否相等的时候的开销也很大。

我后来去 [Weka 3.8 GitHub 只读镜像](https://github.com/Waikato/weka-3.8) 下载并阅读 Weka 3.8 的源码，发现 Weka 的开发者用了另一种方法来表示一组变量的 instantiation，只需要用一个 `int` 就能表示一种 instantiation。我觉得他们的做法挺好的，就在此记录下来。

## 前置条件

1. 变量必须从 0 开始按顺序编号，不能跳号。
2. 每个变量的取值范围必须是从 0 开始的连续的非负整数。
3. 如果原始数据不满足条件 2，那可以用别的数据结构来记录如何转换，例如原始数据中某个变量的取值是 `{low, middle, high}`，就可以用 `map` 或 `dictionary` 做一个映射 `{0: low, 1: middle, 2: high}` 从而满足条件 2。

## Notation

insta 表示某一组已编号的变量，`insta[x]` 表示这组变量的第 x 个变量此时的取值。

dom(x) 表示变量 x 的取值范围（domain），即从 0 开始的一个有限连续非负整数集。

card(dom(x)) 表示变量 x 的取值范围的大小（cardinality），即这个变量有多少个可能取值。

## Method

其实就是应用了 n-ary count 的思想。给每一种 instantiation 一个编号，全 0 就编号为 0，然后最右边的变量 +1 的时候编号就 +1，某个变量到了上限时就进位。所以算法应该是这样

```c++
// C++
number = 0;
for (int i = 0; i < card(dom(i)); ++i) {
    number = number * card(dom(i)) + insta[i];
}
return number;
```

```python
# Python
number = 0
for i in range(card(dom(x))):
    number = number * card(dom(i)) + insta[i]
return number
```

我们可以用二进制来分析一下。假如一组变量 `a, b, c` 的取值只有 `{0, 1}`，我们给这组变量编号为 `0, 1, 2`。现在我们假设一个 instantiation 为 `a=1, b=0, c=1`，这时候就相当于 `insta = [1, 0, 1]`。那么按照上面的算法，这个 instantiation 的编号就应该是

```none
(((0 * 2 + 1) * 2 + 0) * 2 + 1) = 5
```

我们可以发现，这其实就是从二进制数 101 转换成十进制数 5 的过程。

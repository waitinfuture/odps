# Collection {#concept_nsj_ft4_cfb .concept}

DataFrame中所有二维数据集上的操作都属于 [PyODPS DataFrame指南（DataFrame Reference）](cn.zh-CN/用户指南/PyODPS/API Reference/API概述.md#)，可视为一张ODPS表或一张电子表单，DataFrame对象也是CollectionExpr的特例。

CollectionExpr中包含针对二维数据集的列操作、筛选、变换等大量操作。

## 获取类型 {#section_grp_nt4_cfb .section}

`dtypes`可以用来获取CollectionExpr中所有列的类型。`dtypes` 返回的是[Schema类型](cn.zh-CN/用户指南/PyODPS/基本操作/表.md#) 。

```
>>> iris.dtypes
odps.Schema {
  sepallength           float64
  sepalwidth            float64
  petallength           float64
  petalwidth            float64
  name                  string
}
```

## 列选择和增删 {#section_prv_tt4_cfb .section}

如果要从一个CollectionExpr中选取部分列，产生新的数据集，可以使用expr\[columns\] 语法。例如，

```
iris['name', 'sepallength'].head(5)
          name  sepallength
0  Iris-setosa          5.1
1  Iris-setosa          4.9
2  Iris-setosa          4.7
3  Iris-setosa          4.6
4  Iris-setosa          5.0
```

**说明：** 

如果需要选择的列只有一列，需要在columns后加上逗号或者显示标记为列表，例如 df\[df.sepal\_length, \] 或 df\[\[df.sepal\_length\]\]，否则返回的将是一个Sequence对象，而不是Collection。

如果想要在新的数据集中排除已有数据集的某些列，可使用exclude方法：

```
iris.exclude('sepallength', 'petallength')[:5]
   sepalwidth  petalwidth         name
0         3.5         0.2  Iris-setosa
1         3.0         0.2  Iris-setosa
2         3.2         0.2  Iris-setosa
3         3.1         0.2  Iris-setosa
4         3.6         0.2  Iris-setosa
```

0.7.2以后的PyODPS支持另一种写法，即在数据集上直接排除相应的列：

```
del iris['sepallength']
del iris['petallength']
iris[:5]
   sepalwidth  petalwidth         name
0         3.5         0.2  Iris-setosa
1         3.0         0.2  Iris-setosa
2         3.2         0.2  Iris-setosa
3         3.1         0.2  Iris-setosa
4         3.6         0.2  Iris-setosa
```

如果我们需要在已有collection中添加某一列变换的结果，也可以使用expr\[expr, new\_sequence\]语法， 新增的列会作为新collection的一部分。

下面的例子将iris中的sepalwidth列加一后重命名为sepalwidthplus1并追加到数据集末尾，形成新的数据集：

```
iris[iris, (iris.sepalwidth + 1).rename('sepalwidthplus1')].head(5)
   sepallength  sepalwidth  petallength  petalwidth         name  \
0          5.1         3.5          1.4         0.2  Iris-setosa
1          4.9         3.0          1.4         0.2  Iris-setosa
2          4.7         3.2          1.3         0.2  Iris-setosa
3          4.6         3.1          1.5         0.2  Iris-setosa
4          5.0         3.6          1.4         0.2  Iris-setosa

   sepalwidthplus1
0              4.5
1              4.0
2              4.2
3              4.1
4              4.6
```

使用 df\[df, new\_sequence\] 需要注意的是，变换后的列名与原列名可能相同，如果需要与原collection合并， 请将该列重命名。

0.7.2以后版本的PyODPS支持直接在当前数据集中追加，写法为：

```
iris['sepalwidthplus1'] = iris.sepalwidth + 1
iris.head(5)
   sepallength  sepalwidth  petallength  petalwidth         name  \
0          5.1         3.5          1.4         0.2  Iris-setosa
1          4.9         3.0          1.4         0.2  Iris-setosa
2          4.7         3.2          1.3         0.2  Iris-setosa
3          4.6         3.1          1.5         0.2  Iris-setosa
4          5.0         3.6          1.4         0.2  Iris-setosa

   sepalwidthplus1
0              4.5
1              4.0
2              4.2
3              4.1
4              4.6
```

我们也可以先将原列通过exclude方法进行排除，再将变换后的新列并入，而不必担心重名。

```
iris[iris.exclude('sepalwidth'), iris.sepalwidth * 2].head(5)
   sepallength  petallength  petalwidth         name  sepalwidth
0          5.1          1.4         0.2  Iris-setosa         7.0
1          4.9          1.4         0.2  Iris-setosa         6.0
2          4.7          1.3         0.2  Iris-setosa         6.4
3          4.6          1.5         0.2  Iris-setosa         6.2
4          5.0          1.4         0.2  Iris-setosa         7.2
```

对于0.7.2以后版本的PyODPS，如果想在当前数据集上直接覆盖，则可以写

```
iris['sepalwidth'] = iris.sepalwidth * 2
iris.head(5)
   sepallength  sepalwidth  petallength  petalwidth         name
0          5.1         7.0          1.4         0.2  Iris-setosa
1          4.9         6.0          1.4         0.2  Iris-setosa
2          4.7         6.4          1.3         0.2  Iris-setosa
3          4.6         6.2          1.5         0.2  Iris-setosa
4          5.0         7.2          1.4         0.2  Iris-setosa
```

增删列以创建新collection的另一种方法是调用select方法，将需要选择的列作为参数输入。如果需要重命名，使用keyword参数输入，并将新的列名作为参数名即可。

```
iris.select('name', sepalwidthminus1=iris.sepalwidth - 1).head(5)
          name  sepalwidthminus1
0  Iris-setosa               2.5
1  Iris-setosa               2.0
2  Iris-setosa               2.2
3  Iris-setosa               2.1
4  Iris-setosa               2.6
```

此外，我们也可以传入一个lambda表达式，它接收一个参数，接收上一步的结果。在执行时，PyODPS会检查这些lambda表达式，传入上一步生成的collection并将其替换为正确的列。

```
iris['name', 'petallength'][[lambda x: x.name]].head(5)
          name
0  Iris-setosa
1  Iris-setosa
2  Iris-setosa
3  Iris-setosa
4  Iris-setosa
```

此外，在0.7.2以后版本的PyODPS中，支持对数据进行条件赋值，例如

```
iris[iris.sepallength > 5.0, 'sepalwidth'] = iris.sepalwidth * 2
iris.head(5)
   sepallength  sepalwidth  petallength  petalwidth         name
0          5.1        14.0          1.4         0.2  Iris-setosa
1          4.9         6.0          1.4         0.2  Iris-setosa
2          4.7         6.4          1.3         0.2  Iris-setosa
3          4.6         6.2          1.5         0.2  Iris-setosa
4          5.0         7.2          1.4         0.2  Iris-setosa
```

## 引入常数和随机数 {#section_sbm_j54_cfb .section}

DataFrame支持在collection中追加一列常数。追加常数需要使用 [Scalar](cn.zh-CN/用户指南/PyODPS/API Reference/API概述.md#)，引入时需要手动指定列名，如：

```
from odps.df import Scalar
iris[iris, Scalar(1).rename('id')][:5]
   sepallength  sepalwidth  petallength  petalwidth         name  id
0          5.1         3.5          1.4         0.2  Iris-setosa   1
1          4.9         3.0          1.4         0.2  Iris-setosa   1
2          4.7         3.2          1.3         0.2  Iris-setosa   1
3          4.6         3.1          1.5         0.2  Iris-setosa   1
4          5.0         3.6          1.4         0.2  Iris-setosa   1
```

如果需要指定一个空值列，可以使用[NullScalar](cn.zh-CN/用户指南/PyODPS/API Reference/API概述.md#) ，需要提供字段类型。

```
from odps.df import NullScalar
iris[iris, NullScalar('float').rename('fid')][:5]
   sepal_length  sepal_width  petal_length  petal_width     category   fid
0           5.1          3.5           1.4          0.2  Iris-setosa  None
1           4.9          3.0           1.4          0.2  Iris-setosa  None
2           4.7          3.2           1.3          0.2  Iris-setosa  None
3           4.6          3.1           1.5          0.2  Iris-setosa  None
4           5.0          3.6           1.4          0.2  Iris-setosa  None
```

在PyODPS 0.7.12及以后版本中，引入了简化写法：

```
iris['id'] = 1
iris
   sepallength  sepalwidth  petallength  petalwidth         name  id
0          5.1         3.5          1.4         0.2  Iris-setosa   1
1          4.9         3.0          1.4         0.2  Iris-setosa   1
2          4.7         3.2          1.3         0.2  Iris-setosa   1
3          4.6         3.1          1.5         0.2  Iris-setosa   1
4          5.0         3.6          1.4         0.2  Iris-setosa   1
```

需要注意的是，这种写法无法自动识别空值的类型，所以在增加空值列时，仍然要使用如下代码：

```
iris['null_col'] = NullScalar('float')
iris
   sepallength  sepalwidth  petallength  petalwidth         name  null_col
0          5.1         3.5          1.4         0.2  Iris-setosa      None
1          4.9         3.0          1.4         0.2  Iris-setosa      None
2          4.7         3.2          1.3         0.2  Iris-setosa      None
3          4.6         3.1          1.5         0.2  Iris-setosa      None
4          5.0         3.6          1.4         0.2  Iris-setosa      None
```

DataFrame也支持在collection中增加一列随机数列，该列类型为float，范围为0 - 1，每行数值均不同。 追加随机数列需要使用 [RandomScalar](cn.zh-CN/用户指南/PyODPS/API Reference/API概述.md#)，参数为随机数种子，可省略。

```
from odps.df import RandomScalar
iris[iris, RandomScalar().rename('rand_val')][:5]
   sepallength  sepalwidth  petallength  petalwidth         name  rand_val
0          5.1         3.5          1.4         0.2  Iris-setosa  0.000471
1          4.9         3.0          1.4         0.2  Iris-setosa  0.799520
2          4.7         3.2          1.3         0.2  Iris-setosa  0.834609
3          4.6         3.1          1.5         0.2  Iris-setosa  0.106921
4          5.0         3.6          1.4         0.2  Iris-setosa  0.763442
```

## 过滤数据 {#section_ol1_bw4_cfb .section}

Collection提供了数据过滤的功能，我们试着查询`sepallength`大于5的几条数据。

```
iris[iris.sepallength > 5].head(5)
   sepallength  sepalwidth  petallength  petalwidth         name
0          5.1         3.5          1.4         0.2  Iris-setosa
1          5.4         3.9          1.7         0.4  Iris-setosa
2          5.4         3.7          1.5         0.2  Iris-setosa
3          5.8         4.0          1.2         0.2  Iris-setosa
4          5.7         4.4          1.5         0.4  Iris-setosa
```

多个查询条件：

```
iris[(iris.sepallength < 5) & (iris['petallength'] > 1.5)].head(5)
   sepallength  sepalwidth  petallength  petalwidth             name
0          4.8         3.4          1.6         0.2      Iris-setosa
1          4.8         3.4          1.9         0.2      Iris-setosa
2          4.7         3.2          1.6         0.2      Iris-setosa
3          4.8         3.1          1.6         0.2      Iris-setosa
4          4.9         2.4          3.3         1.0  Iris-versicolor
```

或条件：

```
iris[(iris.sepalwidth < 2.5) | (iris.sepalwidth > 4)].head(5)
   sepallength  sepalwidth  petallength  petalwidth             name
0          5.7         4.4          1.5         0.4      Iris-setosa
1          5.2         4.1          1.5         0.1      Iris-setosa
2          5.5         4.2          1.4         0.2      Iris-setosa
3          4.5         2.3          1.3         0.3      Iris-setosa
4          5.5         2.3          4.0         1.3  Iris-versicolor
```

**说明：** 记住，与和或条件必须使用&和|，不能使用and和or。

非条件：

```
iris[~(iris.sepalwidth > 3)].head(5)
   sepallength  sepalwidth  petallength  petalwidth         name
0          4.9         3.0          1.4         0.2  Iris-setosa
1          4.4         2.9          1.4         0.2  Iris-setosa
2          4.8         3.0          1.4         0.1  Iris-setosa
3          4.3         3.0          1.1         0.1  Iris-setosa
4          5.0         3.0          1.6         0.2  Iris-setosa
```

我们也可以显式调用filter方法，提供多个与条件：

```
iris.filter(iris.sepalwidth > 3.5, iris.sepalwidth < 4).head(5)
   sepallength  sepalwidth  petallength  petalwidth         name
0          5.0         3.6          1.4         0.2  Iris-setosa
1          5.4         3.9          1.7         0.4  Iris-setosa
2          5.4         3.7          1.5         0.2  Iris-setosa
3          5.4         3.9          1.3         0.4  Iris-setosa
4          5.7         3.8          1.7         0.3  Iris-setosa
```

同样对于连续的操作，我们可以使用lambda表达式：

```
iris[iris.sepalwidth > 3.8]['name', lambda x: x.sepallength + 1]
          name  sepallength
0  Iris-setosa          6.4
1  Iris-setosa          6.8
2  Iris-setosa          6.7
3  Iris-setosa          6.4
4  Iris-setosa          6.2
5  Iris-setosa          6.5
```

对于Collection，如果它包含一个列是BOOLEAN类型，则可以直接使用该列作为过滤条件。

```
df.dtypes
odps.Schema {
  a boolean
  b int64
}
df[df.a]
      a  b
0  True  1
1  True  3
```

因此，记住对Collection取单个squence的操作时，只有BOOLEAN列是合法的，即对Collection作过滤操作。

```
df[df.a, ]       # 取列操作
df[[df.a]]       # 取列操作
df.select(df.a)  # 显式取列
df[df.a]         # a列是boolean列，执行过滤操作
df.a             # 取单列
df['a']          # 取单列
```

同时，我们也支持Pandas中的`query`方法，用查询语句来做数据的筛选，在表达式中直接使用列名如`sepallength`进行操作， 另外在查询语句中`&`和`and`都表示与操作，`|`和`or`都表示或操作。

```
iris.query("(sepallength < 5) and (petallength > 1.5)").head(5)
   sepallength  sepalwidth  petallength  petalwidth             name
0          4.8         3.4          1.6         0.2      Iris-setosa
1          4.8         3.4          1.9         0.2      Iris-setosa
2          4.7         3.2          1.6         0.2      Iris-setosa
3          4.8         3.1          1.6         0.2      Iris-setosa
4          4.9         2.4          3.3         1.0  Iris-versicolor
```

当表达式中需要使用到本地变量时，需要在该变量前加一个`@`前缀。

```
var = 4
iris.query("(iris.sepalwidth < 2.5) | (sepalwidth > @var)").head(5)
   sepallength  sepalwidth  petallength  petalwidth             name
0          5.7         4.4          1.5         0.4      Iris-setosa
1          5.2         4.1          1.5         0.1      Iris-setosa
2          5.5         4.2          1.4         0.2      Iris-setosa
3          4.5         2.3          1.3         0.3      Iris-setosa
4          5.5         2.3          4.0         1.3  Iris-versicolor
```

目前`query`支持的语法包括：

|语法|说明|
|:-|:-|
|name|没有`@`前缀的都当做列名处理，有前缀的会获取本地变量|
|operator|支持部分运算符：`+`，`-`，`*`，`/`，`//`，`%`，`**`, `==`，`!=`，`<`，`<=`，`>`，`>=`，`in`，`not in`|
|bool|与或非操作，其中`&`和`and`表示与，`|`和`or`表示或|
|attribute|取对象属性|
|index, slice, Subscript|切片操作|

## 并列多行输出 {#section_ndh_rw4_cfb .section}

对于list及map类型的列，explode方法会将该列转换为多行输出。使用apply方法也可以输出多行。 为了进行聚合等操作，常常需要将这些输出和原表中的列合并。此时可以使用 DataFrame提供的并列多行输出功能， 写法为将多行输出函数生成的集合与原集合中的列名一起映射。

并列多行输出的例子如下：

```
df
   id         a             b
0   1  [a1, b1]  [a2, b2, c2]
1   2      [c1]      [d2, e2]
df[df.id, df.a.explode(), df.b]
   id   a             b
0   1  a1  [a2, b2, c2]
1   1  b1  [a2, b2, c2]
2   2  c1      [d2, e2]
df[df.id, df.a.explode(), df.b.explode()]
   id   a   b
0   1  a1  a2
1   1  a1  b2
2   1  a1  c2
3   1  b1  a2
4   1  b1  b2
5   1  b1  c2
6   2  c1  d2
7   2  c1  e2
```

如果多行输出方法对某个输入不产生任何输出，默认输入行将不在最终结果中出现。如果需要在结果中出现该行，可以设置

`keep_nulls=True`

。此时，与该行并列的值将输出为空值：

```
df
   id         a
0   1  [a1, b1]
1   2        []
df[df.id, df.a.explode()]
   id   a
0   1  a1
1   1  b1
df[df.id, df.a.explode(keep_nulls=True)]
   id     a
0   1    a1
1   1    b1
2   2  None
```

关于explode使用并列输出的具体文档可参考 [集合类型相关操作](cn.zh-CN/用户指南/PyODPS/DataFrame/列运算.md#)，对于 apply方法使用并列输出的例子可参考 [对一行数据使用自定义函数](cn.zh-CN/用户指南/PyODPS/DataFrame/排序、去重、采样、数据变换.md#)。

## 限制条数 {#section_n1y_gx4_cfb .section}

```
iris[:3]
   sepallength  sepalwidth  petallength  petalwidth         name
0          5.1         3.5          1.4         0.2  Iris-setosa
1          4.9         3.0          1.4         0.2  Iris-setosa
2          4.7         3.2          1.3         0.2  Iris-setosa
```

值得注意的是，目前切片对于ODPS SQL后端不支持start和step。我们也可以使用limit方法

```
iris.limit(3)
   sepallength  sepalwidth  petallength  petalwidth         name
0          5.1         3.5          1.4         0.2  Iris-setosa
1          4.9         3.0          1.4         0.2  Iris-setosa
2          4.7         3.2          1.3         0.2  Iris-setosa
```

**说明：** 另外，切片操作只能作用在collection上，不能作用于sequence。


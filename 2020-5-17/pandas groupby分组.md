# pandas groupby分组

![1589698179293](C:\Users\戴尔\AppData\Roaming\Typora\typora-user-images\1589698179293.png)

​	通常我们面对数据清洗时，需要对数据加以操作，进行分组，聚合，迭代之类。我们可以使用pandas库简化操作。如以下数据，

```
      data1     data2 key1 key2
0 -0.410122  0.247895    a  one
1 -0.627470 -0.989268    a  two
2  0.179488 -0.054570    b  one
3 -0.299878 -1.640494    b  two
4 -0.297191  0.954447    a  one

```

## 1.按一个key进行分组

```
for name,group in df.groupby(['key1']):
    print(name)
    print(group)
```

​	如果我们想要通过key1的值进行分组，结果如下：

```
a
      data1     data2 key1 key2
0 -0.410122  0.247895    a  one
1 -0.627470 -0.989268    a  two
4 -0.297191  0.954447    a  one
b
      data1     data2 key1 key2
2  0.179488 -0.054570    b  one
3 -0.299878 -1.640494    b  two

```

## 2. 按 多个key进行分组

如果我们想要对于多个限制条件进行分组的话，直接在groupby里面增加条件就行

```python
for name,group in df.groupby(['key1','key2']):
    print(name)  #name=(k1,k2)
    print(group)

```

结果如下

```
('a', 'one')
      data1     data2 key1 key2
0 -0.410122  0.247895    a  one
4 -0.297191  0.954447    a  one
('a', 'two')
     data1     data2 key1 key2
1 -0.62747 -0.989268    a  two
('b', 'one')
      data1    data2 key1 key2
2  0.179488 -0.05457    b  one
('b', 'two')
      data1     data2 key1 key2
3 -0.299878 -1.640494    b  two

```

我们也可以针对一列，而用另一列进行分组并进行一些计算，如

### 对data1按key1进行分组，并计算data1列的平均值

```
df['data1'].groupby(df['key1']).mean()
```

结果如下

```
key1
a   -0.444928
b   -0.060195
Name: data1, dtype: float64

```

等同于

```
df.groupby(['key1'])['data1'].mean()#理解：对df按key1分组，并计算分组后df['data1']的均值
```





## 重点1 agg函数

使用groupby函数之后，我们可以通过

![1589698768656](C:\Users\戴尔\AppData\Roaming\Typora\typora-user-images\1589698768656.png)

如上函数，对groupby分组后的数据进行操作，如 df.groupby(['key1'])['data1'].min()

但现实中我们往往想要自定义函数，满足聚合的目的，因此我们可以使用agg函数

比如我们定义一个最大值-最小值的函数

```
def cal_size(x):
       return x.max() - x.min()
```

运用的时候直接

```
df.groupby(['key1'])['data1'].agg(cal_size)
```

agg也可以同时传入多个函数，比如我们同时得到最大值与最小值

![1589699039157](C:\Users\戴尔\AppData\Roaming\Typora\typora-user-images\1589699039157.png)

```
df.groupby(['key1'])['data1'].agg({'min','max'})
```

![1589699054539](C:\Users\戴尔\AppData\Roaming\Typora\typora-user-images\1589699054539.png)

## 重点二 apply函数

apply函数是对于整个DataFrame的每一行元素进行相同的操作

比如我们有以下数据

```
matrix = [
    [1,2,3],
    [4,5,6],
    [7,8,9]
]
```

我们想要对每一个元素进行平方，可以这么写

```
df.apply(np.square)
```

结果如下

```
    x   y   z
a   1   4   9
b  16  25  36
c  49  64  81

```

如果想要只对x列进行平方

```
df.apply(lambda x : np.square(x) if x.name=='x' else x)
```

结果如下

```
    x  y  z
a   1  2  3
b  16  5  6
c  49  8  9

```

也可以传入自定义函数 如

```
def get_top(df,n):
    return df.head(n)

d = grouped.apply(get_top,n=3)

print(d)
```


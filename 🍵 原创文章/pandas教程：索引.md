## 前言

在本次学习中，我们主要聚焦于Series和DataFrame中的一些索引操作。比如：

- 通过索引访问Series、DataFrame中指定的元素
- 随机访问Series、DataFrame中的数组
- 多级索引的构造和访问
- 索引层的交换和删除
- 索引值和名的修改
- 索引的设置和重置
- 索引的运算

希望通过学会“索引”的相关知识，让我们对一组数据能拥有精准“打击”的能力。

## 一、索引器

这一节主要讲解如何在不改变数据的前提下，利用索引对数据进行 **指定** 和 **非指定（随机）** 提取：

### 1.Series数据的行索引

先考虑Series类型的数据，对于一个包含shape为\(5,\)ndarray类型的Series数据，其直观表现形式如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222195003427.png#pic_center)

那么对它进行索引访问，返回结果势必只有3种情况：

- 目标索引不存在，报错
- 长度为1，类型为标量
- 长度大于1，类型依然为Series

接下来让我们举例说明：

**1）按值访问**

我们可以通过**s\[索引名\]** 和 **.索引名** 去访问，但需要注意的是后者需要保证索引名中不含有空格：

```python
>>> s = pd.Series(np.arange(1,8),index=['a','b','a','a','a','c',' d'])
>>> print(s)
a     1
b     2
a     3
a     4
a     5
c     6
 d    7
dtype: int32
>>> print(s[' d'])
7
>>> print(s.' d')
SyntaxError: invalid syntax
```

所以，接下来我们只介绍**s\[索引名\]** 这种方式，因为它具有一定的稳定性：

```python
>>> s = pd.Series(np.arange(1,7),index=['a','b','a','a','a','c'])
>>> s
a    1
b    2
a    3
a    4
a    5
c    6
dtype: int32
>>> try:
>>>     res = s['d']
>>> except Exception as e:
>>>     Err_Msg = e
>>> Err_Msg
KeyError('d')

# 情况2 返回结果为标量
2 <class 'numpy.int32'>
>>> res = s['b']
>>> print(res,type(res))

# 情况3 返回结果为Series
>>> res = s['a']
>>> print(res,type(res))
a    1
a    3
a    4
a    5
dtype: int32 <class 'pandas.core.series.Series'>
```

我们可以看到，由于Series的index的值并无唯一性，所以按值访问Series有可能得到不止一行的返回结果。

更进一步，我们可以按值对Series进行组合访问：

```python
>>> res = s[['b','c']]
>>> print(res,type(res))
b    2
c    6
dtype: int32 <class 'pandas.core.series.Series'>
```

另外，还有一种访问方法可以取出两个值之间的数据：

```python
>>> res = s['b':'c':2]
>>> print(res,type(res))
b    2
a    4
c    6
dtype: int32 <class 'pandas.core.series.Series'>
```

注意这里的索引值在无序状态下要保持唯一性，否则：

```python
>>> try:
>>>     res = s['a':'c':2]
>>> except Exception as e:
>>>     Err_Msg = e
>>> Err_Msg
KeyError("Cannot get left slice bound for non-unique label: 'a'")
```

如果不保证唯一性，但仍想正常访问，要提前使用`sort_index()`对index进行排序：

```python
>>> s.sort_index()['a':'c':2]
a    1
a    4
b    2
dtype: int32
```

另外需要注意的点是这种方式包含左右两端，且start点访问的是第一个出现的，end点访问的是最后一个出现的，如：

```python
>>> s = pd.Series(np.arange(1,7),index=['a','b','a','a','a','b'])
>>> s.sort_index()['a':'b']
a    1
a    3
a    4
a    5
b    2
b    6
dtype: int32
```

**2）按位置访问**

按位置访问其实和Python原生语法中对list数组的访问类似，这里只举几个例子：

```python
>>> s = pd.Series(np.arange(1,7),index=['a','b','a','a','a','c'])
>>> s[0]
1
>>> s[[0,2]]
a    1
a    3
dtype: int32
>>> s[0:3]
a    1
b    2
a    3
dtype: int32
```

我们可以看到按位置访问与按值访问不同，是“一个萝卜一个坑”，即输入的索引数量和输出的数据数量保持等比例。

注意，上面两种访问方式不能同时使用：

```python
>>> s[['b',2]]
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222211453960.png#pic_center)

### 2.DataFrame的列索引

再考虑DataFrame类型的数据，对于一个包含shape为\(5,5\)ndarray类型的DataFrame数据，其直观表现形式如下：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201223111702470.png#pic_center)

对它进行索引访问，返回结果也只有3种情况：

- 目标索引不存在，报错
- 长度为1，类型为Series
- 长度大于1，类型依然为DataFrame

对于DataFrame的列索引，以`.data/learn_pandas.csv`为目标数据举例说明：

```python
>>> df = pd.read_csv('data/learn_pandas.csv',usecols=['School', 'Grade', 'Name', 'Gender', 'Weight', 'Transfer'])
>>> df
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222211914109.png#pic_center)

下面两种访问方式是等价的：

```python
>>> res = df['School'].head(3)
>>> print(res,type(res))
0    Shanghai Jiao Tong University
1                Peking University
2    Shanghai Jiao Tong University
Name: School, dtype: object <class 'pandas.core.series.Series'>
>>> res = df.School.head(3)
>>> print(res,type(res))
0    Shanghai Jiao Tong University
1                Peking University
2    Shanghai Jiao Tong University
Name: School, dtype: object <class 'pandas.core.series.Series'>
```

但是对于列名中含有空格的列，与Series访问一致，只能通过**s\[列名\]** 去访问：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222212604264.png#pic_center)

我们偷偷进入数据文件中，把列名`School`改成`School Name`，保存后进行访问：

```python
>>> df.'School Name'
SyntaxError: invalid syntax
>>> df.School Name
SyntaxError: invalid syntax
>>> df['School Name'].head()
0    Shanghai Jiao Tong University
1                Peking University
2    Shanghai Jiao Tong University
3                 Fudan University
4                 Fudan University
Name: School Name, dtype: object
```

只有第三种方式可以，验证完之后我们再偷偷把`School Name`改回`School`。

另外，我们仍然可以通过列名组合去访问多个列：

```python
>>> print(df[['School','Gender']].head(3))
                          School  Gender
0  Shanghai Jiao Tong University  Female
1              Peking University    Male
2  Shanghai Jiao Tong University    Male
```

**但DataFrame数据不能按位置去访问**：

```python
>>> df[0]
KeyError: 0
```

简单说一下我的理解:

```python
#打印前两行的nd类型数据
>>> df.head(2).values
array([['Shanghai Jiao Tong University', 'Freshman', 'Gaopeng Yang',
        'Female', 46.0, 'N'],
       ['Peking University', 'Freshman', 'Changqiang You', 'Male', 70.0,
        'N']], dtype=object)
```

那是由于DataFrame和Series的值都是ndarray类型数组，在一维的情况下，我们可以把Series数据想象成一维数组，按位置访问即可返回标量。但在二维情况下，每一条数据与列不匹配，它代表的是由每个列属性组成的单条行数据，因此`df[0]`如果真的生效，访问的也与列无关，是单行数据，但是我们可以通过自定义方法按列的位置拿到数据：

```python
>>> def getCol(data,i):
>>>     return pd.Series(data.values[:,i])
>>> print(getCol(df,0).head(3))
0    Shanghai Jiao Tong University
1                Peking University
2    Shanghai Jiao Tong University
dtype: object
```

即利用numpy特性取出对应列的nd数据，然后再封成pandas数据类型。

### 3.loc索引器

loc索引器能提供更方便的索引访问，我们依然用DataFrame数据作为案例使用，我们事先先把`Name`列设置为行索引方便举例：

```python
>>> df.set_index('Name',inplace=True)
>>> df.head(3)
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222214609380.png#pic_center)  
loc的使用方法大致分为五种情况：

```python
#情况1 访问单个行索引 下面的写法是等价的 只给出一次打印
>>> print(df.loc['Mei Sun'])
>>> print(df.loc['Mei Sun',:])
                                School   Grade  Gender  Weight Transfer
Name                                                                   
Mei Sun  Shanghai Jiao Tong University  Senior    Male    89.0        N
Mei Sun  Shanghai Jiao Tong University  Junior  Female    50.0        N

#情况2 访问多个行索引
>>> print(df.loc[['Mei Sun','Li Zhao']])
>>> print(df.loc[['Mei Sun','Li Zhao'],:])
                                School   Grade  Gender  Weight Transfer
Name                                                                   
Mei Sun  Shanghai Jiao Tong University  Senior    Male    89.0        N
Mei Sun  Shanghai Jiao Tong University  Junior  Female    50.0        N
Li Zhao            Tsinghua University  Senior  Female    50.0        N

#情况3 切片访问
>>> print(df.loc['Gaojuan You':'Gaoqiang Qian','Grade':'Transfer'])
                   Grade  Gender  Weight Transfer
Name                                             
Gaojuan You    Sophomore    Male    74.0        N
Xiaoli Qian     Freshman  Female    51.0        N
Qiang Chu       Freshman  Female    52.0        N
Gaoqiang Qian     Junior  Female    50.0        N

#情况4 通过布尔列表访问
>>> print(df.loc[df.Gender == 'Male'].head(3))
                                       School      Grade Gender  Weight     Transfer 
Name                                                                      
Changqiang You              Peking University   Freshman   Male    70.0            N   
Mei Sun         Shanghai Jiao Tong University     Senior   Male    89.0            N      
Gaojuan You                  Fudan University  Sophomore   Male    74.0            N      
 
#情况5 通过函数访问
>>> def condition(x):
>>>     condition_1 = x.Gender == 'Female'
>>>     condition_2 = x.Grade == 'Freshman'
>>>     return condition_1 & condition_2
>>> print(df.loc[condition].head(3))
                                     School     Grade  Gender  Weight Transfer
Name                                                                          
Gaopeng Yang  Shanghai Jiao Tong University  Freshman  Female    46.0        N
Xiaoli Qian             Tsinghua University  Freshman  Female    51.0        N
Qiang Chu     Shanghai Jiao Tong University  Freshman  Female    52.0        N
```

注意在前三种情况中，loc的形式为loc\[A,B\]，其中A表示行索引，B表示列索引（可不显式指出），且A和B均可使用`[,]`、`[:]`方式进行组合和切片访问。

在第四种情况中，其实loc\[A\]中的A为一个dtype为布尔类型的Series数据。

而在第五种情况中，函数的入参其实代表整个DataFrame数据。

### 练一练

使用`select_dtypes()方法`：

```python
>>> df.select_dtypes('number').head()
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201223185302332.png#pic_center)

### 4.iloc索引器

iloc索引器的用法基本上和loc索引器的用法保持一致，不同的是`iloc[A,B]`中A和B是由位置而不是值构成的：

```python
#情况1 访问单行单列
>>> print(df.iloc[1,0])
Peking University
#情况2 访问多行多列
>>> print(df.iloc[[0,1],[1,2]])
                   Grade  Gender
Name                            
Gaopeng Yang    Freshman  Female
Changqiang You  Freshman    Male

#情况3 利用切片访问多行多列
>>> print(df.iloc[0:2,1:3])
                   Grade  Gender
Name                            
Gaopeng Yang    Freshman  Female
Changqiang You  Freshman    Male

#情况4 利用布尔列表访问
>>> print(df.iloc[(df.Gender == 'Female').values].tail(3))
                                       School   Grade  Gender  Weight Transfer
Name                                                                          
Xiaojuan Sun                 Fudan University  Junior  Female    46.0        N
Li Zhao                   Tsinghua University  Senior  Female    50.0        N
Chengqiang Chu  Shanghai Jiao Tong University  Senior  Female    45.0        N

#情况5 通过函数访问
>>> def condition(x):
>>>     condition_1 = x.Gender == 'Female'
>>>     condition_2 = x.Grade == 'Freshman'
>>>     return (condition_1 & condition_2).values
>>> print(df.loc[condition].head(3))
                                     School     Grade  Gender  Weight Transfer
Name                                                                          
Gaopeng Yang  Shanghai Jiao Tong University  Freshman  Female    46.0        N
Xiaoli Qian             Tsinghua University  Freshman  Female    51.0        N
Qiang Chu     Shanghai Jiao Tong University  Freshman  Female    52.0        N
```

可以看到，`iloc`的用法与`loc`大体相同，这里注意情况4和情况5，返回的Series类型要通过`.values`转为nd类型才可以正常使用。

### 5.query方法

`query方法`的使用类似于数据库SQL语句，对于列名，我们无需在前面加上DataFrame的变量名，而是直接写出即可访问，举例：

```python
# 例1 获得清华女生的信息
>>> query1 = '((School == "Tsinghua University") & (Gender == "Female"))'
>>> print(df.query(query1).head(3))
                            School     Grade  Gender  Weight Transfer
Name                                                                 
Xiaoli Qian    Tsinghua University  Freshman  Female    51.0        N
Gaoqiang Qian  Tsinghua University    Junior  Female    50.0        N
Changli Zhang  Tsinghua University  Freshman  Female    48.0        N

#例2 在参数中直接调用Series方法:获取体重最轻的学生信息
>>> query2 = 'Weight == Weight.min()'
>>> print(df.query(query2).head(3))
                                   School      Grade  Gender  Weight Transfer
Name                                                                         
Gaomei Lv                Fudan University     Senior  Female    34.0        N
Peng Han                Peking University  Sophomore  Female    34.0      NaN
Xiaoli Chu  Shanghai Jiao Tong University     Junior  Female    34.0        N

#例3 引用外部变量
>>> low, high =60, 70
>>> print(df.query('Weight.between(@low, @high)').head(3))
                          School     Grade Gender  Weight Transfer
Name                                                                 
Changqiang You    Peking University  Freshman   Male    70.0        N
Xiaoqiang Qin   Tsinghua University    Junior   Male    68.0        N
Peng Wang       Tsinghua University    Junior   Male    65.0        N
```

这里注意条件判断之间要用括号括起来，在语句中可以直接调用Series的方法。

### 6.随机抽样

`sample方法`可以对数据进行抽样，以Series数据为例：

```python
>>> s = pd.Series(np.arange(1,7),index=['a','b','a','a','a','c'])
>>> p = [0.1,0.2,0.1,0.1,0.2,0.3]
>>> s.sample(3, replace = True, weights = p)
b    2
c    6
b    2
dtype: int32
```

其中3表示抽样次数，replace表示每次抽完是否放回，weights表示每个样本抽中的概率。

## 二、多级索引

### 1.多级索引结构及其相应属性

多级索引，顾名思义，就是指行索引或列索引不止一层。

首先利用刚才的学生数据集构造多级索引：

```python
>>> np.random.seed(20201222)
>>> multi_index = pd.MultiIndex.from_product([list('ABC'), df.Gender.unique()], names=('School', 'Gender'))
>>> multi_column = pd.MultiIndex.from_product([['Height', 'Weight'], df.Grade.unique()], names=('Indicator', 'Grade'))
>>> df_multi = pd.DataFrame(np.c_[(np.random.randn(6,4)*5 + 163).tolist(), (np.random.randn(6,4)*5 + 65).tolist()],
                        index = multi_index, columns = multi_column).round(1)
>>> df_multi
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222223659638.png#pic_center)

我们可以看到，打印出的表格中增加了索引层数，而且还多了索引的名，红色框表示列索引的名字，蓝色框表示行索引的名字，我们利用代码访问一下试试：

```python
#行索引相关属性
>>> print(df_multi.index)
MultiIndex([('A', 'Female'),
            ('A',   'Male'),
            ('B', 'Female'),
            ('B',   'Male'),
            ('C', 'Female'),
            ('C',   'Male')],
           names=['School', 'Gender'])
>>> print(df_multi.index.names)
['School', 'Gender']
>>> print(df_multi.index.values)
[('A', 'Female') ('A', 'Male') ('B', 'Female') ('B', 'Male')
 ('C', 'Female') ('C', 'Male')]

#列索引相关属性
>>> print(df_multi.columns)
MultiIndex([('Height',  'Freshman'),
            ('Height',    'Senior'),
            ('Height', 'Sophomore'),
            ('Height',    'Junior'),
            ('Weight',  'Freshman'),
            ('Weight',    'Senior'),
            ('Weight', 'Sophomore'),
            ('Weight',    'Junior')],
           names=['Indicator', 'Grade'])
>>> print(df_multi.columns.names)
['Indicator', 'Grade']
>>> print(df_multi.columns.values)
[('Height', 'Freshman') ('Height', 'Senior') ('Height', 'Sophomore')
 ('Height', 'Junior') ('Weight', 'Freshman') ('Weight', 'Senior')
 ('Weight', 'Sophomore') ('Weight', 'Junior')]

#列索引的第二层索引值
print(df_multi.columns.get_level_values(1))
```

在第三个例子中，我们可以通过`columns.get_level_values()方法`去访问指定层的索引值。

### 2.loc索引器在多级索引中的使用

loc索引在多级索引中的使用与单级索引中的使用是有共通性的，我们仍采用上一节的数据举例说明：

这是访问单行数据：

```python
>>> df_multi.loc[('A','Female')]
Indicator  Grade    
Height     Freshman     172.1
           Senior       166.6
           Sophomore    158.6
           Junior       164.7
Weight     Freshman      56.8
           Senior        66.8
           Sophomore     61.5
           Junior        61.3
Name: (A, Female), dtype: float64
```

这是访问多个单数据：

```python
>>> df_multi.loc[[('A','Female'),('C','Male')]]
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222225107924.png#pic_center)

这是访问指定行+列的数据：

```python
>>> df_multi.loc[['A','C'],['Height','Weight']]
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222225654669.png#pic_center)

这里通过布尔列表访问行的操作与单级索引一致，所以不再给出。

### 3.IndexSlice对象

`IndexSlice对象`的作用：

- 可以对多层索引每层进行切片
- 允许在loc中将不同类型的访问方式组合进行使用

主要分为两种使用方式`idx[A,B]`型和`[idx[A,B],idx[C,D]]`型。

怎么理解这两种类型呢，第一种**不能对非第一级以外的行列索引进行局部切片**，而第二种可以。

先定义一个拥有多级索引的DF变量：

```python
>>> np.random.seed(20201222)
>>> L1,L2 = ['A','B','C'],['a','b','c']
>>> mul_index1 = pd.MultiIndex.from_product([L1,L2],names=('Upper', 'Lower'))
>>> L3,L4 = ['D','E','F'],['d','e','f']
>>> mul_index2 = pd.MultiIndex.from_product([L3,L4],names=('Big', 'Small'))
>>> df_ex = pd.DataFrame(np.random.randint(-9,10,(9,9)), index=mul_index1, columns=mul_index2)
>>> df_ex
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222230511530.png#pic_center)

**1）idx\[A,B\]型**

```python
>>> idx = pd.IndexSlice
>>> df_ex.loc[idx['B':,('E','e'):]]
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222230808144.png#pic_center)  
参数中的A在这里指的是`‘B’:`，它表示行索引从B开始一直到结尾；参数中的B在这里指的是`('E','e'):`，他表示从列索引\(E,e\)开始一直到结尾。

注意这里就不能针对列索引的第二级进行局部切片，比如只取第二例的e和f，解决这种问题的方式是第二种类型。

**2）\[idx\[A,B\],idx\[C,D\]\]型**

```python
df_ex.loc[idx[:'B','b':],idx['E':,'e':]]
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222231130410.png#pic_center)

参数中的A在这里指的是`:'B'`，它表示行索引的第一层从开始一直到B；参数中的B在这里指的是`'b':`，他表示行索引的第二层从b开始到第二层的结尾；参数中的C在这里指的是`'E':`，它表示列索引的第一层从E一直到结尾；参数中的D在这里指的是`'e':`，他表示列索引的第二层从e开始到第二层的结尾。

可以看到，利用这种方式就可以针对第二级的列索引进行单独的局部切片，更多例子请看本章习题2-3。

### 4.多级索引的构造

这里的构造用三种方式：

```python
#第一种：利用元组列表进行构建
>>> index_names = ['First','Second']
>>> data_tuple = [('A','a'),('A','b'),('B','a'),('B','b')]
>>> idx = pd.MultiIndex.from_tuples(data_tuple,names=index_names)
>>> print(idx)
MultiIndex([('A', 'a'),
            ('A', 'b'),
            ('B', 'a'),
            ('B', 'b')],
           names=['First', 'Second'])
#第二种：利用双层列表进行构建
>>> data_list = [list('abcd'),list('ABCD')]
>>> idx = pd.MultiIndex.from_arrays(data_list,names=index_names)
>>> print(idx)
MultiIndex([('a', 'A'),
            ('b', 'B'),
            ('c', 'C'),
            ('d', 'D')],
           names=['First', 'Second'])
#第三种：利用多个列表的交叉组合进行构建
>>> idx = pd.MultiIndex.from_product([['A','B'],['a','b']],names=index_names)
>>> print(idx)
MultiIndex([('A', 'a'),
            ('A', 'b'),
            ('B', 'a'),
            ('B', 'b')],
           names=['First', 'Second'])
```

## 三、索引的常用方法

首先先构造一个多级索引的表：

```python
>>> np.random.seed(20201222)
>>> L1,L2,L3 = ['A','B'],['a','b'],['alpha','beta']
>>> mul_index1 = pd.MultiIndex.from_product([L1,L2,L3], names=('Upper', 'Lower','Extra'))
>>> L4,L5,L6 = ['C','D'],['c','d'],['cat','dog']
>>> mul_index2 = pd.MultiIndex.from_product([L4,L5,L6], names=('Big', 'Small', 'Other'))
>>> df_ex = pd.DataFrame(np.random.randint(-9,10,(8,8)), index=mul_index1,  columns=mul_index2)
>>> df_ex
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122223214893.png#pic_center)

### 1.索引层的交换和删除

针对行索引，进行不同层的交换：

```python
>>> df_ex.swaplevel(1,2,axis=0)
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222232542504.png#pic_center)

这里采用`swaplevel()方法`进行交换，前面两个参数表示需要交换的层数，以最外层为0，axis代表需要交换的轴，0代表行，1代表列。

但是这种方法只能交换2层，如果想要交换多层可以使用`reorder_levels()方法`：

```python
>>> df_ex.reorder_levels([2,1,0],axis=1)
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222232906169.png#pic_center)

这里只讲第一个参数，对于三层来说原始的层排列是\[0,1,2\]，第一个参数的作用是用来指定目标层的排列，比如例子里的\[2,1,0\]就是指将列索引的不同层颠倒过来。

我们也可以删除行或列索引中指定的某层索引：

```python
>>> df_ex.droplevel(1,axis=0)
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222233221347.png#pic_center)

### 2.索引属性的修改

我们可以通过`rename_axis`和`rename`修改索引的名和值：

```python
>>> df_ex.rename_axis(index={'Upper':'upper'}, columns={'Other':'other'})
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222233413653.png#pic_center)

```python
>>> df_ex.rename(columns={'dog':'not_dog'}, level=2).head()
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222234528329.png#pic_center)

### 练一练

利用`rename_axis`结合匿名函数对索引层的名字进行修改：

```python
>>> df_ex.rename_axis(index=lambda x:str.upper(x))
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201223185658626.png#pic_center)

### 3.索引的设置与重置

```python
>>> df = pd.read_csv('data/learn_pandas.csv',usecols=['School', 'Grade', 'Name', 'Gender', 'Weight', 'Transfer'])
>>> df.set_index('School')
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222234717306.png#pic_center)

```python
>>> df.set_index('School').reset_index()
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222234804448.png#pic_center)

### 4.索引的变形

新建一个用于演示的DF数据：

```python
>>> df_reindex = pd.DataFrame({"Weight":[60,70,80], "Height":[176,180,179]}, index=['1001','1003','1002'])
>>> df_reindex
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222234854764.png#pic_center)

```python
>>> df_reindex.reindex(index=['1001','1002','1003','2020'], columns=['Weight','School'])
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222234945752.png#pic_center)

```python
>>> df_existed = pd.DataFrame(index=['1001','1002','1003','2020'], columns=['Weight','Loction'])
>>> df_reindex.reindex_like(df_existed)
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222235029326.png#pic_center)

## 四、索引运算

```python
>>> df_set_1 = pd.DataFrame([[0,1],[1,2],[3,4]], index = pd.Index(['a','b','a'],name='id1'))
>>> df_set_2 = pd.DataFrame([[5,1],[6,2],[3,4]], index = pd.Index(['a','c','d'],name='id2'))
>>> id1, id2 = df_set_1.index.unique(), df_set_2.index.unique()
>>> print(id1.intersection(id2))
Index(['a'], dtype='object')
>>> print(id1.union(id2))
Index(['a', 'b', 'c', 'd'], dtype='object')
>>> print(id1.difference(id2))
Index(['b'], dtype='object')
>>> print(id1.symmetric_difference(id2))
Index(['b', 'c', 'd'], dtype='object')
```

分别表示交集、并集、差集、并集减去交集
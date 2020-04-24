  @[TOC](task02-索引)
问题记录：
1、什么是联合索引？
2、这个：： 号是什么意思？
3、函数式索引 lambda 是什么意思？
# 1 Pandas单级索引
loc 方法、iloc方法、[]操作符; 其中iloc表示位置索引，loc表示标签索引，[]也具有很大的便利性，各有特点。
总结来说就是：
	loc只能传布尔列表或索引列表;
	iloc只能传整数列表;
loc只能传布尔列表或索引列表
iloc只能传整数列表
## 1.1 loc方法 iloc方法和[]操作符
### 1.1.1 loc方法
本质上来说，loc中能传入的只有布尔列表和索引子集构成的列表。
**loc方法包含切片右端点**
代码示例：
``` python
# 单行索引
dataframe.loc[index]
# 多行索引
dataframe.loc[index1, index2]
dataframe.loc[index: ]
dataframe.loc[开始:结束:步长]
# 多列索引
dataframe[:, '列索引名']
# 函数式索引（传入的参数是dataframe）利用loc函数，根据某个数据来提取数据所在的行
dataframe.loc[lambda x:x['Gender']=='M']

#这里的例子表示，loc中能够传入函数，并且函数的输入值是整张表，输出为标量、切片、合法列表（元素出现在索引中）、合法索引
def f(x):
    return [1101,1103]
df.loc[f]
# 布尔索引  
data.loc[data['A']==0] #提取data数据(筛选条件: A列中数字为0所在的行数据)
Out[10]: 
   A  B  C  D
a  0  1  2  3
In[12]: data[data['A']==0] #dataframe用法
In[13]: data[data['A'].isin([0])] #isin函数
In[14]: data[(data['A']==0)&(data['B']==2)] #dataframe用法
In[15]: data[(data['A'].isin([0]))&(data['B'].isin([2]))] #isin函数
```

## 1.1.2 iloc方法
iloc中接收的参数只能为整数或整数列表，不能使用布尔索引；
注意与loc不同，切片右端点不包含；
代码示例：
```python
# 这几种常见方法
# 单行索引
df.iloc[3]
#多行索引
df.iloc[3:5]
# 单列索引
df.iloc[:,3]
# 多列索引
data.loc[['a','b'],['A','B']] # 取多行和多列
data.iloc[[0,1],[0,1]] # 整数列表
df.iloc[:,7::-2] #这怎么表示的索引呢？
# 混合索引
df.iloc[3::4,7::-2]
# 函数式索引
df.iloc[lambda x:[3]]

```
小结：小节：iloc中接收的参数只能为整数或整数列表或布尔列表，不能使用布尔Series，如果要用就必须如下把values拿出来；
# 1.1.3 []操作符
在Series中的浮点[]并不是进行位置比较，而是值比较，因此不要在行索引为浮点时使用[]操作符。
[]操作符常用于列选择或布尔选择，尽量避免行的选择;

## 1.3.1 Series的[]操作符
代码示例
```python
# 单元数索引
s = pd.Series(df['Math'],index=df.index) # 传入参数，获得数据
s[1101]
# 多行索引
s[0:4]
# 函数式索引
##注意使用lambda函数时，直接切片(如：s[lambda x: 16::-6])就报错，此时使用的不是绝对位置切片，而是元素切片，非常易错
s[lambda x: x.index[1222::-6]]
# 布尔索引
s[s>80]
# 如果不想陷入困境，请不要在行索引为浮点时使用[]操作符；
因为在Series中[]的浮点切片并不是进行位置比较，而是值比较，非常特殊
## 自己制作的数据，做个测试
s_int = pd.Series([1,2,3,4],index=[1,3,5,6])
s_float = pd.Series([1,2,3,4],index=[1.,3.,5.,6.])
s_int
s_int[2:]   # 位置下标索引
s_float
s_float[2:]  # 注意和s_int[2:]结果不一样了，因为2这里 是元素值而不是位置
## 1.3.2 DataFrame的[]操作
代码示例：
```python
# 单行索引  这个是左闭右开
df[1:3]
# 使用get_loc获得行
row = df.index.get_loc(1102)
df[row:row+1]
# 多行索引 用切片，如果是选取指定的某几行，推荐使用loc，否则很可能报错
df[3:5]
# 单列索引
df['School'].head()
# 多列索引
df[['Height','Weight']].head()
# 函数式索引
df[lambda x:['Math','Physics']].head()
# 布尔索引
df[df['Gender']=='F'].head()

```
**小节：一般来说，[]操作符常用于列选择或布尔选择，尽量避免行的选择**

## 1.2 布尔索引
布尔符号：'&','|','~'：分别代表和and，或or，取反not；
```python
# 性别为男 & 在2号街道
df[(df['Gender']=='F')&(df['Address']=='street_2')].head()
# 数学成绩>85 或者 地址在7号街道
df[(df['Math']>85)|(df['Address']=='street_7')].head()
# 布尔逻辑的运算，即数学成绩小于75 & 地址在第一街道
df[~((df['Math']>75)|(df['Address']=='street_1'))].head()
# 布尔列表在loc和[]中相应位置的使用
# 去成绩大于60的同学的物理成绩
df.loc[df['Math']>60,df.columns=='Physics'].head()
#  isin+列表 ，在列表里面是列表的成员
df[df['Address'].isin(['street_1','street_4'])&df['Physics'].isin(['A','A+'])]

```
## 1.3 快速标量索引
at和iat方法，适用于只取一个元素的情况。
同样，at只能传布尔列表或索引列表，iat只能传整数列表
df.at[1101,'School']
df.iat[0,0]

## 1.4 区间索引
此处介绍并不是说只能在单级索引中使用区间索引，只是作为一种特殊类型的索引方式，只是在此处先行介绍而已。
### 1.4.1 interval_range方法
```python
# 利用interval_range方法
pd.interval_range(start=0,end=5)
#periods参数控制区间个数，freq控制步长
pd.interval_range(start=0,periods=8,freq=5)
```
### 1.4.2 利用cut将数值列转为区间为元素的分类变量
例如统计数学成绩的区间情况
```
math_interval = pd.cut(df['Math'],bins=[0,40,60,80,100])
#注意，如果没有类型转换，此时并不是区间类型，而是category类型
math_interval.head()
```
### 1.4.3 区间索引的选取
```
df_i = df.join(math_interval,rsuffix='_interval')[['Math','Math_interval']]\
            .reset_index().set_index('Math_interval')
df_i.head()
# 包含该值就会被选中
df_i.loc[65].head()
df_i.loc[[65,90]].head()
# 如果想要选取某个区间，先要把分类变量转为区间变量，再使用overlap方法：
#df_i.loc[pd.Interval(70,75)].head() 报错
df_i[df_i.index.astype('interval').overlaps(pd.Interval(70, 85))].head()
#只要索引与(70,85]这个区间有交集就会被选中，注意pd.Interval默认构造区间都是左开右闭，

```
# 2 多级索引
## 2.1 创建多级索引
1、通过from_tuple或from_arrays
```python
# 直接创建元组
tuples = [('A','a'),('A','b'),('B','a'),('B','b')]
mul_index = pd.MultiIndex.from_tuples(tuples, names=('Upper', 'Lower'))
mul_index
pd.DataFrame({'Score':['perfect','good','fair','bad']},index=mul_index)
# 利用zip创建元组
L1 = list('AABB')
L2 = list('abab')
tuples = list(zip(L1,L2))
mul_index = pd.MultiIndex.from_tuples(tuples, names=('Upper', 'Lower'))
pd.DataFrame({'Score':['perfect','good','fair','bad']},index=mul_index)
# 通过array创建，基本类型里面没有数组，是不是第三方库里面有数组?
arrays = [['A','a'],['A','b'],['B','a'],['B','b']]
mul_index = pd.MultiIndex.from_tuples(arrays, names=('Upper', 'Lower'))
pd.DataFrame({'Score':['perfect','good','fair','bad']},index=mul_index)
mul_index #由此看出内部自动转成元组
```
2、通过from_product
```python
L1 = ['A','B']
L2 = ['a','b']
pd.MultiIndex.from_product([L1,L2],names=('Upper', 'Lower')) #两两相乘

````
3、指定df中的列创建（set_index方法）
```
df_using_mul = df.set_index(['Class','Address'])
df_using_mul.head()
```
## 2.2 多层索引切片
这里举例都是用上一小节的df_using_mul做演示
df_using_mul.head()
### 2.2.1 一般切片
```python
#df_using_mul.loc['C_2','street_5']
#当索引不排序时，单个索引会报出性能警告
#df_using_mul.index.is_lexsorted()
#该函数检查是否排序
df_using_mul.sort_index().loc['C_2','street_5']

#df_using_mul.sort_index().index.is_lexsorted()
#df_using_mul.loc[('C_2','street_5'):] 报错
#当不排序时，不能使用多层切片
df_using_mul.sort_index().loc[('C_2','street_6'):('C_3','street_4')] #注意此处由于使用了loc，因此仍然包含右端点
df_using_mul.sort_index().loc[('C_2','street_7'):'C_3'].head() #非元组也是合法的，表示选中该层所有元素
```
### 2.2.2 第一类特殊情况：由元组构成列表
 #表示选出某几个元素，精确到最内层索引
df_using_mul.sort_index().loc[[('C_2','street_7'),('C_3','street_2')]] 

### 2.2.3 第二类特殊情况：由列表构成元组
#选出第一层在‘C_2’和'C_3'中且第二层在'street_4'和'street_7'中的行
df_using_mul.sort_index().loc[(['C_2','C_3'],['street_4','street_7']),:]

## 2.3 多层索引中的slice对象
```python
L1,L2 = ['A','B'],['a','b','c']
mul_index1 = pd.MultiIndex.from_product([L1,L2],names=('Upper', 'Lower'))
L3,L4 = ['D','E','F'],['d','e','f']
mul_index2 = pd.MultiIndex.from_product([L3,L4],names=('Big', 'Small'))
df_s = pd.DataFrame(np.random.rand(6,9),index=mul_index1,columns=mul_index2)
df_s

idx=pd.IndexSlice # IndexSlice本质上是对多个Slice对象的包装
idx[1:9:2,'A':'C','start':'end':2]

```
## 2.3.1 **索引Slice可以与loc一起完成切片操作，主要有两种用法:**
1、Ioc[idx[*,*]]型
第一个星号表示行，第二个表示列，且使用布尔索引时，需要索引对齐;
```
#例子1
df_s.loc[idx['B':,df_s.iloc[0]>0.6]]
#df_s.loc[idx['B':,df_s.iloc[:,0]>0.6]] #索引没有对齐报错
#例子2
df_s.loc[idx[df_s.iloc[:,0]>0.6,:('E','f')]]
```
2、 oc[idx[*,*],idx[*,*]]型
这里与上面的区别在于（a）中的loc是没有逗号隔开的，但（b）是用逗号隔开，前面一个idx表示行索引，后面一个idx为列索引；
这种用法非常灵活
```python
#例子1
df_s.loc[idx['A'],idx['D':]] #后面的层出现，则前面的层必须出现
#df_s.loc[idx['a'],idx['D':]] #报错
#例子2
df_s.loc[idx[:'B','b':],:] #举这个例子是为了说明①可以在相应level使用切片②某一个idx可以用:代替表示全选
#例子3
df_s.iloc[:,0]>0.6
df_s.loc[idx[:'B',df_s.iloc[:,0]>0.6],:] #这个例子表示相应位置还可以使用布尔索引
#例子4
#特别要注意，（b）中的布尔索引是可以索引不对齐的，只需要长度一样，比如下面这个例子
df_s.loc[idx[:'B',(df_s.iloc[0]>0.6)[:6]],:]
#例子5
df_s.loc[idx[:'B','c':,(df_s.iloc[:,0]>0.6)],:]
#idx中层数k1大于df层数k2时，idx前k2个参数若相应位置是元素或者元素切片，则表示相应df层的元素筛选，同时也可以选择用同长度bool序列
#idx后面多出来的参数只能选择同bool序列，这样设计的目的是可以将元素筛选和条件筛选同时运用
#例子6
df_s.loc[idx[:'B',(df_s.iloc[:,0]>0.6),(df_s.iloc[:,0]>0.6)],:] #这个就不是元素筛选而是条件筛选
#df_s.loc[idx[:'B',(df_s.iloc[:,0]>0.6),'c',:]] #报错
#df_s.loc[idx[:'c','B',(df_s.iloc[:,0]>0.6),:]] #报错

```
## 2.4 索引层的交换
### 2.4.1 swaplevel(两层交换)
df_using_mul.head()
df_using_mul.swaplevel(i=1,j=0,axis=0).sort_index().head()

### 2.4.2 reorder_levels方法（多层交换）
df_muls = df.set_index(['School','Class','Address'])
df_muls.head()
df_muls.reorder_levels([2,0,1],axis=0).sort_index().head()
#如果索引有name，可以直接使用name
df_muls.reorder_levels(['Address','School','Class'],axis=0).sort_index().head()

# 3 索引设定
## 3.1 index_col 参数
index_col是read_csv中的一个参数，而不是某一个方法：
```
pd.read_csv('data/table.csv',index_col=['Address','School']).head()
```
## 3.2 reindex和reindex_like
reindex是指重新索引，它的重要特性在于索引对齐，很多时候用于重新排序
```
df.head()
df.reindex(index=[1101,1203,1206,2402])
df.reindex(columns=['Height','Gender','Average']).head()

# 可以选择缺失值的填充方法：fill_value和method（bfill/ffill/nearest），其中method参数必须索引单调
df.reindex(index=[1101,1203,1206,2402],method='bfill') #这里的单调是指df必须索引经过排序，否则报错
#bfill表示用所在索引1206的后一个有效行填充，ffill为前一个有效行，nearest是指最近的

df.reindex(index=[1101,1203,1206,2402],method='nearest') #数值上1205比1301更接近1206，因此用前者填充

# reindex_like的作用为生成一个横纵索引完全与参数列表一致的DataFrame，数据使用被调用的表
df_temp = pd.DataFrame({'Weight':np.zeros(5),
                        'Height':np.zeros(5),
                        'ID':[1101,1104,1103,1106,1102]}).set_index('ID')
df_temp.reindex_like(df[0:5][['Weight','Height']])

# 如果df_temp单调还可以使用method参数：
df_temp = pd.DataFrame({'Weight':range(5),
                        'Height':range(5),
                        'ID':[1101,1104,1103,1106,1102]}).set_index('ID').sort_index()
df_temp.reindex_like(df[0:5][['Weight','Height']],method='bfill')
# 可以自行检验这里的1105的值是否是由bfill规则填充

## 3.3 set_index和reset_index
### 3.3.1 set_index方法
先介绍set_index：从字面意思看，就是将某些列作为索引
```python
df.head()
df.set_index('Class').head()
# 利用append参数可以将当前索引维持不变
df.set_index('Class',append=True).head()
# 当使用与表长相同的列作为索引（需要先转化为Series，否则报错）：
df.set_index(pd.Series(range(df.shape[0]))).head()
# 可以直接添加多级索引：
df.set_index([pd.Series(range(df.shape[0])),pd.Series(np.ones(df.shape[0]))]).head()

### 3.3.2 介绍reset_index方法
它的主要功能是将索引重置；
```
# 默认状态直接恢复到自然数索引：
df.reset_index().head()
# 用level参数指定哪一层被reset，用col_level参数指定set到哪一层：
L1,L2 = ['A','B','C'],['a','b','c']
mul_index1 = pd.MultiIndex.from_product([L1,L2],names=('Upper', 'Lower'))
L3,L4 = ['D','E','F'],['d','e','f']
mul_index2 = pd.MultiIndex.from_product([L3,L4],names=('Big', 'Small'))
df_temp = pd.DataFrame(np.random.rand(9,9),index=mul_index1,columns=mul_index2)
df_temp.head()

df_temp1 = df_temp.reset_index(level=1,col_level=1)
df_temp1.head()

df_temp1.columns #看到的确插入了level2
df_temp1.index #最内层索引被移出

## 3.4 rename_axis和rename
rename_axis是针对多级索引的方法，作用是修改某一层的索引名，而不是索引标签;
df_temp.rename_axis(index={'Lower':'LowerLower'},columns={'Big':'BigBig'})
rename方法用于修改列或者行索引标签，而不是索引名：
df_temp.rename(index={'A':'T'},columns={'e':'changed_e'}).head()

# 4 常用索引型函数
## 4.1 where函数
```
当对条件为False的单元进行填充：
df.head()
#不满足条件的行全部被设置为NaN
df.where(df['Gender']=='M').head()
#通过这种方法筛选结果和[]操作符的结果完全一致：
df.where(df['Gender']=='M').dropna().head()
# 第一个参数为布尔条件，第二个参数为填充值：
df.where(df['Gender']=='M',np.random.rand(df.shape[0],df.shape[1])).head()

```
## 4.2 mask 函数
mask函数与where功能上相反，其余完全一致，即对条件为True的单元进行填充
df.mask(df['Gender']=='M').dropna().head()
df.mask(df['Gender']=='M',np.random.rand(df.shape[0],df.shape[1])).head()

## 4.3 query函数
```
df.head()
# query函数中的布尔表达式中，下面的符号都是合法的：行列索引名、字符串、and/not/or/&/|/
#  ~/not in/in/==/!=、四则运算符
df.query('(Address in ["street_6","street_7"])&(Weight>(70+10))&(ID in [1303,2304,2402])')

# 5 重复元素处理
## 5.1 duplicated方法
该方法返回了是否重复的布尔列表
```
df.duplicated('Class').head()
# 可选参数keep默认为first，即首次出现设为不重复，若为last，则最后一次设为不重复，若为
# False，则所有重复项为True
df.duplicated('Class',keep='last').tail()
df.duplicated('Class',keep=False).head()

```
## 5.2 drop_duplicates方法
从名字上看出为剔除重复项，这在后面章节中的分组操作中可能是有用的，例如需要保留每组的第一个值
```
df.drop_duplicates('Class')
参数与duplicate函数类似：
df.drop_duplicates('Class',keep='last')
在传入多列时等价于将多列共同视作一个多级索引，比较重复项:
df.drop_duplicates(['School','Class'])
```
# 6 抽样函数
这里的抽样函数指的就是sample函数
```
#（a）n为样本量
df.sample(n=5)
# (b）frac为抽样比
df.sample(frac=0.05)
# (c）replace为是否放回
df.sample(n=df.shape[0],replace=True).head()
df.sample(n=35,replace=True).index.is_unique
#（d）axis为抽样维度，默认为0，即抽行
df.sample(n=3,axis=1).head()
# (e) weights为样本权重，自动归一化
df.sample(n=3,weights=np.random.rand(df.shape[0])).head()
#以某一列为权重，这在抽样理论中很常见
#抽到的概率与Math数值成正比
df.sample(n=3,weights=df['Math']).head()


@[TOC](Pandas分组)
# 1 SAC过程
## 1.1 内涵
SAC指的是分组操作中的split-apply-combine过程；
其中split指基于某一些规则，将数据拆成若干组，apply是指对每一组独立地使用函数，combine指将每一组的结果组合成某一类数据结构；
## 1.2 apply过程
在该过程中，我们实际往往会遇到四类问题：
整合（Aggregation）——即分组计算统计量（如求均值、求每组元素个数）
变换（Transformation）——即分组对每个单元的数据进行操作（如元素标准化）
过滤（Filtration）——即按照某些规则筛选出一些组（如选出组内某一指标小于50的组）
综合问题——即前面提及的三种问题的混合
# 2 groupby函数
## 2.1 分组函数的基本内容
```python
# 1 根据某一列分组
grouped_single = df.groupby('School')
# 经过groupby后会生成一个groupby对象，该对象本身不会返回任何东西，只有当相应的方法被调
# 用才会起作用

# 2 例如取出某一个组
grouped_single.get_group('S_1').head()
# 3 根据某几列分组
grouped_mul=df.groupby(['School','Class'])
grouped_mul.get_group(('S_2','C_4'))
# 4 组容量与组数
grouped_single.size()
grouped_mul.size()
grouped_single.ngroups
grouped_mul.ngroups
# 5 组的遍历
for name,group in grouped_single:
    print(name)
    display(group.head())
# 6 level参数（用于多级索引）和axis参数
df.set_index(['Gender','School']).groupby(level=1,axis=0).get_group('S_1').head()

```
## 2.2 groubhy对象的特点
```python
# 1 查看所有可调用的方法
# 由此可见，groupby对象可以使用相当多的函数，灵活程度很高
print([attr for attr in dir(grouped_single) if not attr.startswith('_')])# 这是查看函数和属性的小技巧
# 2 分组对象的head和first
# 对分组对象使用head函数，返回的是每个组的前几行，而不是数据集前几行
grouped_single.head(2)
# first显示的是以分组为索引的每组的第一个分组信息
grouped_single.first()

# %% 3 分组依据
# 对于groupby函数而言，分组的依据是非常自由的，
# 只要是与数据框长度相同的列表即可，同时支持函数型分组¶
df.groupby(np.random.choice(['a','b','c'],df.shape[0])).get_group('a').head() 
#相当于将np.random.choice(['a','b','c'],df.shape[0])当做新的一列进行分组
#从原理上说，我们可以看到利用函数时，传入的对象就是索引，
#因此根据这一特性可以做一些复杂的操作，单行函数表达式
df[:5].groupby(lambda x:print(x)).head(0)
# 根据奇偶行分组
df.groupby(lambda x:'奇数行' if not df.index.get_loc(x)%2==1 else '偶数行').groups
# 如果是多层索引，那么lambda表达式中的输入就是元组
# 下面实现的功能为查看两所学校中男女生分别均分是否及格
# 注意：此处只是演示groupby的用法，实际操作不会这样写
math_score = df.set_index(['Gender','School'])['Math'].sort_index()
grouped_score = df.set_index(['Gender','School']).sort_index().\
            groupby(lambda x:(x,'均分及格' if math_score[x].mean()>=60 else '均分不及格'))
for name,_ in grouped_score:
            print(name)

# %% 4 groupby的[]操作
# 可以用[]选出groupby对象的某个或者某几个列，上面的均分比较可以如下简洁地写出：
df.groupby(['Gender','School'])['Math'].mean()>=60
# 用列表可选出多个属性列;
df.groupby(['Gender','School'])[['Math','Height']].mean()
# 5(e)连续型变量分组
# 例如利用cut函数对数学成绩分组：
bins = [0,40,60,80,90,100]
cuts = pd.cut(df['Math'],bins=bins) #可选label添加自定义标签
df.groupby(cuts)['Math'].count()
```
# 3 聚合、过滤和变换
## 3.1 聚合
### 3.1.1 常用聚合函数
所谓聚合就是把一堆数，变成一个标量，因此mean/sum/size/count/std/var/sem/describe/first/last/nth/min/max都是聚合函数
```python
# 1 为了熟悉操作，不妨验证标准误sem函数，下面进行验证：
group_m = grouped_single['Math']
group_m.std().values/np.sqrt(group_m.count().values)== group_m.sem().values 
# 2 同时使用多个聚合函数
group_m.agg(['sum','mean','std'])
# 利用元组进行重命名
group_m.agg([('rename_sum','sum'),('rename_mean','mean')])
# 指定哪些函数作用哪些列
grouped_mul.agg({'Math':['mean','max'],'Height':'var'})
# 3 使用自定义函数
grouped_single['Math'].agg(lambda x:print(x.head(),'间隔')) #可以发现，agg函数的传入是分组逐列进行的，有了这个特性就可以做许多事情
# 官方没有提供极差计算的函数，但通过agg可以容易地实现组内极差计算
grouped_single['Math'].agg(lambda x:x.max()-x.min())
# 4 利用NamedAgg函数进行多个聚合
# 注意：不支持lambda函数，但是可以使用外置的def函数
def R1(x):
    return x.max()-x.min()
def R2(x):
    return x.max()-x.median()
grouped_single['Math'].agg(min_score1=pd.NamedAgg(column='col1', aggfunc=R1),
                           max_score1=pd.NamedAgg(column='col2', aggfunc='max'),
                           range_score2=pd.NamedAgg(column='col3', aggfunc=R2)).head()
# 5 带参数的聚合函数
# 判断是否组内数学分数至少有一个值在50-52之间：
def f(s,low,high):
    return s.between(low,high).max()
grouped_single['Math'].agg(f,50,52)
# 如果需要使用多个函数，并且其中至少有一个带参数，则使用wrap技巧：
def f_test(s,low,high):
    return s.between(low,high).max()
def agg_f(f_mul,name,*args,**kwargs):
    def wrapper(x):
        return f_mul(x,*args,**kwargs)
    wrapper.__name__ = name
    return wrapper
new_f = agg_f(f_test,'at_least_one_in_50_52',50,52)
grouped_single['Math'].agg([new_f,'mean']).head()

```
## 3.2 过滤(Filterration)
filter函数是用来筛选某些组的（务必记住结果是组的全体），因此传入的值应当是布尔标量
grouped_single[['Math','Physics']].filter(lambda x:(x['Math']>32).all()).head()

## 3.3 变换(Transform)
```python
# 1 传入对象
# transform函数中传入的对象是组内的列，并且返回值需要与列长完全一致
grouped_single[['Math','Height']].transform(lambda x:x-x.min()).head()
# 如果返回了标量值，那么组内的所有元素会被广播为这个值
grouped_single[['Math','Height']].transform(lambda x:x.mean()).head()
# 2 利用变换方法进行组内标准化
grouped_single[['Math','Height']].transform(lambda x:(x-x.mean())/x.std()).head()
# 3 利用变换方法进行组内缺失值的均值填充
df_nan = df[['Math','School']].copy().reset_index()
df_nan.loc[np.random.randint(0,df.shape[0],25),['Math']]=np.nan
df_nan.head()
df_nan.groupby('School').transform(lambda x: x.fillna(x.mean())).join(df.reset_index()['School']).head()

```
# 4 apply函数
## 4.1 apply函数的灵活性
可能在所有的分组函数中，apply是应用最为广泛的，这得益于它的灵活性：返回值
```python
# 对于传入值而言，从下面的打印内容可以看到是以分组的表传入apply中：
df.groupby('School').apply(lambda x:print(x.head(1)))
# 函数的灵活性很大程度上来源于返回值的多样性
# 1 标量返回值
df[['School','Math','Height']].groupby('School').apply(lambda x:x.max())
# 2 列表返回值
df[['School','Math','Height']].groupby('School').apply(lambda x:x-x.min()).head()
# 3 数据框返回值
df[['School','Math','Height']].groupby('School')\ # 这个斜杆表示逻辑行继续
    .apply(lambda x:pd.DataFrame({'col1':x['Math']-x['Math'].max(),
                                  'col2':x['Math']-x['Math'].min(),
                                  'col3':x['Height']-x['Height'].max(),
                                  'col4':x['Height']-x['Height'].min()})).head()
```
## 4.2 用apply同时统计多个指标
```python
此处可以借助OrderedDict工具进行快捷的统计：
from collections import OrderedDict
def f(df):
    data = OrderedDict()
    data['M_sum'] = df['Math'].sum()
    data['W_var'] = df['Weight'].var()
    data['H_mean'] = df['Height'].mean()
    return pd.Series(data)
grouped_single.apply(f)
```
# 5 问题与练习

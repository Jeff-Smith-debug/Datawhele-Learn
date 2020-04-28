@[TOC](S4-变形)
我不太理解变形 指的是什么意思？

## 4.1 透视表
### 4.1.1 pivot
一般状态下，数据在DataFrame会以压缩（stacked）状态存放，例如上面的Gender，两个类别被叠在一列中，pivot函数可将某一列作为新的cols：
```python
df.pivot(index='ID',columns='Gender',values='Height').head()
# 然而pivot函数具有很强的局限性，除了功能上较少之外，
# 还不允许values中出现重复的行列索引对（pair），例如下面的语句就会报错：
#df.pivot(index='School',columns='Gender',values='Height').head()
```
### 4.1.2 pivot_table
更多的时候会选择使用强大的pivot_table函数。
pd.pivot_table(df,index='ID',columns='Gender',values='Height').head()
```python
# Pandas中提供了各种选项，下面介绍常用参数：
# 1 aggfunc：对组内进行聚合统计，可传入各类函数，默认为'mean'
pd.pivot_table(df,index='School',columns='Gender',values='Height',aggfunc=['mean','sum']).head()
# 2 margins:汇总边际状态
pd.pivot_table(df,index='ID',columns='Gender',values='Height',aggfunc=['mean','sum'],margins=True).head()
# 3  行、列、值都可以为多级
pd.pivot_table(df,index=['School','Class'],
                  columns=['Gender','Address'],
                    values=['Height','Weight']
              )

```
### 4.1.3 crosstab(交叉表)
交叉表是一种特殊的透视表，典型的用途如分组统计，如现在想要统计关于街道和性别分组的频数：
pd.crosstab(index=df['Address'],columns=df['Gender'])
```python
# 1 values和aggfunc：分组对某些数据进行聚合操作，这两个参数必须成对出现
pd.crosstab(index=df['Address'],columns=df['Gender'],
            values=np.random.randint(1,20,df.shape[0]),aggfunc='min')
#默认参数等于如下方法：
#pd.crosstab(index=df['Address'],columns=df['Gender'],values=1,aggfunc='count')
# 2 除了边际参数margins外，还引入了normalize参数，
pd.crosstab(index=df['Address'],columns=df['Gender'],normalize='all',margins=True)
```
## 4.2 其他变形方法
### 4.2.1 melt
melt函数可以认为是pivot函数的逆操作，将unstacked状态的数据，压缩成stacked，使“宽”的DataFrame变“窄”。
```python
df_m=df[['ID','Gender','Math']]
df_m.head()
df.pivot(index='ID',columns='Gender',values='Math').head()
# melt函数中的id_vars表示需要保留的列，value_vars表示需要stack的一组列
pivoted = df.pivot(index='ID',columns='Gender',values='Math')
result = pivoted.reset_index().melt(id_vars=['ID'],value_vars=['F','M'],value_name='Math')\
                     .dropna().set_index('ID').sort_index()
#检验是否与展开前的df相同，可以分别将这些链式方法的中间步骤展开，看看是什么结果
result.equals(df_m.set_index('ID'))
```
### 4.2.2 压缩与展开
```python
# 1 stack：这是最基础的变形函数，总共只有两个参数：level和dropna
df_s = pd.pivot_table(df,index=['Class','ID'],columns='Gender',values=['Height','Weight'])
df_s.groupby('Class').head(2)
df_stacked = df_s.stack()
df_stacked.groupby('Class').head(2)
# stack函数可以看做将横向的索引放到纵向，因此功能类似与melt，参数level可指定变化的列索引是哪一层（或哪几层，需要列表）
df_stacked = df_s.stack(0)
df_stacked.groupby('Class').head(2)

# 2 unstack：stack的逆函数，功能上类似于pivot_table
df_stacked.head()
result = df_stacked.unstack().swaplevel(1,0,axis=1).sort_index(axis=1)
result.equals(df_s)
#同样在unstack中可以指定level参数

```
## 4.3 哑变量与因子化
```python
# 1 Dummy Variable（哑变量）
# 这里主要介绍get_dummies函数，其功能主要是进行one-hot编码
df_d = df[['Class','Gender','Weight']]
df_d.head()
# 现在希望将上面的表格前两列转化为哑变量，并加入第三列Weight数值：
pd.get_dummies(df_d[['Class','Gender']]).join(df_d['Weight']).head()
#可选prefix参数添加前缀，prefix_sep添加分隔符
# 2 factorize方法
# 该方法主要用于自然数编码，并且缺失值会被记做-1，其中sort参数表示是否排序后赋值
codes,uniques=pd.factorize(['b','None','a','c','b'],sort=True)
display(codes)
display(uniques)
```
## 4.4 问题与练习


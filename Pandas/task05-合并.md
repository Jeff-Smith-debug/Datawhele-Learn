@[Pandas](合并)
# 1 append和assgin方法
## 1.1 append方法
```python
# 1 利用序列添加行（必须制定name）
df_append=df.loc[0:3,['Height','Gender']].copy()
df_append
s=pd.Series({'Gender':'F','Height':'188'},name='new_col')
df_append.append(s)
# 2 用dataframe添加表
df_temp=pd.DataFrame({'Gender':['F','m'],'Height':[188,176]},index=['new_1','new_2'])
df_append.append(s)
```
## 1.2 assgin方法
```python
# 该方法主要是用于添加列，列名直接由参数制指定
s=pd.Series(list('abcd'),index=range(4))
df_append.assign(Letter=s)
# 一次添加多个列
df_append.assign(col1=lambda x:x['Gender']*2,col2=s )
```
# 2 combine和update
combine和update都是用于表的填充函数，可以根据某种规则填充。
## 2.1 combine方法

```python
# 1 填充对象
# combine方法是按照表的顺序轮流进行逐列循环的，而且自动索引对齐，缺失值为NaN，理解这一点很重要
df_combine_1=df.loc[:1,['Gender','Height']].copy()
df_combine_2=df.loc[10:11,['Gender','Height']].copy()
df_combine_1.combine(df_combine_2,lambda x,y:print(x,y))
# 2 例子
# 根据列均值的大小填充
df1=pd.DataFrame({'A':[1,2],'B':[3,4]})
df2=pd.DataFrame({'A':[8,7],'B':[6,5]})
df1.combine(df2,lambda x,y:x if x.mean()>y.mean() else y)


# combine_first方法
这个方法作用是用df2填补df1的缺失值，功能比较简单，但很多时候会比combine更常用，下面举两个例子：


```
## 2.2 update方法
三个特点：
①返回的框索引只会与被调用框的一致（默认使用左连接，下一节会介绍）
②第二个框中的nan元素不会起作用
③没有返回值，直接在df上操作
```python
# 1 索引完全对齐的情况下操作
df1=pd.DataFrame({'A':[1,2,3],'B':[100,200,300]})        
df2 = pd.DataFrame({'B': [4, 5, 6],'C': [7, 8, 9]})
df1.update(df2)
df1
# 2 部分填充
df1 = pd.DataFrame({'A': ['a', 'b', 'c'],
                    'B': ['x', 'y', 'z']})
df2 = pd.DataFrame({'B': ['d', 'e']}, index=[1,2])
df1.update(df2)
df1
# 3 缺失值不会填充
df1 = pd.DataFrame({'A': [1, 2, 3],
                    'B': [400, 500, 600]})
df2 = pd.DataFrame({'B': [4, np.nan, 6]})
df1.update(df2)
df1
```
## 2.3 concat方法
concat方法可以在两个维度上拼接，默认纵向凭借（axis=0），拼接方式默认外连接 所谓外连接，就是取拼接方向的并集，而'inner'时取拼接方向（若使用默认的纵向拼接，则为列的交集）的交集。
```python
df1 = pd.DataFrame({'A': ['A0', 'A1'],
                    'B': ['B0', 'B1']},
                    index = [0,1])
df2 = pd.DataFrame({'A': ['A2', 'A3'],
                    'B': ['B2', 'B3']},
                    index = [2,3])
df3 = pd.DataFrame({'A': ['A1', 'A3'],
                    'D': ['D1', 'D3'],
                    'E': ['E1', 'E3']},
                    index = [1,3])
# 默认状态拼接
pd.concat([df1,df2])
# axis=1时沿列方向拼接：
pd.concat([df1,df2],axis=1)
# join设置为内连接（由于axis=0，因此列取交集）
pd.concat([df1,df2],join='inner')
# join设置为外链接
pd.concat([df3,df1],join='outer',sort=True) #sort设置列排序，默认为False
# verify_integrity检查列是否唯一
#pd.concat([df3,df1],verify_integrity=True,sort=True) 报错
# 同样，可以添加Series
s = pd.Series(['X0', 'X1'], name='X')
pd.concat([df1,s],axis=1)
# key参数用于对不同的数据框增加一个标号，便于索引：
pd.concat([df1,df2], keys=['x', 'y'])
pd.concat([df1,df2], keys=['x', 'y']).index
```
# 4 merge和join函数
## 4.1 merge函数
merge函数的作用是将两个pandas对象横向合并，遇到重复的索引项时会使用笛卡尔积，默认inner连接，可选left、outer、right连接。所谓左连接，就是指以第一个表索引为基准，右边的表中如果不再左边的则不加入，如果在左边的就以笛卡尔积的方式加入merge/join与concat的不同之处在于on参数，可以指定某一个对象为key来进行连接。
```python
left = pd.DataFrame({'key1': ['K0', 'K0', 'K1', 'K2'],
                     'key2': ['K0', 'K1', 'K0', 'K1'],
                      'A': ['A0', 'A1', 'A2', 'A3'],
                      'B': ['B0', 'B1', 'B2', 'B3']}) 
right = pd.DataFrame({'key1': ['K0', 'K1', 'K1', 'K2'],
                      'key2': ['K0', 'K0', 'K0', 'K0'],
                      'C': ['C0', 'C1', 'C2', 'C3'],
                      'D': ['D0', 'D1', 'D2', 'D3']})
right2 = pd.DataFrame({'key1': ['K0', 'K1', 'K1', 'K2'],
                      'key2': ['K0', 'K0', 'K0', 'K0'],
                      'C': ['C0', 'C1', 'C2', 'C3']})
# 以key1为准则连接，如果具有相同的列，则默认suffixes=('_x','_y')：
pd.merge(left, right, on='key1')
# 以多组键连接：
pd.merge(left, right, on=['key1','key2'])
# 默认使用inner连接，因为merge只能横向拼接，所以取行向上keys的交集，下面看如果使用how=outer参数¶
# 注意：这里的how就是concat的join
pd.merge(left, right, how='outer', on=['key1','key2'])
# 左连接：
pd.merge(left, right, how='left', on=['key1', 'key2'])
# 右连接：
pd.merge(left, right, how='right', on=['key1', 'key2'])
# 如果还是对笛卡尔积不太了解，请务必理解下面这个例子，由于B的所有元素为2，因此需要6行：
left = pd.DataFrame({'A': [1, 2], 'B': [2, 2]})
right = pd.DataFrame({'A': [4, 5, 6], 'B': [2, 2, 2]})
pd.merge(left, right, on='B', how='outer')
# validate检验的是到底哪一边出现了重复索引，如果是“one_to_one”则两侧索引都是唯一，如果"one_to_many"则左侧唯一
left = pd.DataFrame({'A': [1, 2], 'B': [2, 2]})
right = pd.DataFrame({'A': [4, 5, 6], 'B': [2, 3, 4]})
#pd.merge(left, right, on='B', how='outer',validate='one_to_one') #报错
left = pd.DataFrame({'A': [1, 2], 'B': [2, 1]})
pd.merge(left, right, on='B', how='outer',validate='one_to_one')
# indicator参数指示了，合并后该行索引的来源
df1 = pd.DataFrame({'col1': [0, 1], 'col_left': ['a', 'b']})
df2 = pd.DataFrame({'col1': [1, 2, 2], 'col_right': [2, 2, 2]})
pd.merge(df1, df2, on='col1', how='outer', indicator=True) # indicator='indicator_column'也是可以的

```
## 4.2 join函数
join函数作用是将多个pandas对象横向拼接，遇到重复的索引项时会使用笛卡尔积，默认左连接，可选inner、outer、right连接。
```python
left = pd.DataFrame({'A': ['A0', 'A1', 'A2'],
                     'B': ['B0', 'B1', 'B2']},
                    index=['K0', 'K1', 'K2'])
right = pd.DataFrame({'C': ['C0', 'C2', 'C3'],
                      'D': ['D0', 'D2', 'D3']},
                    index=['K0', 'K2', 'K3'])
left.join(right)
# 对于many_to_one模式下的合并，往往join更为方便，同样可以指定key：
left = pd.DataFrame({'A': ['A0', 'A1', 'A2', 'A3'],
                     'B': ['B0', 'B1', 'B2', 'B3'],
                     'key': ['K0', 'K1', 'K0', 'K1']})
right = pd.DataFrame({'C': ['C0', 'C1'],
                      'D': ['D0', 'D1']},
                     index=['K0', 'K1'])
left.join(right, on='key')
# 多层key：
left = pd.DataFrame({'A': ['A0', 'A1', 'A2', 'A3'],
                     'B': ['B0', 'B1', 'B2', 'B3'],
                     'key1': ['K0', 'K0', 'K1', 'K2'],
                     'key2': ['K0', 'K1', 'K0', 'K1']})
index = pd.MultiIndex.from_tuples([('K0', 'K0'), ('K1', 'K0'),
                                   ('K2', 'K0'), ('K2', 'K1')],names=['key1','key2'])
right = pd.DataFrame({'C': ['C0', 'C1', 'C2', 'C3'],
                      'D': ['D0', 'D1', 'D2', 'D3']},
                     index=index)
left.join(right, on=['key1','key2'])

```
## 5 问题与练习
pass # to add

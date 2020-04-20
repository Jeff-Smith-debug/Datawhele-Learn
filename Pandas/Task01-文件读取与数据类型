@[TOC](Pandas入门)
**这个主要是对pandas官方文档的学习与做练习**
使用的IDE是jupyter，基于web的交互式编程
# 1 理论部分
掌握常见文件格式的读写操作
理解并熟悉 Series 和 DataFrame 的重要属性和重要方法
掌握各类排序（索引排序和值排序、单级排序和多级排序）
## 1.1 文件读取与写入
1、读取
**重要点就是 文件的路径问题：绝对路径与相对路径**
文件读取路径问题：
绝对路径：f=open('E:/学习相关/Python/数据样例/用户侧数据/账单.csv')
	df=pd.read_csv(f)
相对路径：和ipy程序为相对起点； 必须以后都尽量使用相对路径
	df = pd.read_csv('data/table.csv')
	df.head()
**主要读取格式：csv xlx txt 格式**
2、写入
主要是写入 csv和xls格式
	df.to_csv('data/new_table.csv')
## 1.2 基本数据结构
### 1.2.1 Series
Seris的创建，属性，方法；
代码示例：
### 1.2.2 DataFrame
DataFrame的创建，属性，方法，删除，添加
代码示例：
## 1.3 常用基本函数
常用的基本函数较多，选取以下几个函数：
df.describe() # 统计各个量
df['Math'].apply(lambda x:str(x)+'!').head() #apply函数 可以使用lambda表达式，也可以使用函数
unique和nunique

## 1.4 排序
### 1.4.1 索引排序
df.set_index('Math').head() #set_index函数可以设置索引，将在下一章详细介绍
### 1.4.2 值排序
df.sort_values(by='Class').head() # 按照class 的值进行排序
# 2问题与练习
## 2.1 问题
【问题一】 Series和DataFrame有哪些常见属性和方法？
答：常见属性：index name dtype  value
【问题二】 value_counts会统计缺失值吗？
答：不会，统计非缺失值
【问题三】 与idxmax和nlargest功能相反的是哪两组函数？
答：idxmin 和 nsmallest
【问题四】 在常用函数一节中，由于一些函数的功能比较简单，因此没有列入，现在将它们列在下面，请分别说明它们的用途并尝试使用。
sum求和/mean求均值/median中位数/mad/min最小值/max最大值/abs绝对值/std/var方差/quantile/cummax/cumsum/cumprod
【问题五】 df.mean(axis=1)是什么意思？它与df.mean()的结果一样吗？第一问提到的函数也有axis参数吗？怎么使用？
答：结果不一样：df.mean()是对数值型数据求平均值，每一个col对应的平均值，df.mean(axis=1)是全部展开了； Seris里面没有 DataFrame里面有，使用方法一样；
## 2.2 练习
1、《权利的游戏》剧本数据集分析
（a）在所有的数据中，一共出现了多少人物？ 
df['Name'].nunique()       564
（b）以单元格计数（即简单把一个单元格视作一句），谁说了最多的话？
df['Name'].value_counts()      tyrion lannister      1760
（c）以单词计数，谁说了最多的单词？
不会，怎么统计一句话里面有多少单词呢？

2、科比投篮数据集分析
（a）哪种action_type和combined_shot_type的组合是最多的？
df['action_type'].value_counts()   18880
（b）在所有被记录的game_id中，遭遇到最多的opponent是哪一个？
   df['opponent'].value_counts()        SAS    1978

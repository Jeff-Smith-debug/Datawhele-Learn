@[TOC](OpenCV框架与图像差值算法)
在python环境下配置：
安装CV：使用pip: pip install opencv-python -i https://pypi.tuna.tsinghua.edu.cn/simple 使用清华源镜像。
学习目标：1、了解插值算法与常见几何变换之间的关系
	2、理解插值算法的原理
	3、掌握OpenCV框架下插值算法API的使用
# 1 主要内容
1、插值算法原理介绍
	最近邻插值算法
	双线性插值算法
2、OpenCV代码实践
	cv.resize()各项参数及含义
3、动手实现（由读者自己完成）

# 1 插值算法
在图像处理中，平移变换、旋转变换以及放缩变换是一些基础且常用的操作。这些几何变换并不改变图象的象素值，只是在图象平面上进行象素的重新排列。在一幅输入图象[u，v]中，灰度值仅在整数位置上有定义。然而，变换输出图象[x，y]的灰度值一般由处在非整数坐标上的（u，v）值来决定。这就需要插值算法来进行处理，常见的插值算法有最近邻插值、双线性插值和三次样条插值。
# 2 算法理论介绍
## 2.1 最近邻插值算法原理
原理：最近邻插值，是指将目标图像中的点，对应到源图像中后，找到最相邻的整数点，作为插值后的输出。
 
如上图所示，目标变换图像中的某点投影到原图像中的位置为点P,此时易知，f(P)=f(Q11).
### 例子
如下图所示，将一幅3X3的图像放大到4X4，用f(x,y)表示目标图像，h(x,y)表示原图像，我们有如下公式：
	 
具体结果： f(0,0)=h(0,0) f(0,1)=h(0,0.75)=h(0,1) f(0,2)=h(0,1.50)=h(0,2) f(0,3)=h(0,2.25)=h(0,2) ... 
 
缺点：用该方法作放大处理时，在图象中可能出现明显的块状效应；
## 2.2 双线性插值
在讲双线性插值之前先看以一下线性插值，线性插值多项式为：f(x)=a1x+a0；
 

 
双线性插值就是线性插值在二维时的推广,在两个方向上做三次线性插值，具体操作如下图所示:
 

令f(x，y)为两个变量的函数，其在单位正方形顶点的值已知。假设我们希望通过插值得到正方形内任意点的函数值。则可由双线性方程:	f(x,y)=ax+by+cxy+d
来定义的一个双曲抛物面与四个已知点拟合，接下来就是确定 a,b,c,d这四个数；
首先对上端的两个顶点进行线性插值得：
			f(x,0)=f(0,0)+x[f(1,0)−f(0,0)]
类似地，再对底端的两个顶点进行线性插值有：
			f(x,1)=f(0,1)+x[f(1,1)−f(0,1)]
最后，做垂直方向的线性插值，以确定：
			f(x,y)=f(x,0)+y[f(x,1)−f(x,0)]
整理得：
			f(x,y)=[f(1,0)−f(0,0)]x+[f(0,1)−f(0,0)]y +[f(1,1)+f(0,0)−f(0,1)−f(1,0)]xy+f(0,0)
## 2.3 映射方法
解决是坐标从原图像到目标图像变换，还是从目标图像到原图像变换。理论看起来都是有点抽象的，还是得具体实现例子来具象化；
### 2.3.1 向前映射法
	可以将几何运算想象成一次一个象素地转移到输出图象中。如果一个输入象素被映射到四个输出象素之间的位置，则其灰度值就按插值算法在4个输出象素之间进行分配。称为向前映射法，
**从原图象坐标计算出目标图象坐标镜像、平移变换使用这种计算方法**
### 2.3.2 向后映射法
	向后映射法（或象素填充算法）是输出象素一次一个地映射回到输入象素中，以便确定其灰度级。如果一个输出象素被映射到4个输入象素之间，则其灰度值插值决定，向后空间变换是向前变换的逆
**从结果图象的坐标计算原图象的坐标** 我认为还是这种更容易理解一点；旋转、拉伸、放缩可以使用

# 3 代码实践
即重要的是弄懂这个cv.resize（）里面的参数意义；
cv2.resize(src, dsize,fx,fy,interpolation)
 
```python
#python环境
import cv2

# %%读取图像
img=cv2.imread('yuner.jpg',cv2.IMREAD_UNCHANGED)

print('Original Dimensions : ',img.shape)
scale_percent = 30       # percent of original size
width = int(img.shape[1] * scale_percent / 100)
height = int(img.shape[0] * scale_percent / 100)
dim = (width, height)

# %% resize shape
shrink = cv2.resize(img, dsize=dim, interpolation = cv2.INTER_LINEAR)

#放大
fx = 1.5
fy = 1.5

enlarge1=cv2.resize(shrink, dsize=None, fx=fx, fy=fy, interpolation = cv2.INTER_NEAREST)
enlarge2=cv2.resize(shrink, dsize=None, fx=fx, fy=fy, interpolation = cv2.INTER_LINEAR)

print('shrink Dimensions : ',shrink.shape)

# %% 输出图片
cv2.imshow("shrink image", shrink)
cv2.imshow("INTER_NEAREST image", enlarge1)
cv2.imshow("INTER_LINEAR image", enlarge2)
cv2.waitKey(0)
cv2.destroyAllWindows()
```
# 4 小结
要学会快速使用cv2里面的方法

@[TOC](CV-阈值分割)
学习内容是对经典的阈值分割算法进行回顾，图像阈值化分割是一种传统的最常用的图像分割方法，因其实现简单、计算量小、性能较稳定而成为图像分割中最基本和应用最广泛的分割技术。它特别适用于目标和背景占据不同灰度级范围的图像。
是进行图像分析、特征提取与模式识别之前的必要的图像预处理过程。图像阈值化的目的是要按照灰度级，对像素集合进行一个划分，得到的每个子集形成一个与现实景物相对应的区域，各个区域内部具有一致的属性，而相邻区域不具有这种一致属性。这样的划分可以通过从灰度级出发选取一个或多个阈值来实现。
# 1 学习内容
1、了解阈值基本概念，掌握最大类间方差方法(大津法)，自适应阈值分割的原理
2、掌握OpenCV框架下阈值算法API的使用
# 2 算法原理
## 2.1 最大类间方差法(大津阈值法)
大津法（OTSU）是一种确定图像二值化分割阈值的算法，按照大津法求得的阈值进行图像**二值化分割**后，前景与背景图像的类间方差最大，该方法又称作最大类间方差法。
被认为是图像分割中阈值选取的最佳算法，计算简单，不受图像亮度和对比度的影响。方差是灰度分布均匀性的一种度量,背景和前景之间的类间方差越大,说明构成图像的两部分的差别越大，是针对图像全局阈值。
缺点：对图像噪声敏感；只能针对单一目标分割；当目标和背景大小比例悬殊、类间方差函数可能呈现双峰或者多峰。
公式推导：
## 2.2 自适应阈值
前面介绍了OTSU算法，但那算法属于全局阈值法，所以对于某些光照不均的图像，这种全局阈值分割的方法会显得苍白无力。
自适应阈值法(adaptiveThreshold)，它的思想不是计算全局图像的阈值，而是根据图像不同区域亮度分布，计算其局部阈值，所以对于图像不同区域，能够自适应计算不同的阈值，因此被称为自适应阈值法。(其实就是局部阈值法)。
### 2.2.1 如何确定局部阈值
可以计算某个邻域(局部)的均值、中值、高斯加权平均(高斯滤波)来确定阈值。值得说明的是：如果用局部的均值作为局部的阈值，就是常说的移动平均法。

# 3 代码实践
## 3.1 简单阈值
这个函数就是cv2.threshold()。这个函数的第一个参数就是原图像，原图像应该是灰度图。第二个参数就是用来对像素值进行分类的阀值，第三个参数就是当像素值高于（或者小于）阀值时，应该被赋予新的像素值。
```python
# %% 导入包和读取数据 最大类间方差法
import cv2 as cv
#import numpy as np
from matplotlib import pyplot as plt

img=cv.imread('lena.jpg')
img_gray=cv.cvtColor(img,cv.COLOR_BGR2GRAY)

ret,thresh1=cv.threshold(img_gray,127,255,cv.THRESH_BINARY) #为什么加 ret ，这个函数有两个返回值
plt.subplot(121),plt.imshow(img_gray,'gray'),plt.title('original')
plt.subplot(122),plt.imshow(thresh1,'gray'),plt.title('thresh')
plt.xticks([]),plt.yticks([])
plt.show()
```
## 3.2 Otsu's二值化阈值
Otsu’s就可以自己找到一个认为最好的阈值。并且Otsu’s非常适合于图像灰度直方图具有双峰的情况，他会在双峰之间找到一个值作为阈值，对于非双峰图像，可能并不是很好用。
```python
# %% Otsu's二值化阈值
import cv2 as cv
#import numpy as np
from matplotlib import pyplot as plt
img=cv.imread('lena.jpg')
img=cv.cvtColor(img,cv.COLOR_BGR2GRAY)

ret,th1=cv.threshold(img,0,255,cv.THRESH_BINARY+cv.THRESH_OTSU)

blur= cv.GaussianBlur(img,(5,5),0)
ret2,th2=cv.threshold(blur,0,255,cv.THRESH_BINARY+cv.THRESH_OTSU)


plt.subplot(121),plt.imshow(th1,'gray'),plt.title('OSTU')
plt.subplot(122),plt.imshow(th2,'gray'),plt.title('blur')
plt.xticks([]),plt.yticks([])
plt.show()
```
## 3.3. 自适应阈值
根据图像上的每一个小区域计算与其对应的阀值。因此在同一幅图像上的不同区域采用的是不同的阀值，从而使我们能在亮度不同的情况下得到更好的结果。这种方法需要我们指定三个参数，返回值只有一个
```python
# %% 自适应阈值
import cv2 as cv
#import numpy as np
from matplotlib import pyplot as plt

img=cv.imread('lena.jpg')
img_gray=cv.cvtColor(img,cv.COLOR_BGR2GRAY)

img=cv.medianBlur(img_gray,5) # 中值滤波
# 自适应阈值制定方法
ret,th1=cv.threshold(img,127,255,cv.THRESH_BINARY) 
th2 = cv.adaptiveThreshold(img,255,cv.ADAPTIVE_THRESH_MEAN_C,cv.THRESH_BINARY,11,2 )
th3 = cv.adaptiveThreshold(img,255,cv.ADAPTIVE_THRESH_GAUSSIAN_C,cv.THRESH_BINARY,11,2)

titles=['original image','gloable thresholding','Adaptive mean thresholding','Adaptive \
        gaussian threshodpng']
images=[img,th1,th2,th3]

for i in range(4):
    plt.subplot(2,2,i+1),plt.imshow(images[i],'gray'),plt.title('titles[i]')
    plt.xticks([]),plt.yticks([])
    
plt.show()

# %% Otsu's二值化阈值
import cv2 as cv
#import numpy as np
from matplotlib import pyplot as plt
img=cv.imread('lena.jpg')
img=cv.cvtColor(img,cv.COLOR_BGR2GRAY)

ret,th1=cv.threshold(img,0,255,cv.THRESH_BINARY+cv.THRESH_OTSU)

blur= cv.GaussianBlur(img,(5,5),0)
ret2,th2=cv.threshold(blur,0,255,cv.THRESH_BINARY+cv.THRESH_OTSU)


plt.subplot(121),plt.imshow(th1,'gray'),plt.title('OSTU')
plt.subplot(122),plt.imshow(th2,'gray'),plt.title('blur')
plt.xticks([]),plt.yticks([])
plt.show()
```


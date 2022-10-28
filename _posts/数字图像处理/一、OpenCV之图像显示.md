---
title: OpenCV之图像显示
categories: 
-  数字图像处理
typora-root-url: ..\..
---

记录OpenCV入门经历以及数字图像处理课程的实验

<!--more-->

# OpenCV学习记录（一）

## 环境配置

使用Python搭配VS-Code和OpenCV库进行实验环境的配置。

由于python环境和VS-Code很早就安装配置好了，此部分只谈如何在前两者的基础上配置OpenCV的实验环境。

相比于C++&VS-Code&OpenCV 环境的配置，我要做的其实很简单。

在命令行窗口（cmd）中输入`pip install opencv-python`，环境就配置完成了。

然后在新建一个文件夹作为项目地址，用VSCode将其打开，然后新建一个文件，如`ex.py`，写入代码

```python
import numpy as np
import cv2

img=cv2.imread('eg.jpg',cv2.IMREAD_UNCHANGED)
cv2.namedWindow('girl',cv2.WINDOW_NORMAL)
cv2.imshow('girl',img)
cv2.waitKey(0)
```

其中，`eg.jpg`是py文件同级文件夹下的一张图片，如果图片位置在其他地方，则可以通过相对路径或者绝对路径来访问。

然后运行即可，运行效果如下：

<img src="/../数字图像处理/一、OpenCV之图像显示/image-20221027204812288.png" alt="image-20221027204812288" style="zoom:50%;" />

 绝对路径较为简单，但若路径较长会使观感不太好。若使用相对路径，可能还需要简单的配置。



## VS-Code设置

说到相对路径，VS-Code有一个较为奇怪的设置。

即运行`print(os.getcwd())`这行代码时，程序在VS-Code中运行的输出和直接运行后在cmd中的输出结果不一致。如，若该文件的实际位置是`D:\Course\DigImgProcessing\exp1\exp.py`，则前者输出是`D:\Course\DigImgProcessing`，后者输出是`D:\Course\DigImgProcessing\exp1`。

原因：VS Code把用“File”菜单->"Open Folder..."选项打开的文件夹的位置(${workspaceFolder} )当做**默认当前工作路径**。

**解决方案：**

先在Settings中，**勾选“Terminal：Excute In File Dir”**，

然后在项目配置文件`launch.json`中添加`"cwd": "${fileDirname}"`即可。



## 从实验讲起

终于谈到了学习OpenCV的原因，其实就是数字图像处理实验需要我学习和使用它。

我是怎么摸索并知道要用它来做实验？其实很简单，无非是老一套的搜索原题找到教程罢了，这一点就不再赘述，先看看第一次实验的题目吧！

> Pset1_Basic Image Manipulation
>
> 实验1-1：图像显示
>
> 实验要求：
>
> 1）利用图像库的功能，实现从文件加载图像，并在窗口中进行显示的功能；
>
> 2）利用常见的图像文件格式（.jpg； .png； .bmp； .gif）进行测试。
>
> ![img](/../数字图像处理/一、OpenCV之图像显示/clip_image002.jpg)
>
> 
>
> 实验1-2：图像合成
>
> 实验要求：
>
> 1）现有一张4通道透明图像a.png:
>
> 2）从其中提取出alpha通道并显示;
>
> 3）用alpha混合，为a.png替换一张新的背景（bg.png）。 
>
> ![image-20221027221157348](/../数字图像处理/一、OpenCV之图像显示/image-20221027221157348.png)

### 实验1-1

首先是实现从文件加载图像。

对于像以`jpg`, `png`, `bmp` 甚至`webp`为文件格式（后缀）的静态图片来说，打开并加载的方式又很多种，因为除了OpenCV以外，python还有很多图像处理的库。

对于此实验，可以使用 opencv或者python的PIL库（pillow）来实现。

#### 打开静态图片

```python
import cv2
from PIL import Image

path='./1-1/eg.jpg'

#opencv
img=cv2.imread(path,cv2.IMREAD_UNCHANGED)
#nameWindow()的作用是为图片准备一个窗口，使得图片是可交互的（如改变大小，这与第二个参数有关），前提是nameWindow和imshow的命名一致。
#直接使用imshow而不用nameWindow也行
cv2.nameWindow('girl',cv2.WINDOW_NORMAL)
cv2.imshow('girl',img)
#waitKey中参数决定了窗口打开时长，当参数为0时表示需要手动操作，不会自动关闭。
cv2.waitKey(0)


#PIL
img=Image.open(path)
img.show()
```

#### 打开GIF图

由于没有基础，做这个就颇费了一番功夫，好在也不算太复杂。

对此，我也找到了两种实现方式。

```python
#识别图像模式
im=Image.open(file)
#gif图的mode为P，其他图像一般为RGB或者RGBA
print(im.mode)
#或者根据文件的格式来选择读入时使用的方法
print(im.format)

#使用PIL库来处理GIF
def showGif1(im):
    #使用迭代器
    for frame in ImageSequence.Iterator(im):    
        frame = frame.convert('RGB')
        cv2_frame = np.array(frame)
        show_frame = cv2.cvtColor(cv2_frame, cv2.COLOR_RGB2BGR)
        cv2.imshow(im.format, show_frame)
        cv2.waitKey(10)   
    cv2.destroyAllWindows()
    
# 使用opencv的videoCaptrue来处理GIF
def showGif2(file):
    cap = cv2.VideoCapture(file)
    while(cap.isOpened()):
        # 一帧一帧捕捉
        # ret=True if it finds a frame else False. 
        ret, frame = cap.read()
        if(ret==False):
            break
        # 对帧的操作,转为灰度图像
        # gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        # 显示返回的每帧
        # cv2.imshow('frame',gray)
        # 正常读
        cv2.imshow('GIF',frame)
        cv2.waitKey(10)
    # 当所有事完成，释放 VideoCapture 对象
    cap.release()
    cv2.destroyAllWindows()    
```



#### 图像同窗

此外，要实现图片中的效果，还需要将四张图片拼接在一起，确切地说是把他们放到同一个窗口中。又花了一天左右，终于实现了此功能，进度太慢了，主要还是不了解python和它的库......

##### 静态图片同窗

实现图像同窗展示，其一是用另一个库pyplot的功能。

关键代码就是

```python
from matplotlib import pyplot as plt
import cv2
# 先用cv2读入文件
img=cv2.imread(file)
# 再转换图片的模式，因为opencv处理图像是以BGR处理，python库是以RGB处理
# 若不转换模式，图片展示时颜色会异于原图
img=cv2.cvtColor(img,cv2.COLOR_BGR2RGB)
# 三个参数依次为目标窗口展示图片的行，列和读入图片所在位置
plt.subplot(row,col,location)
plt.imshow(img)
# 图片上的标题
plt.title('title.png')
# 用以隐藏附着于图的坐标系
plt.axis('off')
# 作用和上面一行代码一致，二者可替换
# plt.xticks([])
# plt.yticks([])

# 展示拼接后的窗口
plt.show()
```

**实现效果**

![image-20221028222036144](/../数字图像处理/一、OpenCV之图像显示/image-20221028222036144.png)

但是上面方法实现的还不够，因为还需要将一张gif存进去。而我实现gif图的展示的最终方式都是通过逐帧播放。

如果使用python库，如pillow，它打开图片的方式是下面这样，窗口要手动关闭或者借助其他库来关闭，这就使得图片不能连续播放。而opencv的imshow则可以设置cv2.waitKey(time)自动刷新，进而控制gif的播放速度。为了使gif也能和其他静态图片同窗，需要找到其他方法，但是已经很接近了。

![image-20221028223134711](/../数字图像处理/一、OpenCV之图像显示/image-20221028223134711.png)

##### 动静同窗

上课的时候，学到图片本质就是像素的堆叠。一般来说，图像是一个标准的举行，有着长宽。而矩阵也有行和列，矩阵的操作在数学和计算机中的处理都很常见且成熟，于是很自然的就把图像作为一个矩阵，把对图像的操作转换成对矩阵的操作，实际上所有的图像处理工具都是这么做的。所以接下来，我们操作的本质也是把图片当矩阵处理。

ps:这些其实都是做完实验后分析过程和结论给出的马后炮了，上课的时候我可没记住......

开始我找到的还是简单的将图片堆叠的方法，具体就是使用python的numpy库来。**NumPy**(Numerical Python) 是 Python 语言的一个扩展程序库，支持大量的维度数组与矩阵运算，此外也针对数组运算提供大量的数学函数库。图片像素值的读取，替换，随机剪裁，拼接等等都可以使用ndarray。对于已经习惯使用Numpy的人们来说，已经可以不使用OpenCV进行图像处理。

首先是`np.concatenate()`函数，它是用来对数列或矩阵进行合并的。

示例：

```python
import numpy as np
a=[1,2,3]
b=[4,5,6]
c=np.concatenate((a,b),axis=0)
print(c)   
#输出
[1 2 3 4 5 6]
```

此函数对数组的处理我并不是充分了解，但是先明白它是可以处理图像拼接的。

`merge1=np.concatenate((img1,img2),axis=1)`

此行代码的作用是实现两张的图片的横向拼接。不过在拼接前，要令被拼接的图像大小和格式一致，至少对横向拼接的两张图片而言，不要求都是正方形，但图片的纵向长度（宽）应该保持一致。

`cv2.imshow('merge1',merge1)`

![image-20221028230419526](/../数字图像处理/一、OpenCV之图像显示/image-20221028230419526.png)

那剩下的一张静态图和一张gif，就可以继续横向拼接。但为了美观，我们可以将其拼接为**“田”**字。也就是在处理gif图时，插入`merge2=np.concatenate((img3,frame),1)`来拼接图三和gif的某一帧frame，然后再将两张横向图进行纵向的拼接，最后展示。

也就是

```
 # 纵向拼接两个图像
merge=np.concatenate((merge1,merge2),0)
# 展示
cv2.imshow('GIF',merge)
cv2.waitKey(10)
```

最终结果

![res](/../数字图像处理/一、OpenCV之图像显示/res.gif)

此外，还可以用 

`merge1=np.hstack([img1,img2])`

`merge2=np.hstack([img3,frame])`

`merge=np.vstack([merge1,merge2])`

分别替代上面的`np.concatenate()`函数，实现的效果是一样的。

那么至此，实验1-1算是做完了。

#### 保存gif

但是写实验报告时，也许还需要贴一张gif图来展示，那么如何制作一张gif图片呢？

要制作gif图，首先要准备足够的图片作为gif或者视频的帧页，对此，我需要在前面的循环中保存每一帧图片。

在循环中插入代码

`cv2.imwrite("./frames/frame%d.png" % i,merge)`

即可。



然后根据已有帧页，制作gif。

方法有很多，贴一个我使用的。

```python
import imageio
frames = []
for i in range(110):
    # print(img)
	img = imageio.imread('./frames/frame{}.png'.format(i+1))  # RGB格式的array
	frames.append(img)
imageio.mimsave('res.gif', frames, fps=30)
```

注意不要使用这种方式

```python
import os
import cv2
import imageio

path = 'data'
files = [os.path.join(path, f) for f in os.listdir(path)]
frames = []

for f in files:
	img = imageio.imread(f)  # RGB格式的array
	frames.append(img)
imageio.mimsave('res.gif', frames, fps=20)
```

具体来说是指循环的方式。当帧数大于100时，此循环读文件的顺序并不是我们想象中的从file1到file101，而是按file1,  file100, file101, file102 ,..., file2, file3, ... file99的顺序来读的，这使得合成的图片播放顺序有明显的错误。

这下才算真的而做完了1-1，下一篇更新实验1-2，图像合成。

## 参考资料

1 [OpenCV-Python 教程简介 (apachecn.org)](https://opencv.apachecn.org/#/docs/4.0.0/1.1-tutorial_py_intro)

2 [Pillow是什么 (biancheng.net)](http://c.biancheng.net/pillow/what-is-pillow.html)

3 [Vscode的相对路径读取问题及处理](https://blog.csdn.net/sumaliqinghua/article/details/90404173)

4 [【OpenCV】imshow()和namedWindow()之间的关系]([【OpenCV】imshow()和namedWindow()之间的关系，解决两个窗口问题_Running Y的博客-CSDN博客_namedwindow](https://blog.csdn.net/weixin_43243787/article/details/104755685))

5 [python使用opencv或matplotlib把多张图片显示在一个窗口内](https://blog.csdn.net/ITBigGod/article/details/87009082)

6 [OpenCV实验（1）：图像的加载与显示_](https://blog.csdn.net/weixin_43360801/article/details/109487956)

7[pip安装第三方库报错](https://www.cnblogs.com/xiaofeng91/p/14840128.html)

8 [opencv把一系列图像保存为视频、一系列图像保存为giff](https://blog.csdn.net/weixin_43508499/article/details/115522755)
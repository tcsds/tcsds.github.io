---
title: 面向Python的PlotNeuralNet教程
date: 2021-05-10 19:44:06
categories: 
    - 科研工具
    - 可视化

tags:
    - 深度学习可视化
    - PlotNeuralNet
---

本文参考了[不够优秀][1]、[Zhanglw882][2]等人的博文，在此表示感谢！

# 入门案例详解
该案例是pyexample中的例子，能够通过 bash …/tikzmake.sh test_simple 直接运行出来，
```python
import sys
sys.path.append('../')
#pycore.tikzeng是发布者写的python接口，可以直接调用里面的函数来就行绘图，本博客末尾有解释
from pycore.tikzeng import *

#定义自己的网络图结构
arch = [
    # 前三行是一个初始化的过程，以后写自己的网络图照抄上去就行
    to_head( '..' ),
    to_cor(),
    to_begin(),
    
    # 网络头部添加图片
    to_input('img_1.png', width=8, height=8),

    #添加一个卷积层conv1
    to_Conv("conv1", 512, 64, offset="(0,0,0)", to="(0,0,0)", height=64, depth=64, width=2 ),
    #添加一个池化层pool1
    to_Pool("pool1", offset="(0,0,0)", to="(conv1-east)"),
    #添加一个卷积层conv2
    to_Conv("conv2", 128, 64, offset="(1,0,0)", to="(pool1-east)", height=32, depth=32, width=2 ),
    #添加一个连接线，在pool1和conv2之间
    to_connection( "pool1", "conv2"), 
    #添加一个池化层pool2
    to_Pool("pool2", offset="(0,0,0)", to="(conv2-east)", height=28, depth=28, width=1),
    #添加一个softmax层soft1
    to_SoftMax("soft1", 10 ,"(3,0,0)", "(pool1-east)", caption="SOFT"  ),
    #添加一个连接线，在pool2和soft1之间
    to_connection("pool2", "soft1"), 
    #添加一个 “+” 按钮sum1
    to_Sum("sum1", offset="(1.5,0,0)", to="(soft1-east)", radius=2.5, opacity=0.6),
    #添加一个连接线，在soft1和sum1之间
    to_connection("soft1", "sum1"),

    #结束
    to_end()
    ]

#后面的东西在自己画图是直接粘贴过去就行
def main():
    namefile = str(sys.argv[0]).split('.')[0]
    to_generate(arch, namefile + '.tex' )

if __name__ == '__main__':
    main()
```
结果如图所示

![入门案例](./面向Python的PlotNeuralNet教程/demo.png)


## 添加头部图片
使用to_put()函数
```python
#####################################################################
# 添加输入的图像到网络头部，使用函数to_input
# to_input(pathfile,to='(-3,0,0)', width=8, height=8, name="temp")
# pathfile--输入图片的路径，
# to="(-3,0,0)"--输入图片的坐标(x,y,z),
# width=16,height=16--图片显示时的宽和高,根据需要自己调节
# name--指名称
# 这行代码紧跟前三行的后面
#####################################################################
to_input('img_1.png',width=8, height=8),
```

## 添加卷积层

注意一下，s_filer，n_filer指的是你在图中标注的尺寸大小，width,height, depth指的是在图中图形显示的实际厚、长与深度（宽）。
```python
#####################################################################
# 添加卷积层
# to_Conv(name,s_filer=256,n_filer=64,offset="(0,0,0)",to="(0,0,0)",width=1,height=40, depth=40, caption=" ")
# name--名称
# s_filer--卷积层图像尺寸
# n_filer--卷积层图像深度(通道数)
# s_filer、n_filer指卷积层结构的参数,并非制图时的尺寸
# offset--与前一层分别在x,y,z方向的距离
# to--在x,y,z方向的坐标，
# offset、to的在后面会深入讲，你可以自己该不同的值试验一下，找规律
# width--制图时的厚度
# height、depth--制图时的长宽
# width、height、depth指在制图时,卷积层的尺寸
# caption--备注信息
#####################################################################
to_Conv("conv1", 512, 64, offset="(0,0,0)", to="(0,0,0)", height=64, depth=64, width=32 ),
```
效果如图所示

![添加卷积层](./面向Python的PlotNeuralNet教程/conv.png)

## 添加池化层

```python
#####################################################################
# 添加池化层
# to_Pool(name,offset="(0,0,0)",to="(0,0,0)",width=1,height=32,depth=32,opacity=0.5,caption=" ")
# 部分参数与卷积层相同
# opacity--透明度,0-1之间
# to="(conv1-east)"--在con1层的东侧，表示与conv1层相邻，距离为0。
#####################################################################
to_Pool("pool", offset="(0,0,0)",to="(conv1-east)",width=1, height=64, depth=64,opacity=0.9),
```
效果如下图所示：
![pool1](./面向Python的PlotNeuralNet教程/pool.png)

offset、to这两个参数，使用它们可以调整层对应的位置：
to表示参考坐标，表示x，y，z，to="(conv1-east)"可以表示在con1层的正东侧(放在平面上理解是右边)，
通过offset的值设置偏移的距离，to的表示offset所参考坐标位置。比如：offset="(0,0,0)",to="(conv1-east)"


测试如下代码：

```python
to_Conv("conv1", 512, 64, offset="(0,0,0)", to="(0,0,0)", height=64, depth=64, width=2 ),
to_Pool("pool1", offset="(0,0,0)", to="(conv1-east)"),

to_Conv("conv2", 512, 64, offset="(5,0,0)", to="(0,0,0)", height=64, depth=64, width=2 ),
to_Pool("pool2", offset="(5,0,0)", to="(conv2-east)"),
```

这里创建了依次创建了卷积层conv1、池化层pool1、卷积层conv2、池化层pool2。
pool1在conv1的正东侧，偏移offset为0；
conv2在位置相对于上一幅图的位置，在X轴方向上偏移了5个单位(水平向右为x的正方向)，即上一幅图pool1的位置是(0，0，0)，故邻接的图片conv2的位置是(5,0,0),相对偏移offset(5，0，0)；
pool2在conv2的正东侧，x方向上偏移offset为5。如下图所示：

![](./面向Python的PlotNeuralNet教程/offset_to.png)


## 添加层间连接和跳接
网络中层与层之间的需要用带箭头的线进行连接，用到以下两个函数：

- to_connection()
- to_skip()

```python
#水平连接，参数为：起始位置、中止位置
to_connection("pool1", "conv2"),
to_connection("conv2","pool2"),
#跨连接，参数为：起始位置、中止位置，pos默认为1.25
to_skip(of = "pool1",to = "pool2",pos=1.25),
```
效果如图所示：

![层间连接](./面向Python的PlotNeuralNet教程/connect_skip.png)

pos的设置，若值大于1，表示向上平移1.25个高度，例如pos=1.25，表示skip这条线,相对于原点到终点的直线，向上平移1.25个高度；若小于1，表示该直线向下平衡若干单位

## 添加卷积层和Relu层
to_ConvConvRelu：卷积层+激活函数to_ConvConvRelu。该函数可以绘制2个卷积层和2个Relu层。即：Conv+Relu+Conv_Relu; 也可以只绘制1个卷积层和1个激活层。这里的关键点在于width=(number, number)。若绘制2个卷积层和Relu激活层，则width中的两个数字都不能为空；若只绘制1个激活层和1个Relu激活层，则width中的两个值，其中之一需要设置为0，例如width=(5, 0)。

```python
to_ConvConvRelu(name='conv3', s_filer=224, n_filer=(64, 64), offset="(30,0,0)", to="(0,0,0)", width=(5, 5), height=120, depth=120,caption='ConvRelu3'),
to_ConvConvRelu(name='conv4', s_filer=224, n_filer=(64, 64), offset="(43,0,0)", to="(0,0,0)", width=(5, 0), height=120, depth=120,caption='ConvRelu4'),
to_ConvConvRelu("conv5", s_filer=224, n_filer=(64, 64), offset="(40,0,0)", to="(pool2-east)", height=100, depth=100, width=(4,4),caption="ConvRelu5" ),
```
效果如图所示：

![卷积层和Relu层](./面向Python的PlotNeuralNet教程./convconvrelu.png)

- n_filer=(64,64),width=(10,10)，会同时产生两个convrelu层，relu层厚度为默认厚度
- n_filer=(64,64),width=(10, 0)表示产生一个convrelu层，其中relu厚度为默认厚度

## 添加其它层
其他层的添加方法与上面几层基本一致。

例如：

```python
to_UnPool('Unpool', offset="(5,0,0)", to="(0,0,0)",height=64, width=2, depth=64, caption='Unpool'),
to_ConvRes("ConvRes",  s_filer=512, n_filer=64, offset="(10,0,0)", to="(0,0,0)", height=64, width=2, depth=64, caption='ConvRes'),
to_ConvSoftMax("ConvSoftMax",  s_filer=512,  offset="(15,0,0)", to="(0,0,0)", height=64, width=2, depth=64, caption='ConvSoftMax'),
to_Sum("sum", offset="(5,0,0)", to="(ConvSoftMax-east)", radius=2.5, opacity=0.6),
```
效果如下：

![其他网络层](./面向Python的PlotNeuralNet教程./others.png)


# 函数使用说明

## 卷积层api详解
to_Conv(name="conv1", s_filer=512, n_filer=64, offset="(0,0,0)", to="(0,0,0)", height=64, depth=64, width=2, caption='conv1' )
to_ConvConvRelu的参数含义相同。

| 参数             | 参数值   | 函数的功能                                                   |
| ---------------- | ------- | ------------------------------------------------------------ |
| name             | “conv1” | 该层的名字                                                   |
| s_filer(z_label) | 512     | 写在左下角的z方向的数字                                      |
| n_filer(x_label) | 64      | 写在左下角的x方向的数字                                      |
| offset(x,y,z)    | (0,0,0) | 和上一个图形在x,y,z方向上的偏移值，若x为1,则图像位于前面1的位置 |
| to(x,y,z)        | (0,0,0) | 若为(0,0,0)表示第一个图，若加上某一层的name,表示与该层相邻，距离为0。                                     |
| height           | 64      | 图像的高度,若没填，则为上一个图的一半                                                   |
| depth            | 64      | 图像的深度,若没填，则为上一个图的一半                                                   |
| width            | 2       | 图像的宽度,若没填，则为上一个图的一半                                                   |
| opacity          | 0.5     | 图像的透明度                                                |
| caption          | 2       | 卷积层的名称                                                 |


## 池化层api详解
to_Pool(name="pool1", offset="(0,0,0)", to="(conv1-east)",caption='pool1')

| 参数             | 参数值   | 函数的功能                                                   |
| ---------------- | -------------- | ------------------------------------------------------------ |
| name          | “pool1”        | 该层的名字                                                   |
| offset(x,y,z) | (0,0,0)        | 和上一个图形在x,y,z方向上的偏移值，若都为0，则和原图像的位置相同 |
| to(x,y,z)     | “(conv1-east)” | 上一个图的name，再加上"-east"，跟在name这个图的后面          |
| height        | 32             | 若没填，则为上一个图的一半                                   |
| depth         | 32             | 若没填，则为上一个图的一半                                   |
| width         | 1              | 若没填，则为上一个图的一半                                   |
| opacity       | 若为0.5        | 透明度                                                       |

## 卷积激活函数-to_ConvConvRelu

to_ConvConvRelu可以生成**一个或多个convrelu层**

```python
to_ConvConvRelu(name='conv3', s_filer=224, n_filer=(64, 64), offset="(40,0,0)", to="(0,0,0)", width=(2, 2), height=120, depth=120,caption='ConvRelu3'),
to_ConvConvRelu("conv4", 112, (128,128), offset="(6,0,0)", to="(pool2-east)", height=100, depth=100, width=(4,4),caption="ConvRelu4" ),
```

## 激活函数-SoftMax
to_SoftMax(name="soft1", s_filer=10, offset="(3,0,0)", to="(pool1-east)", caption="SOFT" )

| 参数             | 参数值   | 函数的功能                                                   |
| ---------------- | -------------- | ------------------------------------------------------------ |
| name             | “soft1”        | 该层的名字                                                   |
| s_filer(z_label) | 10             | 写在左下角的z方向的数字                                      |
| offset(x,y,z)    | “(3,0,0)”      | 和上一个图形在x,y,z方向上的偏移值，若都为0，则和原图像的位置相同 |
| to(x,y,z)        | “(pool1-east)” | 上一个图的name，再加上"-east"，表示与conv1层相邻，距离为0。          |
| caption          | “SOFT”         | 标题（会显示在图中）                                         |


## 箭头连接
### to_connection
to_connection( “pool1”, “conv2”)：从哪一个图到哪一个图画个箭头

| 参数             | 参数值   | 函数的功能                                                   |
| ---------------- | -------------- | --------------------------------------------------- |
| of             | "pool1"        | 起始层                                                  |
| to  | "conv2"            | 中止层                                      |

### 跳接
to_skip(of = "pool1",to = "pool2",pos=1.5):参数为：起始位置、中止位置，pos默认为1.25 

| 参数             | 参数值   | 函数的功能                                                   |
| ---------------- | -------------- | --------------------------------------------------- |
| of             | "pool1"        | 起始层                                                  |
| to  | "conv2"            | 中止层                                      |
| pos  | 1.25            | 表示skip这条线,相对于原点到终点的直线，向上平移1.25个高度；若小于1，表示该直线向下平衡若干单位 |

## 自定义网络构件
如果自定义新的网络构件，就需要修改pycore文件夹下的tikzeng.py文件。
也可以可直接下载[demonlord1997][3]博主修改好的[tikzeng.py][3]文件。

修改规则：
```python
def func(parameter):
    return r"""
    LaTeX grammar
    """
```
理论上，需要掌握基本的LaTeX语法才能对其进行修改，不过可以按照tikzeng.py文件中相关函数的定义，进行新构件的定制。

为了支持中文，需要将tikzmake.sh中的pdflatex改成xelatex，如果安装了TeXlive或者MikTeX，可以手动指定xelatex的路径，例如，我的xelatex路径是`/usr/local/texlive/2019/bin/x86_64-linux/xelatex`,则tikzmake.sh文件可以修改为：

```bash
#!/bin/bash


python3 $1.py #如果是Windows系统，则这里的python3应该写为python

/usr/local/texlive/2019/bin/x86_64-linux/xelatex $1.tex ##指明xelatex路径

rm *.aux *.log *.vscodeLog
rm *.tex

if [[ "$OSTYPE" == "darwin"* ]]; then
    open $1.pdf
else
    xdg-open $1.pdf
fi
```




[1]:https://blog.csdn.net/bug_with_me/article/details/107138400, "Windows下使用PlotNeuralNet+latex快速绘制深度学习模型图"
[2]:http://www.cxyzjd.com/article/Zhanglw882/115537471, "使用PlotNeuralNet绘制深度学习网络图"
[3]:https://demonlord1997.github.io/2020/07/03/PlotNeuralNet%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E/, "PlotNeuralNet个性化定制与使用说明"
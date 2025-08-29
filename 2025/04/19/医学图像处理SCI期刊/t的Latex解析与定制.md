---
title: PlotNeuralNet的Latex解析与定制
date: 2021-05-31 19:05:06
categories: 
    - 科研工具
    - 可视化

tags:
    - 深度学习可视化
    - PlotNeuralNet
    - 唐朝生

---


本文参考了[demonlord1997][1]、[糯米鸡呀呀呀呀][2]等人的博文，在此表示感谢！

# 基本构件

## 构件类型

基本构件类型和顺序是：

1. **\pic**, 表示添加层  
2. 指定层的位置： \pic[shift  ={(x, y, z)}] at  (x, y, z)， 表示：第1个坐标是指相对于第2个坐标的偏移量  
3. 添加具体类型的层，其中，**Box={...}**表示单个层， **RightBandedBox={...}**表示多个层，**Ball={...}**表示层间相加符号$\bigoplus$  
4. 添加层间连线，**\draw [connection]  (*层的位置，如vonv1-east,或坐标(x,y,z)*)    -- node { *箭头形状，如\midarrow*} (cr2-west)**  
5. 自定义层间连线路径，**\path (p4-east) -- (cr5-west) coordinate[pos=0.25] (between4_5) **

## 层的类型
网络中层包括：layer, block 和 unit。

在绘制网络层时，要在指定层的类型

其中，layer是单个网络层，对应的配置文件是layers文件夹下的**Box.sty**，在绘图时，所用的命令是：
```latex
 {Box                ={
            name    =...,
            caption =  ...,
            xlabel  = ...,
            ylabel  = ...,
            fill    = ...,
            opacity = ...,
            height  = ...,width= ...,depth= ...,
                    }
    }
```

其中，以下语句是绘图时常用的，其中，**\subimport{../layers/}{init}**，表示layer文件夹下的init文件，因此，这一句的具体写法要结合当前tex文件所在位置。**\def\ReLUColor{rgb:red,1;black,0.3}**是自定义的ReLU层的颜色，可以根据需要自己调整。

```latex
\documentclass[border=15pt, multi, tikz]{standalone}
\usepackage{import}
\subimport{../layers/}{init}
\usetikzlibrary{positioning}
\usetikzlibrary{3d} %for including external image 

\def\ReLUColor{rgb:red,1;black,0.3}

\begin{document}
\begin{tikzpicture}
\tikzstyle{connection}=[ultra thick,every node/.style={sloped,allow upside down},draw=\edgecolor,opacity=0.7]

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
% 这部分添加绘图命令
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
\end{tikzpicture}
\end{document}
```



可以看到，图中标签“深度”的位置没有居中，需要自行设置，具体设置方法见下文自定义配置。

### 单个layer

单个layer要在shift后指定**Box**，可以通过fill指定填充的颜色，`height=..,width=...,depth=...`指定图片的高、宽和深。

- `[shift   ={(0,0,0)}]  at  (0,0,0) `表示第1个坐标是指相对于第2个坐标的偏移量。

- name   =score16,% 表示该层的名称 

- caption= 标题, %表示该层的标题

- `xlabel ={{ "宽度： 显示的是s\_filer的值",  }}`,  无论是中文还是英文，只能显示第1个位置的字符;同时，要在s后加转义符\：s\\_filer。xlabel的值是数组形式，即(x,y,z,....),在**Box**中只显示第1个值,即**x**。如果是在"RightBandedBox",则可以显示多个值。

- `ylabel ={{高度 }}`, 加不加引号，似无影响；这里可以显示多个字符;ylabel在神经网络架构中一般不显示

- `zlabel = 深度`, ： 显示的是n_filer的值, 加不加{}，似无影响；这里可以显示多个字符

- opacity表示透明度

""
```latex
\documentclass[border=15pt, multi, tikz]{standalone}
\usepackage[UTF8]{ctex} %显示中文
\usepackage{import}
\subimport{../layers/}{init}
\usetikzlibrary{positioning}
\usetikzlibrary{3d} %for including external image 

\def\ReLUColor{rgb:red,1;black,0.3}

\begin{document}
\begin{tikzpicture}
\tikzstyle{connection}=[ultra thick,every node/.style={sloped,allow upside down},draw=\edgecolor,opacity=0.7]

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%% Draw Layer Blocks
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

% one layer for a conv or BN or ReLU layer, etc
\pic[shift         ={(5,0,0)}] at  (0,0,0) % 第1个坐标是指相对于第2个坐标的偏移量
    {
        Box        ={
            name   =conv,% 表示该层的名称 
            caption= 标题, %表示该层的标题
            xlabel ={{ "宽度： 显示的是s\_filer的值",  }}, % 无论是中文还是英文，只能显示第1个位置的字符;同时，要在s后加转义符\
            ylabel ={{高度 }}, % 加不加引号，似无影响；这里可以显示多个字符;ylabel在神经网络架构中一般不显示
            zlabel = 深度： 显示的是n\_filer的值, % 加不加{}，似无影响；这里可以显示多个字符
            fill   =\ReLUColor,
            opacity = 0.2,
            height=64,width=30,depth=45
                    }
    }; 
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
\end{tikzpicture}
\end{document}
```


效果如图所示：

![layer](./PlotNeuralNet的Latex解析与定制/layer.png)

### layer组合
多个layer的组合，有2种方法实现。

1. 在shift后使用**RightBandedBox**，此外，layer的名称和层数，是通过**xlabel**和**width**进行设定。例如：`xlabel={{"32","64", "128"}}， ···，width={2,2,4}`。xlabel和width的值是数组形式，即(x,y,z,....)，数组内的数值个数决定了有多少个层。
2. 通过多个**Box**，使多个单层layer成为layers的组合。

例如：
```latex
\pic[shift={(5,0,0)}] at (0,0,0) 
    {RightBandedBox={
            name=cr1,
            caption=使用RightBandedBox,
            xlabel={{"1","2", "3"}},
            % zlabel=layer_blocks,
            fill=\ConvColor,
            bandfill=\ConvReluColor,%
            opacity = 0.2,
            height=40,width={1,1,1},depth=40}
    };

% conv+bn+relu
\pic[shift          ={    (10,0,0)}] at  (0,0,0) 
    {
    Box             ={
        name        =conv1,%
        xlabel      ={{1}},
        fill        =\ConvColor,%
        height      =40,width=1,depth=40}
    };
\pic[shift          ={ (0,0,0)}] at (conv1-east) 
    {
        Box         ={
        name        =bn1,%
        caption     =layer叠加,
        xlabel      ={{2}},
        fill        =\BnColor,%
        opacity     =0.5,
        height      =40,width=1,depth=40}
    };
\pic[shift          ={(0,0,0)}] at (bn1-east)
    {
            Box     ={
            name    =relu1,%
            xlabel  ={{3}},
            fill    =\ReLUColor,
            opacity =0.5,
            height  =40,width=1,depth=40,
             }
    };
```

效果如图所示：

![layers](./PlotNeuralNet的Latex解析与定制/layers.png)

## 残差结构

```latex
\documentclass[border=15pt, multi, tikz]{standalone}
\usepackage{import}
\subimport{../../layers/}{init}
\usetikzlibrary{positioning}
\usetikzlibrary{3d} %for including external image

\def\ConvColor{rgb:yellow,5;red,2.5;white,5}
\def\ConvReluColor{rgb:yellow,5;red,5;white,5}
\def\ConvBnReluColor{rgb:yellow,5;red,5;white,2.5}
\def\PoolColor{rgb:red,1;black,0.3}
\def\DcnvColor{rgb:blue,5;green,2.5;white,5}
\def\SoftmaxColor{rgb:magenta,5;black,7}
\def\SumColor{rgb:blue,5;green,15}

\begin{document}
\begin{tikzpicture}
\tikzstyle{connection}=[ultra thick,every node/.style={sloped,allow upside down},draw=\edgecolor,opacity=0.7]
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%% Draw Layer Blocks
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

% conv1_1,conv1_2,%pool1
\pic[shift={(0,0,0)}] at (0,0,0) {RightBandedBox={name=cr1,caption=conv1,%
        xlabel={{"64","\ 64"}},zlabel=I,fill=\ConvColor,bandfill=\ConvBnReluColor,%
        height=40,width={2,2},depth=40}};
\pic[shift={(0,0,0)}] at (cr1-east) {Box={name=p1,%
        fill=\PoolColor,opacity=0.5,height=35,width=1,depth=35}};

\pic[shift={(3,0,0)}] at (p1-east) {Ball={name=elt1,%
        fill=\SumColor,opacity=0.6,%
        radius=2.5,logo=$+$}};


%% score16
\pic[shift={(0,-4,0)}] at (elt1-west) {Box={name=score16,%
            fill=\PoolColor,%
            opacity=0,height=0.01,width=0.01,depth=0.01}};
%%　draw connections
%\draw [connection]  (p1-east) -- node {\midarrow} (p1-east -| elt1-west) -- node {\midarrow} (elt1-west);
\draw [connection]  (p1-east)    -- node {\midarrow} (elt1-west);
\path (p1-east) -- (elt1-south) coordinate[pos=0.25] (between4_5) ;
\draw [connection]  (between4_5)    -- node {\midarrow} (score16-west-|between4_5) -- node {\midarrow} (score16-west);
\draw [connection]  (score16-east) -- node {\midarrow} (score16-east -| elt1-south) -- node {\midarrow} (elt1-south);
\draw [connection]  (p1-east)    -- node {\midarrow} (elt1-west);


%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
\end{tikzpicture}
\end{document}
```

![残差结构](PlotNeuralNet的Latex解析与定制/残差结构.png)


# PlotNeuralNet的个性化定制

PlotNeuralNet默认将图形的深度，也就是*n_fliter的数据*标注在角上，如下图所示，没有居中，因此，需要对它进行[修改][3]。

自定义方法：修改layers文件夹下的Box.sty和RightBandedBox.sty文件。

![zlabel](./PlotNeuralNet的Latex解析与定制/zlabel.png)

## **Box.sty**

将depthlabel设置中的pos=0改为pos=0.5，将captionlabel中的文本宽度由text width=15改为text width=40，这是因为默认宽度有些小，长文本显示不全。
>
`\coordinate (a1) at (0 , \y/2 , \z/2);`  
`\coordinate (b1) at (0 ,-\y/2 , \z/2);`  
~~\tikzstyle{depthlabel}=[pos=0,text width=14*\z,text centered,sloped]~~  
`\tikzstyle{depthlabel}=[pos=0.5,text width=14*\z,text centered,sloped]`  
>
~~\tikzstyle{captionlabel}=[text width=15*\LastEastx/\scale,text centered]~~   
` \tikzstyle{captionlabel}=[text width=40*\LastEastx/\scale,text centered]`    
>
`\path (\LastEastx/2,-\y/2,+\z/2) + (0,-25pt) coordinate (cap)`  
`edge ["\textcolor{black}{ \bf \caption}"',captionlabel](cap) ; %Block caption/pic object label`  

## **RightBandedBox.sty** 

RightBandedBox.sty的修改与Box.sty的修改类似。pos=0改为pos=0.5，text width=15改为text width=40
>
`\coordinate (a1) at (0 , \y/2 , \z/2);`  
`\coordinate (b1) at (0 ,-\y/2 , \z/2);`  
~~\tikzstyle{depthlabel}=[pos=0,text width=14*\z,text centered,sloped]~~  
`\tikzstyle{depthlabel}=[pos=0.5,text width=14*\z,text centered,sloped]`  
>
`\path (c) edge ["\small\zlabels"',depthlabel](f); %depth label'  
`\path (b1) edge ["\ylabel",midway] (a1);  %height label'  
>
~~\tikzstyle{captionlabel}=[text width=15*\LastEastx/\scale,text centered]~~  
`\tikzstyle{captionlabel}=[text width=40*\LastEastx/\scale,text centered]'  
>
`\path (\LastEastx/2,-\y/2,+\z/2) + (0,-25pt) coordinate (cap)'  
`edge ["\textcolor{black}{ \bf \caption}"',captionlabel] (cap); %Block caption/pic object label'  





[1]:https://demonlord1997.github.io/2020/07/03/PlotNeuralNet%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E/, "PlotNeuralNet个性化定制与使用说明"
[2]:https://blog.csdn.net/weixin_41892813/article/details/106157933, "卷积神经网络画图神器PlotNeuralNet（基于Latex）使用记录"
[3]:https://github.com/demonlord1997/PlotNeuralNet, "PlotNeuralNet的demonlord1997修改版"

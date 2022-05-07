---
layout:     post                    # 使用的布局（不需要改）
title:      Gabor 滤波器 与 GIST特征           # 标题 
date:       2022-04-20              # 时间
author:     vhwz                     # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 技术
    - 计算机视觉
---

## Gabor 滤波器 与 GIST特征

网络上关于Gabor滤波器和GIST的资料比较混乱，存在(疑似)相互矛盾的内容, 总结如下

GIST是[7]中提出的一种场景特征提取算法。

### ref

1. Evaluation of GIST descriptors for web-scale image search
2. The 2021 Image Similarity Dataset and Challenge
3. [https://ustc-dia.github.io/slides/](https://ustc-dia.github.io/slides/DIA%E7%AC%AC5%E7%AB%A0%202020-5-%E5%9B%BE%E5%83%8F%E7%89%B9%E5%BE%81%E8%A1%A8%E8%BE%BE.pdf)
4. [融合Gist特征与卷积自编码的闭环检测算法](https://www.opticsjournal.net/Articles/OJ70930a15c59470e9/FullText)
5. https://senitco.github.io/2017/06/10/image-feature-hog/
6. https://inc.ucsd.edu/mplab/75/media//gabor.pdf
7. Modeling the Shape of the Scene: A Holistic Representation of the Spatial Envelope

### Gabor滤波器

Gabor滤波器是由正弦函数与高斯核函数相乘得到的。
$$
g(x, y)=s(x, y) w_{r}(x, y)
$$
其中，
$$
s(x, y)=\exp \left(j\left(2 \pi\left(u_{0} x+v_{0} y\right)+P\right)\right)
$$
$s(x, y)$是正弦信号，其中$(u_{0}, v_{0})$是频率，$P$是相位

也可以用极坐标表示$s(x, y)$
$$
s(x, y)=\exp \left(j\left(2 \pi F_{0}\left(x \cos \omega_{0}+y \sin \omega_{0}\right)+P\right)\right)
$$
其中,
$$
F_{0} = \sqrt{u_{0}^2+v_0^2} \\
\omega_0 = tan^{-1}(\frac{v_0}{u_0})
$$
另外，
$$
w_{r}(x, y)=K \exp \left(-\pi\left(a^{2}\left(x-x_{0}\right)_{r}^{2}+b^{2}\left(y-y_{0}\right)_{r}^{2}\right)\right)
$$
$w_{r}(x, y)$是高斯函数，$(x_0, y_0)$是函数峰值位置，$(a, b)$是x, y两个方向上的缩放参数，下标$r$代表坐标旋转操作，即
$$
\begin{array}{l}
\left(x-x_{0}\right)_{r}=\left(x-x_{0}\right) \cos \theta+\left(y-y_{0}\right) \sin \theta \\
\left(y-y_{0}\right)_{r}=-\left(x-x_{0}\right) \sin \theta+\left(y-y_{0}\right) \cos \theta
\end{array}
$$
$\theta$  代表顺时针旋转的角度

展开写在一起如下：
$$
\begin{array}{c}
g(x, y)=K \exp \left(-\pi\left(a^{2}\left(x-x_{0}\right)_{r}^{2}+b^{2}\left(y-y_{0}\right)_{r}^{2}\right)\right) \\
\exp \left(j\left(2 \pi\left(u_{0} x+v_{0} y\right)+P\right)\right)
\end{array}
$$
极坐标形式
$$
\begin{array}{c}
g(x, y)=K \exp \left(-\pi\left(a^{2}\left(x-x_{0}\right)_{r}^{2}+b^{2}\left(y-y_{0}\right)_{r}^{2}\right)\right) \\
\exp \left(j\left(2 \pi F_{0}\left(x \cos \omega_{0}+y \sin \omega_{0}\right)+P\right)\right)
\end{array}
$$


总结一下，Gabor滤波器一共有6个参数

$K$ : 对高斯核函数的取值进行缩放

$(a, b)$ : 对高斯核函数的两个轴方向进行缩放

$(x_0, y_0)$ : 高斯核函数峰值的位置

$(u_0, v_0)$ : 正弦函数的频率

$P$ : 正弦函数的相位

有的论文中会设置默认值$(x_0, y_0) = (0, 0), a = b = \sigma, \theta = 0, P=0$ 

还存在另一种形式
$$
g(x, y ; \lambda, \theta, \psi, \sigma, \gamma)=\exp \left(-\frac{x^{\prime 2}+\gamma^{2} y^{\prime 2}}{2 \sigma^{2}}\right) \exp \left(i\left(2 \pi \frac{x^{\prime}}{\lambda}+\psi\right)\right)
$$
其中
$$
x^{\prime}=x \cos \theta+y \sin \theta \text { and } y^{\prime}=-x \sin \theta+y \cos \theta
$$
可以看出， 两种形式存在对应关系
$$
K = 1, x_0 = 0,  y_0 = 0\\
\theta = \omega_0\\
\sigma = \frac{1}{\sqrt{2\pi a^2}} \\
\gamma = \frac{b}{a}\\
\lambda = \frac{1}{F_0}
$$
根据维基百科，

$\lambda$代表正弦函数的波长

$\theta$代表Gabor 函数的平行条纹的法线方向

$\sigma$是高斯核函数的标准差

$\psi$是相位偏移

$\gamma$空间纵横比，指定 Gabor 函数的椭圆度

### GIST计算步骤

我找到了两种步骤，一种来自[1,2]，一种来自[3, 4]，我不确定它们是否等价

#### 版本一

1.将图片缩放为$32\times 32$

2.将图片划分为$4 \times 4$个单元(cell)，每个单元尺寸为$8 \times 8 \times 3$

3.在每个单元中计算HOG特征，连接每个单元的HOG特征

最后获取960维的向量

#### 版本二

1.构建一系列Gabor滤波器，包括m种尺度和n种方向，共mn个滤波器。用这mn个滤波器对图像进行卷积，得到mn个大小与原图相同的特征图

2.将特征图划分为$4 \times 4$个单元(cell)，每个单元内取均值。一共有mn个特征图，每个特征图有16个均值，最后得到$m\times n \times 16$的向量。

如果处理的是RGB图像，则对3个通道单独处理，最后得到$m\times n \times 16 \times 3$的向量

这里的方向指的是$\theta$，尺度指的是使用Gabor滤波器矩阵的大小

参考代码，[来源](https://blog.csdn.net/DragonGirI/article/details/104586633)

```python
import cv2,os
import numpy as np
import matplotlib.pyplot as plt
 
 
def get_img(input_Path):
    img_paths = []
    for (path, dirs, files) in os.walk(input_Path):
        for filename in files:
            if filename.endswith(('.jpg','.png')):
                img_paths.append(path+'/'+filename)
    return img_paths
 
 
#构建Gabor滤波器
def build_filters():
     filters = []
     ksize = [7,9,11,13,15,17] # gabor尺度，6个
     lamda = np.pi/2.0         # 波长
     for theta in np.arange(0, np.pi, np.pi / 4): #gabor方向，0°，45°，90°，135°，共四个
         for K in range(6):
             kern = cv2.getGaborKernel((ksize[K], ksize[K]), 1.0, theta, lamda, 0.5, 0, ktype=cv2.CV_32F)
             kern /= 1.5*kern.sum()
             filters.append(kern)
     plt.figure(1)
 
     #用于绘制滤波器
     for temp in range(len(filters)):
         plt.subplot(4, 6, temp + 1)
         plt.imshow(filters[temp])
     plt.show()
     return filters
 
#Gabor特征提取
def getGabor(img,filters):
    res = [] #滤波结果
    for i in range(len(filters)):
        # res1 = process(img, filters[i])
        accum = np.zeros_like(img)
        for kern in filters[i]:
            fimg = cv2.filter2D(img, cv2.CV_8UC1, kern)
            accum = np.maximum(accum, fimg, accum)
        res.append(np.asarray(accum))
 
    #用于绘制滤波效果
    plt.figure(2)
    for temp in range(len(res)):
        plt.subplot(4,6,temp+1)
        plt.imshow(res[temp], cmap='gray' )
    plt.show()
    return res  #返回滤波结果,结果为24幅图，按照gabor角度排列
 
 
if __name__ == '__main__':
    input_Path = './content'
    filters = build_filters()
    img_paths = get_img(input_Path)
    for img in img_paths:
        img = cv2.imread(img)
        getGabor(img, filters)
```


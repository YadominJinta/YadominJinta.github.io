---
title: 图像压缩与线性代数
date: 2019-12-02
tags: 线性代数
layout: post
cover: https://i.loli.net/2019/12/07/MIp1AdkTaUe6Gl3.jpg
---

# 图像与线性代数

计算机中的图像，一般都是以矩形的形式呈现。它们由像素点组成，每一个像素点包含着各自的颜色信息。这样看的话，计算机中的图像确实与矩阵有着千丝万缕的联系。

# 为什么要压缩图像

假设现在我们有一张3000x3000像素的灰度图像（也就是一个3000x3000的矩阵），对于每一个像素来说，它都包含着0-255的灰度信息（即$2^8$，一个字节），所以这整张图片的大小为$9\times10^6$ Byte，8.53 Mb。

如果这张图片是彩色的，那么这张图片的每个像素都包含了RGB三种颜色的信息，以非常简单的256色为例，这张照片的大小也就是25.7Mb。

这样的大小显然是不可接受的，即便是对如今的电子设备来说。所以，我们需要对图片进行压缩。

如今的市面有很多压缩算法，其中不少都是无损压缩，而今天我想要聊一种有损压缩的算法--奇异值分解（SVD）

# 奇异值分解

## 简介

SVD 全称：Singular Value Decomposition。SVD 是一种提取信息的强大工具，它提供了一种非常便捷的矩阵分解方式，能够发现数据中十分有意思的潜在模式。

主要应用领域包括：

隐性语义分析 (Latent Semantic Analysis, LSA) 或隐性语义索引 (Latent Semantic Indexing, LSI)；  
推荐系统 (Recommender system)，可以说是最有价值的应用点；  
**矩阵形式数据（主要是图像数据）的压缩。**

## 线性变换

在了解SVD之前，我们先来了解一下线性变换，以这个简单的$2\times2$矩阵为例

$$M=\begin{bmatrix}
    3 & 0 \\
    0 & 1
\end{bmatrix} $$

从集合上讲， $M$ 是将二维平面上的点 $(x,y)$ 经过线性变换到另一个点的变换矩阵，如下所示：

$$ \begin{bmatrix}
    3 & 0 \\
    0 & 1
\end{bmatrix} 

\begin{bmatrix}
    x \\
    y    
\end{bmatrix}
=
\begin{bmatrix}
    3x \\
    y
\end{bmatrix}
$$

该变换的几何效果是，变换后的平面沿着x水平方向进行了3倍拉伸，垂直方向没有发生变化。

## 特征值与特征向量

首先了解一下一下特征值与特征向量:
$$ Ax = \lambda x$$

其中 $A$ 是一个$n\times n$矩阵， $x$是一个$n$维向量，则$\lambda$是矩阵$A$的一个特征值，而$x$是矩阵$A$的特征值$\lambda$所对应的特征向量。

这有什么意义呢？这样我们就可以可以将矩阵$A$特征分解。比如我们可以求出$A$的$n$个特征值 
$$\lambda_1 \le \lambda_2 \le \dots \lambda_n$$

则矩阵$A$可用下式表示：
$$ A = W\Sigma W^{-1} $$

其中W是这n个特征向量所张成的n×n维矩阵，而Σ为这n个特征值为主对角线的n×n维矩阵。

一般我们会把W的这n个特征向量标准化，即满足 $\vert\vert W_i\vert\vert _2=1$，或者 $W_i^T\times W_i=1$ ，此时W的n个特征向量为标准正交基，满足 $WW^T$=I ，即 $W^T=W^{-1}$ ，也就是说W为酉矩阵。

这样我们的特征分解表达式可以写成

$$A=W\Sigma W^T$$

## SVD

特征值分解是提取矩阵信息的好方式，但是仅对方阵而言。现实生活我们遇到大多数矩阵都不是方阵，这个时候就可以使用SVD：
$$A=U\Sigma V^T$$

其中$A$是一个$N \times M$的矩阵，那么得到的$U$是一个$N \times N$的方阵（里面的向量是正交的，$U$里面的向量称为左奇异向量），$\Sigma$是一个$N \times M$的矩阵（除了对角线的元素都是0，对角线上的元素称为奇异值），$V^T$(V的转置)是一个$N \times N$的矩阵，里面的向量也是正交的，$V$里面的向量称为右奇异向量）。

所以奇异值和特征值的对应关系是怎样的呢？
$$(A^TA)\nu_i=\lambda_i \nu_i$$

这里的$\nu_i$，其实就是上文所述的右奇异向量，此外，还有：
$$\sigma_i = \sqrt{\lambda_i}$$
$$\mu_i=\frac{1}{\sigma_i}A\nu_i$$

这里的$\sigma$就是上面说的奇异值，$\mu$就是上面说的左奇异向量。奇异值$\sigma$跟特征值类似，在矩阵$\Sigma$中也是从大到小排列，而且$\sigma$的减少特别的快，

对于矩阵A来说，可以被分解为
$$A=\sigma_1u_1v_1^T+\sigma_2u_2v_2^T+\dots+\sigma_ru_rv_r^T$$
在很多情况下，前10%甚至1%的奇异值的和就占了全部的奇异值之和的99%以上了。也就是说，我们也可以用前r大的奇异值来近似描述矩阵，这里定义一下部分奇异值分解：
$$A_{m\times n} \approx U_{m \times n} \Sigma_{r \times r} V^T_{r \times n}$$

# 压缩图像

由前文可知，可由矩阵前n个奇异值来大致的表示矩阵，这也是使用SVD压缩图像的原理，具体实现代码如下

``` python
import numpy as np
import numpy.linalg as la
import matplotlib.pyplot as plt
from sklearn import datasets
from skimage import io

img = io.imread('./flower.png')
# Read original image
plt.imshow(img)
plt.show()
# Show Original image
r_mat = np.mat(img[:,:,0])
g_mat = np.mat(img[:,:,1])
b_mat = np.mat(img[:,:,2])
# Split RGB Data

def runSVD(imgMat, k):
    # singular value decomposition
    U, s, V = la.svd(imgMat)
    # choose top k important singular values (or eigens)
    Uk = U[:, 0:k]
    Sk = np.diag(s[0:k])
    Vk = V[0:k, :]
    # recover the image
    imgMat_new = Uk * Sk * Vk
    return imgMat_new

top_k = 30

r_new = runSVD(r_mat,top_k)
g_new = runSVD(g_mat,top_k)
b_new = runSVD(b_mat,top_k)
# Use SVD to compress RGB data
r_new = np.int32(np.array(np.array(r_new).reshape((3000,3000,1))))
g_new = np.int32(np.array(np.array(g_new).reshape((3000,3000,1))))
b_new = np.int32(np.array(np.array(b_new).reshape((3000,3000,-1))))
new_img = np.concatenate((r_new,g_new,b_new),axis=2)
# generate new img

plt.imshow(img)
plt.show()
# Show new imgae
```

修改top_k的值（即所取的前n个奇异值）可以得到不同压缩率下的图片
原图  
![原图](/assets/img/compression5.png)  
top_k=10下，只能看清轮廓  
![top_k=10](/assets/img/compression1.png)  
top_k=30下，基本能看清了
![top_k=30](/assets/img/compression2.png)
top_k=50下，大部分细节得以保留  
![top_k=50](/assets/img/compression3.png)  
top_k=100下，很难分辨与原图有什么区别  
![top_k=100](/assets/img/compression4.png)
# 我对于MVP矩阵的理解



## 前言

关于MVP矩阵，其实已经有足够多的资料讲解，每个矩阵的具体含义这里就不再赘述。这里主要想分享和记录一下自己对于矩阵的不一样的理解。



常规理解中，对于M矩阵，一般都是分解为位移，旋转，缩放三个步骤。对于V矩阵，我们知道就是将世界空间的坐标转换成相机坐标。大部分情况下，都认为相机朝向-z的方向进行拍摄，因此可以认为是将相机放置到世界空间下的原点，且朝向-z观察，同样的，为了保持物体与相机相对位置的不变，物体也得做同样的变换，如下图

![image-20240328171108125](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/image-20240328171108125.png)

所以对于V矩阵，其实相当于不考虑缩放（相机的GameObject设置缩放好像也没什么意义？）的相机M矩阵的逆。



然而，这里我想说的是，在我们描述一个位置的时候，都需要先确定一个坐标系，然后用这个坐标系下的坐标值来进行描述。



而矩阵，其实就是包含了**坐标系**的信息。



## Model矩阵



### 概述

M矩阵，是为了将顶点从模型空间转到世界空间。在建模的时候，构造的顶点数据，都是以模型空间作为参考系。例如我们在创建一个矩形面片，顶点分别为（-1，-1），（1，1），（-1，1），（1，-1），是将坐标系设置为模型的中心点来描述的。



下面所有GameView中的矩阵均按照此顺序显示

![image-20240330234008604](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/image-20240330234008604.png)



且Unity的Matrix4x4定义为

```c#
public Matrix4x4(Vector4 column0, Vector4 column1, Vector4 column2, Vector4 column3)
{
  this.m00 = column0.x;
  this.m01 = column1.x;
  this.m02 = column2.x;
  this.m03 = column3.x;
  this.m10 = column0.y;
  this.m11 = column1.y;
  this.m12 = column2.y;
  this.m13 = column3.y;
  this.m20 = column0.z;
  this.m21 = column1.z;
  this.m22 = column2.z;
  this.m23 = column3.z;
  this.m30 = column0.w;
  this.m31 = column1.w;
  this.m32 = column2.w;
  this.m33 = column3.w;
}
```





当把这个模型放置于世界空间中，假设模型坐标系与世界坐标系重合，那么模型空间中的坐标即是世界空间的坐标，因为坐标系重合。

![image-20240330203017626](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/image-20240330203017626.png)

上图中的矩阵，即Cube的M矩阵。可以发现，**前三列**正好是对应**xyz**三个**轴向量**在**世界空间**中的表示。

那么当我们对Cube进行变换时，其实可以认为我们是在变换模型的坐标系，顶点的坐标值是在制作模型数据时就固定不变的，但是坐标系改变，模型在世界中的位置也随之改变。



### 位移

对于坐标系，本质上就是一组两两垂直的向量和原点，而向量是没有位移一说的。因此对于正交基（xyz轴）可以用矩阵的3x3部分就能表示，而第四列则可以用于表示坐标原点的位置。



例如这个时候，我们对Cube做一个位移，则可以得出新的原点位置，三个轴在世界空间中的描述依然是不变的。



![image-20240330204116976](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/image-20240330204116976.png)

可以看到，每一行均代表的是世界空间系下的轴分量，

第一行代表X轴分量，第二行代表Y轴分量，第三行代表Z轴分量。



### 旋转

那么什么时候会改变坐标系的轴呢？答案是**旋转**。



假如，我对Cube做一个绕y轴旋转的变换，则x轴和z轴的世界空间下的描述都会有所不同。

![image-20240330221026298](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/image-20240330221026298.png)



如上图所示，

模型坐标系的X轴在世界坐标系的描述为（cos60，0，-cos30），正好对应矩阵第一列的前三行数据

模型坐标系的Z轴在世界坐标系的描述为（cos30，0，cos60），正好对应矩阵第三列的前三行数据



到此，我们也只是通过实验推断轴的表示是按照列来进行的（Unity用的是列矩阵）

在Matrix4x4与向量的乘法中如下定义

```c#
public static Vector4 operator *(Matrix4x4 lhs, Vector4 vector)
{
  Vector4 vector4;
  vector4.x = (float) ((double) lhs.m00 * (double) vector.x + (double) lhs.m01 * (double) vector.y + (double) lhs.m02 * (double) vector.z + (double) lhs.m03 * (double) vector.w);
  vector4.y = (float) ((double) lhs.m10 * (double) vector.x + (double) lhs.m11 * (double) vector.y + (double) lhs.m12 * (double) vector.z + (double) lhs.m13 * (double) vector.w);
  vector4.z = (float) ((double) lhs.m20 * (double) vector.x + (double) lhs.m21 * (double) vector.y + (double) lhs.m22 * (double) vector.z + (double) lhs.m23 * (double) vector.w);
  vector4.w = (float) ((double) lhs.m30 * (double) vector.x + (double) lhs.m31 * (double) vector.y + (double) lhs.m32 * (double) vector.z + (double) lhs.m33 * (double) vector.w);
  return vector4;
}
```

那么下面从运算的角度再来验证一下，考虑用一个3维向量乘一个3x3矩阵，会发生什么，

假设**向量是一个模型空间下的向量，3x3矩阵是模型空间正交基**（不这么假设计算与上面的推论构不成意义）
$$
\begin{bmatrix}
&m00&m01&m02\\
&m10&m11&m12\\
&m20&m21&m22\\
 \end{bmatrix}* (x,y,z)^T 

 = (x·m00+y·m01+z·m02, 
 x·m10+y·m11+z·m12,   
 x·m20+y·m21+z·m22)
$$
可以看到，得到新的向量，xyz分量分别为原向量与矩阵每一行向量的点乘结果，而点乘的意义正好就是投影。

对于上面的矩阵，第一列为X轴，第二列为Y轴，第三列为Z轴。

第一行为三个轴向量在世界空间下的x分量。

第二行为三个轴向量在世界空间下的y分量。

第三行为三个轴向量在世界空间下的z分量。

上面的运算结果得到的新的向量，则变成了**世界空间**下的向量。

新向量x分量为将原向量投影到矩阵正交基在世界空间下x分量

新向量y分量为将原向量投影到矩阵正交基在世界空间下y分量

新向量z分量为将原向量投影到矩阵正交基在世界空间下z分量



这么说可能有点绕，可以用一个简单的例子来说明

![f5085f055eaa8f46d4da8c0f65b613d](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/f5085f055eaa8f46d4da8c0f65b613d.jpg)







当我们对Cube任意旋转之后，依然可以从矩阵得出新的模型坐标系xyz轴在世界空间下的表示

![image-20240330224640935](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/image-20240330224640935.png)







### 缩放



倘若我们此时做一下缩放会发生什么呢？

![image-20240330224754336](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/image-20240330224754336.png)

可以看到，模型坐标系的x轴向量表示，三个分量均被放大2倍，很合理，因为我们就是放大的顶点的x坐标。



对三个轴都进行放大，可以看到，其实可以认为就是对坐标轴刻度的放大。

![image-20240330225036546](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/image-20240330225036546.png)



**关于为什么是先旋转缩放再平移？**



我们可以把M矩阵拆分为3x3部分和齐次的部分。对于tx，ty，tz，前面我们说过可以认为表示模型坐标系原点的位置，它是一个世界空间系下的量。而3x3部分则是正交基。

假设我们把顶点当作是模型空间系下的向量，将模型空间系下的向量左乘正交基矩阵，则表示把向量转到了世界空间，把（tx，ty，tz）也当作一组向量，进行相加，则正好得到世界空间下由原点指向顶点的向量。

![image-20240331214822676](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/image-20240331214822676.png)

如上图所示，p为模型顶点，蓝色的向量代表齐次矩阵的xyz位移量的向量表示，而黄色矩阵则是由模型中心指向顶点p的向量。

对于蓝色向量，是一个世界坐标系下的向量，对于黄色向量则是模型坐标系下。

倘若先进行位移，即黄色向量与蓝色向量相加，两个在不同坐标系下的向量相加是没有意义的。

因此需要先将黄色向量乘3x3正交基转换到世界坐标系，然后再于蓝色向量相加。

即可以得到顶点p在世界空间中的位置。



## View矩阵

V矩阵可以理解为相机M矩阵的逆，而前面3x3部分则是正交基部分，下面的**right，up，forward**均为相机**transform**的。

对于正交基，转置就是逆。因此，对于M矩阵正交基，原本是
$$
\begin{bmatrix}
&right.x &up.x &forward.x\\
&right.y &up.y &forward.y\\
&right.z &up.z &forward.z\\
\end{bmatrix}
$$
而因为相机空间是朝向-z进行观察，z轴需要反向（右手坐标系）。得矩阵应该为
$$
\begin{bmatrix}
&right.x &up.x &-forward.x\\
&right.y &up.y &-forward.y\\
&right.z &up.z &-forward.z\\
\end{bmatrix}
$$
转置后作为V矩阵的正交基为
$$
\begin{bmatrix}
&right.x &right.y &right.z\\
&up.x &up.y &up.z\\
&-forward.x &-forward.y &-forward.z\\
\end{bmatrix}
$$
在HDRP中有这么一段shader

```c
// Returns the forward (central) direction of the current view in the world space.
float3 GetViewForwardDir()
{
    float4x4 viewMat = GetWorldToViewMatrix();
    // 实际上，这里就是做了个右手系到左手系的转化。
    return -viewMat[2].xyz;
}
```

在微软的文档中说到对于矩阵的访问

![image-20240331225551767](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/image-20240331225551767.png)

因此GetViewForwardDir中，ViewMat[2]获取的实际上正好是-forward，再取反即可得到相机的forward。

![image-20240331225802970](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/image-20240331225802970.png)

综上，相机View矩阵最终形态应该为
$$
\begin{bmatrix}
&right.x &right.y &right.z &-tx\\
&up.x &up.y &up.z &-ty\\
&-forward.x &-forward.y &-forward.z &-tz\\
&0 &0 &0 &1
\end{bmatrix}
$$


下面来进行验证。

![image-20240331233035754](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/image-20240331233035754.png)

首先如上图，可以看到相机View矩阵的z轴跟transform的forward正好反向。在不旋转的情况下，tx，ty做了反向，而因为z轴在左右手坐标系中是反向的。

TODO。。。。



## Projection矩阵





### 正交投影



![image-20240403134415407](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/image-20240403134415407.png)





### 透视投影










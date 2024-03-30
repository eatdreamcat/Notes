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



## View矩阵





## Projection矩阵





### 正交投影







### 透视投影










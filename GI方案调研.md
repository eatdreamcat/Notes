# GI方案调研

## Local Radiance Transfer



### 元胞自动机

是一种[离散模型](https://zh.wikipedia.org/w/index.php?title=離散模型&action=edit&redlink=1)。它是由无限个有规律、坚硬的方格组成，每格均处于一种有限[状态](https://zh.wikipedia.org/wiki/狀態)。整个格网可以是任何有限维的。同时也是[离散](https://zh.wikipedia.org/wiki/離散)的。每格于t时的态由t-1时的邻域的状态决定。

![img](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/Gospers_glider_gun.gif)



细胞自动机有三个特征：

- 平行计算（parallel computation）：每一个细胞个体都同时同步的改变
- 局部的（local）：细胞的状态变化只受周遭细胞的影响。
- 一致性的（homogeneous）：所有细胞均受同样的规则所支配



核心在于状态和规则的建模。



### DDGI

全称Dynamic Diffuse Global Illumination。





### PRT

![diagram of how prt works](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/prt-lightingpicture.png)

- Source Radiance：表示环境光源，在PRT中使用球谐来近似
- Exit Radiance：表示光经过物体交互后，从物体表面任意方向射出的量
- Transfer Vector：用于表示入射量跟出射量的对应关系，横坐标表示入射角度，纵坐标表示出射角度。需要离线计算好。



PRT把渲染分为了两个阶段：

![diagram of prt data flow](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/prt-dataflow.png)







### LPV

全称Light Propagation Volumes，核心思想是将光照信息注入到体积网格中，并通过体积网格传播光照，从而实现间接照明。

以下是 LPV 全局光照算法的主要步骤：



#### 1. 场景体素化 (Voxelization)

首先，将场景体素化，即将场景的几何信息转换为体素（3D 像素）网格。体素化的过程如下：

- 将场景中的几何体转换为适合体素网格的数据结构。
- 每个体素记录几何体的颜色、法线和其他相关信息。

#### 2. 光源注入 (Injecting Light)

将直接光源的信息注入到体素网格中。这一步通常包括以下过程：

- 计算直接光源照射到体素网格的颜色和强度。
- 将这些光照信息存储在体素网格的相应位置。

#### 3. 光传播 (Light Propagation)

通过体素网格传播光照信息，以模拟间接光照。光传播步骤如下：

- 初始化光照体积：创建光照体积，并初始化为零。
- 将注入的光源信息传播到周围的体素中。通常使用一系列的体积滤波器来模拟光线在体素网格中的传播。
- 在每个体素中，计算光照的传播方向和强度，更新光照体积。

#### 4. 光照合并 (Combining Lighting)

将传播后的光照信息与直接光照信息合并，以生成最终的光照结果：

- 直接光照：通过常规的光照计算（如光栅化）得到直接光照。
- 间接光照：从光照体积中采样得到间接光照。
- 将直接光照和间接光照结合，得到场景的最终光照。

#### 5. 应用光照 (Applying Lighting)

将计算好的光照应用到场景中，进行最终渲染：

- 在每个像素中，采样光照体积，获取间接光照。
- 合并直接光照和间接光照，计算最终的像素颜色。
- 将结果渲染到屏幕上。





### Photon Mapping

在[计算机图形学](https://zh.wikipedia.org/wiki/计算机图形学)中，photon mapping是一个2 pass的全局[光照算法](https://zh.wikipedia.org/w/index.php?title=光照算法&action=edit&redlink=1)，发明者为Henrik Wann Jensen。其做法为：分别从光源和照相机发出射线，当满足终止条件时，两条射线被连接在一起，用来在下一步中产生辐射度数值。

![img](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/300px-Glas-1000-enery.jpg)



#### 1.光子图创建

在光子映射算法中，光子从光源发射到场景中。一旦光子和一个几何面（surface）相交，相交点和入射方向就会被存在一个叫光子贴图（photon map，一般使用kd-tree来存储）的缓存中。一般来讲我们会存两张贴图，一个给caustics另一个给其他光。

![img](https://users.csc.calpoly.edu/~zwood/teaching/csc572/final15/dschulz/images/pointVsarea.png)

上图为点光跟面光的光子发射示意图，当光子与物体表面交互时，通过俄罗斯轮盘赌的方式来决定光子的行为（反射，折射，吸收等）。



#### 2.渲染

在这个阶段，使用光子图估算每个表面点的间接光照：

- **直接光照**：通过常规的光栅化或光线追踪技术计算直接光照。
- **间接光照**：在渲染过程中，当光线与表面相交时，从光子图中检索相邻光子，并计算这些光子的密度。通过密度估算间接光照的强度。
- **最终合成**：将直接光照和间接光照合成，生成最终的图像。





### 算法流程



![img](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/v2-a129dbb0f1ac80069f83f636721cfa1a_1440w.webp)



#### 局部空间近似



把一个局部空间近似成一个球形空间。

![img](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/v2-7b2376d4b0166a43e10b12fc0d234b47_1440w.webp)

![img](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/v2-db701984086a753b98c2b57e3bccf111_1440w.webp)



然后在这个球空间的基础上，计算SH，包括SH_BaseColor，SH_localVisibility。



#### 光照传输



参考元胞自动机的思想，每个球只考虑邻域内26个球的光照信息。对于邻域的采样，使用分帧进行操作。

![img](https://raw.githubusercontent.com/eatdreamcat/PicGo-01/main/v2-ff019f982ae8906930fc018deee1560b_1440w.webp)

#### 光照注入





#### Screen Space Gather







#### CPU 场景管理



## 参考

[LPV in CryEngine](https://advances.realtimerendering.com/s2009/Light_Propagation_Volumes.pdf)

[Local Radiance Transfer](https://zhuanlan.zhihu.com/p/653044045)

[Precomputed Randiance Transfer](https://jankautz.com/publications/prtSIG02.pdf)


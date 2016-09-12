#3 基本图像滤波运算

##3.1 介绍
对于N*N的图像，FFT将时间复杂度从$O(N^3)$降至$O(N^2*\log_{2}N)$。
相比傅里叶正反变化及频域相乘操作，直接在空间域卷积或许会节省计算量。
##3.2 通过高斯平滑的噪声抑制
低通滤波器锋利截断会产生震荡，e.g. 矩形窗<->(sinx/x)。
高斯滤波器：
f(x) = $\dfrac{1}{\left(2\pi\sigma^{2}\right)^{\dfrac{1}{2}}}e^{-\dfrac{x^{2}}{2\sigma ^{2}}}$
$F(\omega)$ = $e^{-\dfrac{1}{2}\sigma^2\omega^2}$
一种接近高斯性质的卷积蒙板：
$\dfrac{1}{16}\begin{bmatrix} 1 & 2 & 1 \\ 2 & 4 & 2 \\1 & 2 & 1 \end{bmatrix}$
尽管抑制了噪声，同时有用信号也被影响。
##3.3 中值滤波
###限幅滤波器("limit" filter)
	for all pixels in image do {
	    minP = min(P1,P2,P3,P4,P5,P6,P7,P8);
	    maxP = max(P1,P2,P3,P4,P5,P6,P7,P8);
	    if(P0 < minP)Q0 = minP;
	    else if(P0 > maxP)Q0 = maxP;
	    else Q0 = P0;
	}

###中值滤波器(median filter)
	for(i = 0; i <= 255; i++)hist[i]=0;
	for all pixels in image do {
	    for(m = 0; m <= 8; m++)hist[P[m]]++;
	    i = 0; sum = 0;
	    while(sum < 5){
	        sum = sum + hist[i];
	        i = i + 1;
	    }
	    Q0 = i - 1;
	    for(m = 0; m <= 8; m++)hist[P[m]] = 0;
	}

中值滤波器在抑制脉冲噪声上有很好的表现，并且减少了模糊，弥补了高斯平滑滤波器的主要缺陷。
对于n*n邻域的中值滤波，若采用冒泡排序，时间复杂度高达$O(n^4)$所以n通常不大于4。
##3.4 众数滤波器(mode filters)
由于区域亮度分布只计算邻域中非常少的亮度值，所以我们很可能得到多峰值的分布，而分布的最高点不一定能象征潜在的众数。所以在计算众数前分布需要进行相当的平滑(smooth out)。
边缘附近的分布固有存在双峰，平滑图像一般为单峰。
若邻域横跨一条边缘，并且亮度分布是双峰的，则应该选取较大的峰作为该点亮度值。选取较大峰的一种好的策略是消除较小峰。
众数的位置能在中值确定后合理的估计。
首先找到众数mode，然后找到与众数最接近的极限值e，向相反方向移动众数与极限值之间的距离得到截断值2*mode-e。由于最初不知道众数，可以用中值代替来估计。
截断后分布的中值更接近众数，可以通过迭代的方式用中值来逼近众数。
TMF(truncated median filter)，截断中值滤波器，不仅去除了噪声，还增强了边界。
TMF的一种实现：

	do{//as many passes over image as necessary
	    for all pixels in image do{
	        compute local intensity distribution;
	        do{//iterate to improve estimate of mode
	            find minimum,median,and maximum intensity values;
	            decide from which end local intensity distribution should be truncated;
	            deduce where local intensity distribution should be truncated;
	            truncate local intensity distribution;
	            find median of truncated local intensity distribution;
	        }until median sufficiently close to mode of local distribution;
	        transfer estimate of mode to output image space;
	    }
	}until sufficient enhancement of image;

使用场景：
中值滤波器：需要较大的脉冲信号抑制能力。
众数滤波器 or TMF：锐化边缘来增强图像。
尽管TMF设计更多的是用来图像增强，但是实际发现当脉冲噪声非常多时，TMF的抑制效果很好。
##3.5 排序滤波器(rank order filters)
邻域n个值中的第r位作为输出。中值滤波器可以看作r=(n+1)/2的排序滤波器。
##3.6 减少计算压力
对于256*256的图像，用30*30的高斯卷积蒙板来平滑，需要6400万次基本运算。
可以将二维高斯卷积分解成一位高斯卷积：
$e^{-\dfrac{r^2}{2\sigma^2}}$ = $e^{-\dfrac{x^2}{2\sigma^2}}e^{-\dfrac{y^2}{2\sigma^2}}$
将3*3高斯滤波器分解成：
$\dfrac{1}{16}\begin{bmatrix} 1 & 2 & 1 \\ 2 & 4 & 2 \\1 & 2 & 1 \end{bmatrix}$ = $\dfrac{1}{4}\begin{bmatrix} 1 \\ 2 \\ 1 \end{bmatrix}$$\dfrac{1}{4}\begin{bmatrix} 1 & 2 & 1\end{bmatrix}$
可以将一个$O(n^2)$的运算分解成2个$O(n)$的运算。
高斯滤波器的分解是精确而不是近似的，而中值滤波器无法不近似的分解。通常的做法是依次进行两次一维的中值滤波。
不能精确分解并不是非线性滤波器的特点，很多线性滤波器也不能精确分解。因为n*n蒙板系数$n^2$大于n*1和1*n蒙板的系数和。
##3.7 锐化-反锐化蒙板(Sharp-Unsharp Masking)
将原图像进行模糊处理，并用原图像去减模糊的图像。
##3.8 中值滤波器引进的移位
###3.8.1 连续二值图像模型
![图3-1](http://img.blog.csdn.net/20160525162234770)
如图，对于连续二值图像，对于r=b的圆形边界B（图像中圆B内部应为黑色），以r=a的圆A为邻域，进行中值滤波，获得到的边界D(r=d)，有：
两个圆形重叠部分面积$C = \dfrac{\pi a^2}{2}$
重叠部分面积$C = S_a + S_b$
其中$S_a$为两圆交点连接成的弦与圆A被弦分割出的劣弧所围成区域面积，$2\alpha$为圆A圆心O(A)到该弦的夹角；$S_b$为两圆交点连接成的弦与圆B被弦分割出的劣弧所围成区域面积，$2\beta$为圆B圆心O(B)到该弦的夹角。
以及：
$S_a$ = $a^2(\alpha - sin\alpha cos\alpha)$
$S_b$ = $b^2(\beta - sin\beta cos\beta)$
$a^2 = b^2 + d^2 -2bdcos\beta$
$b^2 = a^2 + d^2 -2adcos\alpha$

当$a<<b$，$\alpha \approx \pi/2$,$d\approx b$,$\beta \approx \dfrac{a}{b}$
由中值滤波引进的边界半径减小$\Delta =b-d\approx \dfrac{a^2}{6b}= \dfrac{\kappa a^2}{6}$其中曲率$\kappa = 1/b$
###3.8.2 推广到灰度图像
同样的，对于直线边界，中值滤波不带来移位。
对于曲线边界，灰度图像具有有限斜率。我们只需要找到一条等亮线，能把邻域中的图像分为两个相同大小的部分。
##3.9 中值移位的离散模型
同样的，对于直线边界，不带来移位。
假设一个像素的亮度是整个像素区域的平均值。
对于3×3邻域，考虑中值等亮度线的曲率为$\kappa$，等亮度线经过中点附近时，法线与x轴夹角为$\theta$，那么位移$D_\theta \approx \dfrac{1}{2}\kappa a_0^2-a_0\theta = \dfrac{1}{2}\kappa -\theta$
其中$a_0$为像素间距，这里取1个单位。
当中值落在中间一列的上下两个像素，设曲率半径$b=\dfrac{1}{\kappa }$，有$a_0^2=D*(2b-D)\approx 2Db$
##3.10 众数滤波器引进的移位
同样的，对于直线边界，由于对称性，不会带来移位。
对于阶跃曲线边界，众数滤波器与中值滤波器表现相同，由众数滤波引进的边界半径减小$\Delta =b-d\approx \dfrac{a^2}{6b}= \dfrac{\kappa a^2}{6}$其中曲率$\kappa = 1/b$
对于亮度平滑变化的边界，圆形邻域C中，等亮度线最长的那条成为结果的亮度。显而易见这条等亮度线与C的两个交点，相连是一条直径。我们需要计算该等亮度线的中点与直径中点的偏差，模型同3.9中最后一例，有$a_0^2=D*(2b-D)\approx 2Db$，于是，$D=\dfrac{1}{2}\kappa a^2$
邻域平均滤波器的移位概括：
|边缘类型|均值滤波器|中值滤波器|众数滤波器|
|:------------:|:-------------:|:-------------:|:-------------:|
|阶跃|$\dfrac{1}{6}\kappa a^2$|$\dfrac{1}{6}\kappa a^2$|$\dfrac{1}{6}\kappa a^2$|
|中间型?(Intermediate)|~$\dfrac{1}{7}\kappa a^2$|$\dfrac{1}{6}\kappa a^2$|$\dfrac{1}{2}\kappa a^2$|
|线性|$\dfrac{1}{8}\kappa a^2$|$\dfrac{1}{6}\kappa a^2$|$\dfrac{1}{2}\kappa a^2$|
##3.11 均值滤波器和高斯滤波器引进的移位
同样的，对于直线边界，不会带来移位。
对于阶跃曲线边界，模型与中值和众数滤波器相同，边界半径减小$\Delta =b-d\approx \dfrac{a^2}{6b}= \dfrac{\kappa a^2}{6}$其中曲率$\kappa = 1/b$

对于亮度平滑变化的边界，直观的得到$D=\dfrac{1}{8}\kappa a^2$
##3.12 排序滤波器引进的移位
//TBD
##3.13 滤波器在视觉的工业应用中的作用
中值滤波器在抑制噪声的同时会清除一些有用的细节。
边界移位是噪声抑制滤波器的普遍特征。
可以采用边缘检测算法，然后在一个整体内抑制噪声。
##3.14 彩色图像滤波
颜色会增加计算量，但是在评估水果成熟度等应用中非常有用。
中值滤波在彩色域中没有定义，一种简单的方式是分别对三个通道进行中值滤波，然后重新组成彩色图像。
但是会带来很多问题，最明显的一个就是渗色（bleeding）。这种情况发生在脉冲噪声只发生在一个通道且该点处于边界附近。
一种标准的解决方案：
单通道的度量 median = $arg\ min_i\sum_j{|d_{ij}|}$
扩展到三色彩域 median = $arg\ min_i\sum_j
 {|\widetilde{d_{ij}}|}$
其中$\widetilde{d_{ij}}$ = $[\sum_{k=1}^3(I_{i,k}-I_{j,k})^2]^{1/2}$，$I_i,I_j$分别是RGB向量。
尽管矢量中值滤波器（VMF）不再分开对待三个通道，但是绝不保证完全消除渗色效应。
对于颜色的交汇处，会迷惑这些滤波算法，并无意中带来少量的渗色效应。
矢量中值滤波器和矢量众数滤波器都显著的拜托了渗色效应，标量众数滤波器和标量中值滤波器一样会产生渗色效应
##3.15 结论
本章详述了基于邻域的噪声抑制和图像增强算法的实现。
展示了针对特定需求设计特定算法的重要性：不仅有效实现功能，并且针对速度、存储等条件优化。
另外论述了选择策略时要考虑和处理不利的属性，例如移位。
接着论述了一些处理离散化图像的基本问题。
最后，特定类型的排序滤波器的边界移位十分重要，因为它可以在形态学处理时转化为有利条件。
下一章将介绍图像分割，以寻找图像中目标的位置。
##3.16 参考文献
//忽略

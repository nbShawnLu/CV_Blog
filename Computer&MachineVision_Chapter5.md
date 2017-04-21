#边缘识别
##5.1 介绍
边缘识别一直作为图像分割的一个可供选择的方式，因为其在额外数据空间和计算量的优势（典型的可以节约100倍）。
两种主要的方式：
1.template matching（TM），模板匹配。
2.differential gradient（DG），梯度差分。
两种方式都是寻找足够大的梯度幅值g，以作为边缘的象征。
两种方式区别在于评价局部梯度g的方式不同，以及判断边缘方向的方式不同。
##5.2 边缘识别的基本理论
计算g：
TM：对12个评估不同方向梯度的模板卷积，取最大值。
$g=max(g_i:i=1\ to\ n)$
DG：仅需两个模板，分别评价x方向梯度$g_x$和y方向梯度$g_y$，使用非线性变换。
$g=(g_x^2+g_y^2)^{1/2}$
为了节省计算量，通常被简化为$g=|{g_x}|+|g_y|$或$g=max(|{g_x}|,|g_y|$)
计算方向：
TM：使用g取最大值的i对应的方向。
DG：$\theta=arctan\frac{g_y}{g_x}$
比较：
DG需要更多计算量，尽管更加精确。由于很多时候不需要方向数据，或者图像对比度范围较大，取得精确的g值意义不大，所以TM被较为广泛的使用。两者都涉及局部梯度的评估，TM和DG的卷积蒙版经常完全相同。
DG算子：
|---|---|---|
|---|---|---|
|Roberts 2*2|$R_x$=\begin{bmatrix}
0 & 1\\
-1 & 0\\
\end{bmatrix}|$R_y$=\begin{bmatrix}
1 & 0\\
0 & -1\\
\end{bmatrix}
|Sobel 3*3|$S_x$=\begin{bmatrix}
-1 & 0 & 1\\
-2 & 0 & 2\\
-1 & 0 & 1\\
\end{bmatrix}|$S_y$=\begin{bmatrix}
1 & 2 & 1\\
0 & 0 & 0\\
-1 & -2 & -1\\
\end{bmatrix}
|Prewitt 3*3|$P_x$=\begin{bmatrix}
-1 & 0 & 1\\
-1 & 0 & 1\\
-1 & 0 & 1\\
\end{bmatrix}|$P_y$=\begin{bmatrix}
1 & 1 & 1\\
0 & 0 & 0\\
-1 & -1 & -1\\
\end{bmatrix}
##5.3 TM
另外6个角度可以通过外圈系数的旋转或者对称变换得到
|---|$0^\circ$|$45^\circ$|
|---|---|---|
|Prewitt|\begin{bmatrix}
-1 & 1 & 1\\
-1 & -2 & 1\\
-1 & 1 & 1\\
\end{bmatrix}|\begin{bmatrix}
1 & 1 & 1\\
-1 & -2 & 1\\
-1 & -1 & 1\\
\end{bmatrix}
|Kirsch|\begin{bmatrix}
-3 & -3 & 5\\
-3 & 0 & 5\\
-3 & -3 & 5\\
\end{bmatrix}|\begin{bmatrix}
-3 & 5 & 5\\
-3 & 0 & 5\\
-3 & -3 & -3\\
\end{bmatrix}
|Robinson 3-Level|\begin{bmatrix}
-1 & 0 & 1\\
-1 & 0 & 1\\
-1 & 0 & 1\\
\end{bmatrix}|\begin{bmatrix}
0 & 1 & 1\\
-1 & 0 & 1\\
-1 & -1 & 1\\
\end{bmatrix}
|Robinson 5-Level|\begin{bmatrix}
-1 & 0 & 1\\
-2 & 0 & 2\\
-1 & 0 & 1\\
\end{bmatrix}|\begin{bmatrix}
 0 & 1 & 2\\
-1 & 0 & 1\\
-2 & 1 & 0\\
\end{bmatrix}
##5.4 3*3模板算子的理论
由于另外6个方向的蒙版和$0^\circ$和$45^\circ$的只有符号之间的区别，仅用这两者表示。
\begin{bmatrix}
-A & 0 & A\\
-B & 0 & B\\
-A & 0 & A\\
\end{bmatrix}
\begin{bmatrix}
 0 & C & D\\
-C & 0 & C\\
-D & -C & 0\\
\end{bmatrix}

由于$g_{45}=\frac{g_0+g_{90}}{\sqrt{2}}$
解得
$C=\frac{B}{\sqrt{2}}$
$D=A\sqrt{2}$
进一步根据$22.5^\circ$处的梯度，得到
$\frac{B}{A}=\sqrt{2}\frac{9t^2-(14-4\sqrt{2})t+1}{t^2-(10-4\sqrt{2})t+1}$
其中$t=tan22.5^\circ$
$\frac{B}{A}=\frac{13\sqrt{2}-4}{7}=2.055$
##5.5 DG算子的设计
离散图像取方形邻域进行估计会导致梯度方向最大$6.63^\circ$的偏离。
##5.6 圆形算子的概念
##5.7 圆形算子的实现细节

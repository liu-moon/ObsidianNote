ORB（Oriented FAST and Rotated BRIEF）特征点出自美国公司在2012年发表的一篇论文
## ORB特征点
特征点由关键点（Keypoint）和描述子（Descriptor）两部分组成。
关键点：特征点在图像中的位置
描述子：量化关键点周围的像素信息
这里的“量化”一般是认为设计的某种方式，设计的原则是“外观相似的特征应该具有相似的描述子”。
如果想判断两个位置的关键点是否相似时，可以通过计算他们之间的描述子距离来确定。
ORB的关键点为Oriented FAST，描述子为Steered BRIEF。
### 关键点 Oriented FAST
#### FAST角点
FAST角点是一种检测角点的方法。这种方法确定特征点的速度非常快。
判断是否是角点可以通过在图像中取一个小窗口，窗口内的像素如果沿所有方向都有灰度变化，说明该点是角点。
![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/20230922131440.png)
在FAST中不再需要使用窗口统计的思想，而是通过另一种更快的方式，总的来说就是：如果一个像素和它周围的像素灰度差别较大（超过设定的阈值），而且达到一定数目，那么这个像素就很可能是角点。
具体的检测过程如下：
1. 在图像中选择某个像素$p$，将它的灰度值记为$I_p$
2. 设定一个阈值$T$，用于判断两个像素灰度值的差异大小。为了能够适应不同的图像，一般采用相对百分比例，比如设置为$I_p$的20%。
3. 以像素$p$为中心，选取半径为3的圆上的16个像素点。选取方式如下图所示。
   ![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/20230922131945.png)
4. 如果16个像素点中有连续的N个点的灰度大于$I_p+T$或者小于$I_p-T$，就可以将像素$p$确定为关键点。在ORB论文中，作者说$N=9$时效果较好，称为FAST-9。在实际操作中，为了加速，可以把第1、5、9、13个像素点当作锚点。在FAST-9算法中，只有当这4个锚点中有3个及以上锚点的灰度值同时大于$I_p+T$或者小于$I_p-T$时，当前像素才可能是一个关键点，否则就可以排除掉，这大大加快了关键点检测的速度。
5. 遍历图像中每个像素点，循环执行以上4个步骤。

#### 图像金字塔
但是FAST角点还存在尺度问题，所谓的尺度问题产生的原因如下：
由于FAST固定选取的是半径为3的圆，那这个关键点就和相机拍摄物体的分辨率息息相关了，当在初始位置通过FAST-9判定是关键点，但是当相机靠近或者后退的时候，该点的大小发生变化，又因为固定选取的半径是不变的，就会产生无法检测到该点的情况。
![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/20230922133001.png)
ORB特征点使用图像金字塔来确保特征点的尺度不变性。图像金字塔是计算机视觉领域中常用的一种方法，如图所示
![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/20230922133027.png)
金字塔底层是原始图像，在ORB-SLAM2中对应的金字塔层级是level=0。每往上一层，就对图像进行一个固定倍率的缩放，得到不同分辨率的图像。当提取ORB特征点时，我们会在每一个金字塔层级上进行特征提取，这样不管相机拍摄时距离物体是远还是近，都可以在某个金字塔层级上提取到真正的角点。我们在对不同图像特征点进行匹配时，就可以匹配不同图像中在不同金字塔层级上提取到的特征点，实现尺度不变性。
#### 灰度质心法
ORB特征点的旋转不变性是通过以下方式实现的：
先想办法计算出每个关键点的“主方向”，然后统一将像素旋转到这个“主方向”，使得每个特征点的描述子不再受旋转的影响。
计算关键点的主方向的方法为灰度质心法。这里的灰度质心就是将一个图像区域内像素的灰度值作为权重的中心，是需要我们计算的。这里的图像区域一般是圆形区域，在ORB-SLAM2中设定的是直径为31的圆形。这个圆形的圆心叫做形心，也就是几何中心。形心指向质心的向量就代表这个关键点的“主方向”。
我们定义该区域图像的矩为（类似物理中的力矩）：
$$m_{pq} = \sum_{x,y}x^py^qI(x,y),\quad  p,q=\left \{ 0,1 \right \} $$
式中，$p,q$取0或1；$I(x,y)$表示在像素坐标$(x,y)$处图像的灰度值；$m_{pq}$表示图像的矩。
图形区域内所有像素的灰度值总和为
$$\begin{align}{} 
  m_{00} = \sum_{x=-R}^{R}\sum_{y=-R}^{R}I(x,y)
\end{align}$$
在半径为$R$的圆形图像区域，沿两个坐标轴$x,y$方向的图像矩分别为
$$\begin{align}{} 
  m_{10} = \sum_{x=-R}^{R}\sum_{y=-R}^{R}xI(x,y)\\m_{01} = \sum_{x=-R}^{R}\sum_{y=-R}^{R}yI(x,y)
\end{align}$$
图像的质心为
$$\begin{align} 
  C = \left ( c_x,c_y \right )=\left ( \frac{m_{10}}{m_{00}}, \frac{m_{01}}{m_{00}} \right )  
\end{align}$$
关键点的“主方向”可以表示为从圆形图像形心$O$指向质心$C$的方向向量$\overrightarrow{OC}$，于是关键点的旋转角度记为
$$\theta = arctan2(c_y,c_x)= arctan2(m_{01},m_{10})$$
以上就是利用灰度质心法求关键点旋转角度的原理。
ORB-SLAM2代码使用了一些技巧加速计算灰度质心，原理和流程如下：
![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/20230923102320.png)

1. 要处理的是一个圆形图像区域，加速的原理是根据对称性一次索引多行像素。首先把索引基准点放在圆形的中心像素点上，记为center
2. 图形半径记为R，先计算圆形区域内水平坐标轴上的一行像素灰度（图中红色区域），对应的坐标范围是（-R<x<R,y=0）。这一行对应的图像矩分别为
   $$\begin{align}{} 
  m_{10}^{center} &= \sum_{-R<x<R,y=0}xI(x,y)\\m_{01}^{center} &= \sum_{-R<x<R,y=0}yI(x,y)=0
\end{align}$$
3. 以水平坐标轴为对称轴，一次性索引与水平坐标轴上下对称的两行像素（图中绿色区域），上下某两个对称的像素分别记为
   $$\begin{align}{} 
  p_{up} &= (x',-y')\\p_{bottom} &= (x',y')
\end{align}$$
则这两个点对应的图像矩分别为
$$\begin{align}{} 
  m_{10}^{up'} &= x'I(p_{bottom})+x'I(p_{up}) = x'(I(x',y')+I(x',-y'))\\
  m_{01}^{bottom'} &= y'I(p_{bottom})-y'I(p_{up}) = y'(I(x',y')-I(x',-y'))
\end{align}$$
最后累加即可。

### 描述子 Steered BRIEF
前面我们用Oriented FAST确定了关键点，下面就要对每个关键点的信息进行量化，计算其描述子。ORB特征点中的描述子是在BRIEF的基础上进行改进的，称为Steered BRIEF。我们先来了解什么是BRIEF。
BRIEF是一种二进制编码的描述子，在ORB-SLAM2中它是一个256bit的向量，其中每个bit是0或1。这样我们在比较两个描述子时，可以直接用异或位运算计算汉明距离，速度非常快。
以下是BRIEF描述子的具体计算方法。
1. 为减少噪声干扰，先对图像进行高斯滤波。
2. 以关键点为中心，取一定大小的图像窗口p。在窗口内随机选取一对点，比较二者像素的大小，进行如下二进制赋值。
   $$\tau(p ; x, y):=\left\{\begin{array}{ll}
1 & : p(x)<p(y) \\
0 & : p(x) \geqslant p(y)
\end{array}\right.$$
	式中，$p(x)$表示像素$x$在窗口$p$内的灰度值
3. 在窗口中随机选取$N$（在ORB-SLAM2中$N$=256）对随机点，重复第2步的二进制赋值，最后得到一个256维的二进制描述子。

对于上述的第2步，在ORB-SLAM2中采用了一种固定的选点模板。这个模板是精心设计的，保证了描述子具有较高的辨识度。下面是模板的前四行。

```c++
// 语句定义了一个名为`bit_pattern_31_`的静态整数数组
// 数组的大小为 256*4
// 表示 256个点对，每个点各有横纵坐标 = 256*2*2
// 在定义变量的时候，可以写出中间的计算过程，提高代码的可读性
static int bit_pattern_31_[256*4] = 
{
	8,-3,9,5    /*mean (0), correlation (0)*/,
	4,2,7,-12    /*mean (1.12461e-05), correlation (0.0437584)*/,
	-11,9,-8,2    /*mean (3.37382e-05), correlation (0.0617409)*/,
	7,-12,12,-13    /*mean (5.62303e-05), correlation (0.0636977)*/,
	...
}
```

我们可以看到，这是一个256x4个值组成的数组。256代表描述子的维度，每行的4个值表示一对点的坐标，比如第一行表示$p(x) = (8,-3),p(y)= (9,5)$，根据第2步即可判断这一行对应的二进制是0还是1，这样最终就得到了256维的描述子。
在ORB特征点中对原始的BRIEF进行了改进，利用前面计算的关键点的主方向旋转BRIEF描述子，旋转之后的BRIEF描述子称为Steered BRIEF，这样ORB的描述子就具有了旋转不变性。
如何实现选装不变性呢？
假设现在有一个点$V=(x,y)$，原点指向它的向量$\overrightarrow{OV}$，经过角度为$\theta$的旋转，得到一个新的点$V'=(x',y')$，那么在数学上如何实现呢？
![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/20230928164134.png)
我们来推导。如图7-7所示，假设向量$\overrightarrow{OV}$和水平坐标轴$x$的夹角为$\varphi$，向量模长$||\overrightarrow{OV}  || = r$，则有
$$\begin{align} 
  x = rcos(\varphi) \\
  y = rsin(\varphi)
\end{align}$$
根据三角公式容易得到
$$\begin{aligned}
x^{\prime} & =r \cos (\theta+\varphi)=r \cos (\theta) \cos (\varphi)-r \sin (\theta) \sin (\varphi)=x \cos (\theta)-y \sin (\theta) \\
y^{\prime} &= r \sin (\theta+\varphi)=r \sin (\theta) \cos (\varphi)+r \cos (\theta) \sin (\varphi)=x \sin (\theta)+y \cos (\theta)
\end{aligned}$$
将上式写成矩阵形式，可以得到
$$\begin{bmatrix}
x' \\
y'
\end{bmatrix} = 
\begin{bmatrix}
cos\theta   & -sin\theta\\
sin\theta  & cos\theta
\end{bmatrix}
\begin{bmatrix}
x \\
y
\end{bmatrix}$$
以上就是Steered BRIEF的原理，代码实现如下。
```c++
/**
 * @brief 计算 ORB 特征点的描述子
 * @param[in] kpt       特征点对象
 * @param[in] img       提取特征点的图像
 * @param[in] pattern   预定义好的采样模板
 * @param[out] desc     用作输出变量，保存计算好的描述子，维度为 32x8 bit = 256bit
 */
 static void computeOrbDescriptor(const KeyPoint& kpt, const Mat& img, const Point* pattern, uchar* desc)
{
    // 得到特征点的角度，用弧度制表示。其中 kpt.angle 是角度制，范围为 [0,360) 度
    float angle = (float)kpt.angle*factorPI;
    // 计算这个角度的余弦值和正弦值
    float a = (float)cos(angle), b = (float)sin(angle);
   
    // 获得图像中心指针
    const uchar* center = &img.at<uchar>(cvRound(kpt.pt.y),cvRound(kpt.pt.x));
    // 获得图像每行的字节数
    const int step = (int)img.step;
    
    // 原始的 BRIEF 描述子没有方向不变性，通过加入关键点的方向来计算描述子
    // 称为 Steered BRIEF，具有较好的旋转不变性
    // 具体地，在计算时需要将这里选取的采样模板中点的 x 轴方向旋转到特征点的方向
    // 获得采样点中某个 idx 所对应的点的灰度值，这里旋转前的坐标为 (x,y)
    // 旋转后的坐标为 (x',y')，它们的变换关系：
    // x'= xcos(θ) - ysin(θ)， y'= xsin(θ) + ycos(θ)
    // 下面表示 y'*step + x'
    #define GET_VALUE(idx) center[cvRound(pattern[idx].x*b+
 patten[idx].y*a)*step + cvRound(pattern[idx].x*a-patten[idx].y*b)]
    
    // BRIEF 描述子由 32x8 bit 组成
    // 其中每一位都来自两个像素点灰度的直接比较，所以每比较出 8bit 结果需要 16 个随机点
    // 这也是 pattern 需要 +=16 的原因
    for(int i = 0; i < 32; ++i, pattern += 16)
    {
        int t0,     // 参与比较的第 1 个特征点的灰度值
            t1,     // 参与比较的第 2 个特征点的灰度值
            val;    // 描述子这个字节的比较结果， 0 或 1
        t0 = GET_VALUE(0); t1 = GET_VALUE(1);
        val = t0 < t1;                              // 描述子本字节的 bit0
        t0 = GET_VALUE(2); t1 = GET_VALUE(3);
        val |= (t0 < t1) << 1;                      // 描述子本字节的 bit1
        t0 = GET_VALUE(4); t1 = GET_VALUE(5);
        val |= (t0 < t1) << 2;                      // 描述子本字节的 bit2
        t0 = GET_VALUE(6); t1 = GET_VALUE(7);
        val |= (t0 < t1) << 3;                      // 描述子本字节的 bit3
        t0 = GET_VALUE(8); t1 = GET_VALUE(9);
        val |= (t0 < t1) << 4;                      // 描述子本字节的 bit4
        t0 = GET_VALUE(10); t1 = GET_VALUE(11);
        val |= (t0 < t1) << 5;                      // 描述子本字节的 bit5
        t0 = GET_VALUE(12); t1 = GET_VALUE(13);
        val |= (t0 < t1) << 6;                      // 描述子本字节的 bit6
        t0 = GET_VALUE(14); t1 = GET_VALUE(15);
        val |= (t0 < t1) << 7;                      // 描述子本字节的 bit7

        // 保存当前比较出来的描述子的字节
        desc[i] = (uchar)val;
    }
    
    // 为了避免和程序中的其他部分冲突，在使用完之后就取消这个宏定义
    #undef GET_VALUE
}
```

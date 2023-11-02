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

## ORB特征点均匀化策略
### 为什么需要特征点均匀化
在ORB-SLAM2中，代码中ORB特征点为什么没有直接调用OpenCV的函数呢？
OpenCV的ORB特征提取方法存在一个问题，就是特征点往往集中在纹理丰富的区域，而在缺乏纹理的区域提取到的特征点数量会少很多，这会带来一些问题。
比如会导致一部分特征点是没有用的，本来一个特征点就可以表达清楚的一个小区域，如果在这个区域附近提取了10个特征点，那么其他9个特征点就是冗余的。
还有一个问题就是会影响位姿的解算。特征点在空间中分布的层次越多、越均匀，特征匹配越能精确地表达出空间的几何关系。极端来说，当所有特征点都集中在一个点上时，是无法计算出相机的位姿的。也就是说，特征点分布太过集中会影响SLAM的精度。
因此，ORB-SLAM2采用了特征点均匀化策略来避免特征点过于集中。我们来看在同一张图中ORB-SLAM2的ORB特征点提取结果和OpenCV的ORB特征点提取结果的对比。
![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/20231004163216.png)

从图中可以看到，ORB-SLAM2的特征点均匀化效果非常明显，它是怎么做到的呢？下面来看具体步骤。

1. 根据总的图像金字塔层级数和待提取的特征点总数，计算图像金字塔中每个层级需要提取的特征点数量。
2. 划分格子，在ORB-SLAM2中格子固定尺寸为30像素x30像素。
3. 对每个格子提取FAST角点，如果初始的FAST角点阈值没有检测到角点，则降低FAST角点阈值，这样在弱纹理区域也能提取到更多的角点。如果降低一次阈值后还是提取不到角点，则不在这个格子里提取，这样可以避免提取到质量特别差的角点。
4. 使用四叉树均匀地选取FAST角点，直到达到特征点总数。

### 如何给图像金字塔分配特征点数量
图像金字塔层级数越高，对应层级数的图像分辨率越低，面积（高x宽）越小，所能提取到的特征点数量就越少。所以分配策略是根据图像的面积来定，将总特征点数目根据面积比例均摊到每层图像上。
假设需要提取的特征点数目为$N$，图像金字塔总共有$m$层，第0层图像的宽为$W$，高为$H$，对应的面积$HW=C$，图像金字塔缩放因子为$s$，$0<s<1$。在ORB-SLAM2中，$m=8$，$s=1/1.2$。
那么整个图像金字塔总的图像面积是：
$$
\begin{aligned}
S & =H W\left(s^{2}\right)^{0}+H W\left(s^{2}\right)^{1}+\cdots+H W\left(s^{2}\right)^{(m-1)} \\
& =H W \frac{1-\left(s^{2}\right)^{m}}{1-s^{2}}=C \frac{1-\left(s^{2}\right)^{m}}{1-s^{2}}
\end{aligned}
$$
单位面积应该分配的特征点数量为：
$$
N_{\text {avg }}=\frac{N}{S}=\frac{N}{C \frac{1-\left(s^{2}\right)^{m}}{1-s^{2}}}=\frac{N\left(1-s^{2}\right)}{C\left[1-\left(s^{2}\right)^{m}\right]}
$$
第0层应该分配的特征点数量为
$$N_0 = \frac{N(1-s^2)}{1-(s^2)^m}$$
第$i$层应该分配的特征点数量为
$$  N_ {i}  =  \frac {N(1-s^ {2})}{C[1-(s^ {2})^ {m}]}  C  (s^ {2})^ {i}  =  \frac {N(1-s^ {2})}{1-(s^ {2})^ {m}}   (s^ {2})^ {i}  
 $$
 在ORB-SLAM2的代码中不是按照面积来均摊特征点的，而是按照面积的开方均摊，也就是将式中的$s^2$换成$s$。

```CPP
ORBextractor::ORBextractor(int _nfeatures,		//指定要提取的特征点数目
						   float _scaleFactor,	//指定图像金字塔的缩放系数
						   int _nlevels,		//指定图像金字塔的层数
						   int _iniThFAST,		//指定初始的FAST特征点提取参数，可以提取出最明显的角点
						   int _minThFAST):		//如果初始阈值没有检测到角点，降低到这个阈值提取出弱一点的角点
    nfeatures(_nfeatures), scaleFactor(_scaleFactor), nlevels(_nlevels),
    iniThFAST(_iniThFAST), minThFAST(_minThFAST)//设置这些参数
{
	//存储每层图像缩放系数的vector调整为符合图层数目的大小
    mvScaleFactor.resize(nlevels);  
	//存储这个sigma^2，其实就是每层图像相对初始图像缩放因子的平方
    mvLevelSigma2.resize(nlevels);
	//对于初始图像，这两个参数都是1
    mvScaleFactor[0]=1.0f;
    mvLevelSigma2[0]=1.0f;
	//然后逐层计算图像金字塔中图像相当于初始图像的缩放系数 
    for(int i=1; i<nlevels; i++)  
    {
		//其实就是这样累乘计算得出来的
        mvScaleFactor[i]=mvScaleFactor[i-1]*scaleFactor;
		//原来这里的sigma^2就是每层图像相对于初始图像缩放因子的平方
        mvLevelSigma2[i]=mvScaleFactor[i]*mvScaleFactor[i];
    }

    //接下来的两个向量保存上面的参数的倒数
    mvInvScaleFactor.resize(nlevels);
    mvInvLevelSigma2.resize(nlevels);
    for(int i=0; i<nlevels; i++)
    {
        mvInvScaleFactor[i]=1.0f/mvScaleFactor[i];
        mvInvLevelSigma2[i]=1.0f/mvLevelSigma2[i];
    }

    //调整图像金字塔vector以使得其符合设定的图像层数
    mvImagePyramid.resize(nlevels);

	//每层需要提取出来的特征点个数，这个向量也要根据图像金字塔设定的层数进行调整
    mnFeaturesPerLevel.resize(nlevels);
	
	//图片降采样缩放系数的倒数
    float factor = 1.0f / scaleFactor;
	//第0层图像应该分配的特征点数量
    float nDesiredFeaturesPerScale = nfeatures*(1 - factor)/(1 - (float)pow((double)factor, (double)nlevels));

	//用于在特征点个数分配的，特征点的累计计数清空
    int sumFeatures = 0;
	//开始逐层计算要分配的特征点个数，顶层图像除外（看循环后面）
    for( int level = 0; level < nlevels-1; level++ )
    {
		//分配 cvRound : 返回个参数最接近的整数值
        mnFeaturesPerLevel[level] = cvRound(nDesiredFeaturesPerScale);
		//累计
        sumFeatures += mnFeaturesPerLevel[level];
		//乘系数
        nDesiredFeaturesPerScale *= factor;
    }
    //由于前面的特征点个数取整操作，可能会导致剩余一些特征点个数没有被分配，所以这里就将这个余出来的特征点分配到最高的图层中
    mnFeaturesPerLevel[nlevels-1] = std::max(nfeatures - sumFeatures, 0);

	//成员变量pattern的长度，也就是点的个数，这里的512表示512个点（上面的数组中是存储的坐标所以是256*2*2）
    const int npoints = 512;
	//获取用于计算BRIEF描述子的随机采样点点集头指针
	//注意到pattern0数据类型为Points*,bit_pattern_31_是int[]型，所以这里需要进行强制类型转换
    const Point* pattern0 = (const Point*)bit_pattern_31_;	
	//使用std::back_inserter的目的是可以快覆盖掉这个容器pattern之前的数据
	//其实这里的操作就是，将在全局变量区域的、int格式的随机采样点以cv::point格式复制到当前类对象中的成员变量中
    std::copy(pattern0, pattern0 + npoints, std::back_inserter(pattern));

    //This is for orientation
	//下面的内容是和特征点的旋转计算有关的
    // pre-compute the end of a row in a circular patch
	//预先计算圆形patch中行的结束位置
	//+1中的1表示那个圆的中间行
    umax.resize(HALF_PATCH_SIZE + 1);
	
	//cvFloor返回不大于参数的最大整数值，cvCeil返回不小于参数的最小整数值，cvRound则是四舍五入
    int v,		//循环辅助变量
		v0,		//辅助变量
		vmax = cvFloor(HALF_PATCH_SIZE * sqrt(2.f) / 2 + 1);	//计算圆的最大行号，+1应该是把中间行也给考虑进去了
				//NOTICE 注意这里的最大行号指的是计算的时候的最大行号，此行的和圆的角点在45°圆心角的一边上，之所以这样选择
				//是因为圆周上的对称特性
				
	//这里的二分之根2就是对应那个45°圆心角
    
    int vmin = cvCeil(HALF_PATCH_SIZE * sqrt(2.f) / 2);
	//半径的平方
    const double hp2 = HALF_PATCH_SIZE*HALF_PATCH_SIZE;

	//利用圆的方程计算每行像素的u坐标边界（max）
    for (v = 0; v <= vmax; ++v)
        umax[v] = cvRound(sqrt(hp2 - v * v));		//结果都是大于0的结果，表示x坐标在这一行的边界

    // Make sure we are symmetric
	//这里其实是使用了对称的方式计算上四分之一的圆周上的umax，目的也是为了保持严格的对称（如果按照常规的想法做，由于cvRound就会很容易出现不对称的情况，
	//同时这些随机采样的特征点集也不能够满足旋转之后的采样不变性了）
	for (v = HALF_PATCH_SIZE, v0 = 0; v >= vmin; --v)
    {
        while (umax[v0] == umax[v0 + 1])
            ++v0;
        umax[v] = v0;
        ++v0;
    }
}
```


### 使用四叉树实现特征点均匀化分布

利用四叉树实现特征点均匀化的步骤和原理如下
1. 确定初始的节点（node）数目。根据图像宽高比取整来确定，所以一般的VGA（640像素 x 480像素）分辨率图像刚开始时只有一个节点，也是四叉树的根节点。
   下面用一个例子分析四叉树是如何均匀化选取特定数目的特征点的。
   ![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/20231004204744.png)
   
   假设初始节点只有1个，那么所有的特征点都属于该节点。我们的目标是均匀地选取25个特征点，因此后面就需要分裂出25个节点，然后从每个节点中选取一个有代表性的特征点。
2. 节点第1次分裂，1个根节点分裂为4个节点。如图所示
   ![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/20231004205017.png)
   
   分裂之后根据图像的尺寸划分节点的区域，对应的边界为$UL_ {i}$，$UR_{i}$ ，$BL_ {i}$ ， $BR_ {i},i=1,2,3,4$ 。分别对应左上角、右上角、左下角、右下角的4个坐标。有些坐标会被多个节点共享，比如图像中心点坐标就同时被$BR_ {1}$，$BL_ {2}$ ，$UR_ {3}$ ，$UL_ {4}$ 4个点共享。落在某个节点区域范围内的所有特征点都属于该节点的元素。
3. 对第1次分裂得到的4个节点再次进行分裂，将每一个节点一分为四，得到16个节点，这时有的区域已经没有特征点了，所以在节点的链表中将其删除，图中x的位置，还有的节点整个区域只有一个特征点，下次分裂将不再对该节点进行分裂的操作。最终得到14个节点。
   ![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/20231004205733.png)
   
4. 因为第三步得到的节点数量为14＜25个，需要继续分裂，如果再将上述14个节点进行1分4的操作会得到56个节点，超过了我们预期的25个的数量，源码中对于这一问题的处理是要选取一些特指的节点进行分裂，使得分裂后的节点数量达到25个时，就停止分裂，这样做一方面避免了分裂之后，对多余节点再删除的操作；另一方面，提取避免了无效的指数级的分裂，大大加速了四叉树分裂的过程。
   那如何选取分裂的顺序呢？在源码中采用的是按照节点内的特征点数目进行排序，优先分裂特征点数目多的节点，这样的目的使得特征密集的区域能够分的更加精细。
   ![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/20231004210416.png)
   
5. 在得到25个节点之后，从每个节点中选出角点响应值最高的特征点最为最终的结果，删除其他的点即可。
   ![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/20231004211424.png)
   
```CPP
/**
 * @brief 使用四叉树法对一个图像金字塔图层中的特征点进行平均和分发
 * 
 * @param[in] vToDistributeKeys     等待进行分配到四叉树中的特征点
 * @param[in] minX                  当前图层的图像的边界，坐标都是在“半径扩充图像”坐标系下的坐标
 * @param[in] maxX 
 * @param[in] minY 
 * @param[in] maxY 
 * @param[in] N                     希望提取出的特征点个数
 * @param[in] level                 指定的金字塔图层，并未使用
 * @return vector<cv::KeyPoint>     已经均匀分散好的特征点vector容器
 */
vector<cv::KeyPoint> ORBextractor::DistributeOctTree(const vector<cv::KeyPoint>& vToDistributeKeys, const int &minX,
                                       const int &maxX, const int &minY, const int &maxY, const int &N, const int &level)
{
    // Compute how many initial nodes
    // Step 1 根据宽高比确定初始节点数目
	//计算应该生成的初始节点个数，根节点的数量nIni是根据边界的宽高比值确定的，一般是1或者2
    // ! bug: 如果宽高比小于0.5，nIni=0, 后面hx会报错
    const int nIni = round(static_cast<float>(maxX-minX)/(maxY-minY));

	//一个初始的节点的x方向有多少个像素
    const float hX = static_cast<float>(maxX-minX)/nIni;

	//存储有提取器节点的链表
    list<ExtractorNode> lNodes;

	//存储初始提取器节点指针的vector
    vector<ExtractorNode*> vpIniNodes;

	//重新设置其大小
    vpIniNodes.resize(nIni);

	// Step 2 生成初始提取器节点
    for(int i=0; i<nIni; i++)
    {      
		//生成一个提取器节点
        ExtractorNode ni;

		//设置提取器节点的图像边界
		//注意这里和提取FAST角点区域相同，都是“半径扩充图像”，特征点坐标从0 开始 
        ni.UL = cv::Point2i(hX*static_cast<float>(i),0);    //UpLeft
        ni.UR = cv::Point2i(hX*static_cast<float>(i+1),0);  //UpRight
		ni.BL = cv::Point2i(ni.UL.x,maxY-minY);		        //BottomLeft
        ni.BR = cv::Point2i(ni.UR.x,maxY-minY);             //BottomRight

		//重设vkeys大小
        ni.vKeys.reserve(vToDistributeKeys.size());

		//将刚才生成的提取节点添加到链表中
		//虽然这里的ni是局部变量，但是由于这里的push_back()是拷贝参数的内容到一个新的对象中然后再添加到列表中
		//所以当本函数退出之后这里的内存不会成为“野指针”
        lNodes.push_back(ni);
		//存储这个初始的提取器节点句柄
        vpIniNodes[i] = &lNodes.back();
    }

    //Associate points to childs
    // Step 3 将特征点分配到子提取器节点中
    for(size_t i=0;i<vToDistributeKeys.size();i++)
    {
		//获取这个特征点对象
        const cv::KeyPoint &kp = vToDistributeKeys[i];
		//按特征点的横轴位置，分配给属于那个图像区域的提取器节点（最初的提取器节点）
        vpIniNodes[kp.pt.x/hX]->vKeys.push_back(kp);
    }
    
	// Step 4 遍历此提取器节点列表，标记那些不可再分裂的节点，删除那些没有分配到特征点的节点
    // ? 这个步骤是必要的吗？感觉可以省略，通过判断nIni个数和vKeys.size() 就可以吧
    list<ExtractorNode>::iterator lit = lNodes.begin();
    while(lit!=lNodes.end())
    {
		//如果初始的提取器节点所分配到的特征点个数为1
        if(lit->vKeys.size()==1)
        {
			//那么就标志位置位，表示此节点不可再分
            lit->bNoMore=true;
			//更新迭代器
            lit++;
        }
        ///如果一个提取器节点没有被分配到特征点，那么就从列表中直接删除它
        else if(lit->vKeys.empty())
            //注意，由于是直接删除了它，所以这里的迭代器没有必要更新；否则反而会造成跳过元素的情况
            lit = lNodes.erase(lit);			
        else
			//如果上面的这些情况和当前的特征点提取器节点无关，那么就只是更新迭代器 
            lit++;
    }

    //结束标志位清空
    bool bFinish = false;

	//记录迭代次数，只是记录，并未起到作用
    int iteration = 0;

	//声明一个vector用于存储节点的vSize和句柄对
	//这个变量记录了在一次分裂循环中，那些可以再继续进行分裂的节点中包含的特征点数目和其句柄
    vector<pair<int,ExtractorNode*> > vSizeAndPointerToNode;

	//调整大小，这里的意思是一个初始化节点将“分裂”成为四个
    vSizeAndPointerToNode.reserve(lNodes.size()*4);

    // Step 5 利用四叉树方法对图像进行划分区域，均匀分配特征点
    while(!bFinish)
    {
		//更新迭代次数计数器，只是记录，并未起到作用
        iteration++;

		//保存当前节点个数，prev在这里理解为“保留”比较好
        int prevSize = lNodes.size();

		//重新定位迭代器指向列表头部
        lit = lNodes.begin();

		//需要展开的节点计数，这个一直保持累计，不清零
        int nToExpand = 0;

		//因为是在循环中，前面的循环体中可能污染了这个变量，所以清空
		//这个变量也只是统计了某一个循环中的点
		//这个变量记录了在一次分裂循环中，那些可以再继续进行分裂的节点中包含的特征点数目和其句柄
        vSizeAndPointerToNode.clear();

        // 将目前的子区域进行划分
		//开始遍历列表中所有的提取器节点，并进行分解或者保留
        while(lit!=lNodes.end())
        {
			//如果提取器节点只有一个特征点，
            if(lit->bNoMore)
            {
                // If node only contains one point do not subdivide and continue
				//那么就没有必要再进行细分了
                lit++;
				//跳过当前节点，继续下一个
                continue;
            }
            else
            {
                // If more than one point, subdivide
				//如果当前的提取器节点具有超过一个的特征点，那么就要进行继续分裂
                ExtractorNode n1,n2,n3,n4;

				//再细分成四个子区域
                lit->DivideNode(n1,n2,n3,n4); 

                // Add childs if they contain points
				//如果这里分出来的子区域中有特征点，那么就将这个子区域的节点添加到提取器节点的列表中
				//注意这里的条件是，有特征点即可
                if(n1.vKeys.size()>0)
                {
					//注意这里也是添加到列表前面的
                    lNodes.push_front(n1);   

					//再判断其中子提取器节点中的特征点数目是否大于1
                    if(n1.vKeys.size()>1)
                    {
						//如果有超过一个的特征点，那么待展开的节点计数加1
                        nToExpand++;

						//保存这个特征点数目和节点指针的信息
                        vSizeAndPointerToNode.push_back(make_pair(n1.vKeys.size(),&lNodes.front()));

						//?这个访问用的句柄貌似并没有用到？
                        // lNodes.front().lit 和前面的迭代的lit 不同，只是名字相同而已
                        // lNodes.front().lit是node结构体里的一个指针用来记录节点的位置
                        // 迭代的lit 是while循环里作者命名的遍历的指针名称
                        lNodes.front().lit = lNodes.begin();
                    }
                }
                //后面的操作都是相同的，这里不再赘述
                if(n2.vKeys.size()>0)
                {
                    lNodes.push_front(n2);
                    if(n2.vKeys.size()>1)
                    {
                        nToExpand++;
                        vSizeAndPointerToNode.push_back(make_pair(n2.vKeys.size(),&lNodes.front()));
                        lNodes.front().lit = lNodes.begin();
                    }
                }
                if(n3.vKeys.size()>0)
                {
                    lNodes.push_front(n3);
                    if(n3.vKeys.size()>1)
                    {
                        nToExpand++;
                        vSizeAndPointerToNode.push_back(make_pair(n3.vKeys.size(),&lNodes.front()));
                        lNodes.front().lit = lNodes.begin();
                    }
                }
                if(n4.vKeys.size()>0)
                {
                    lNodes.push_front(n4);
                    if(n4.vKeys.size()>1)
                    {
                        nToExpand++;
                        vSizeAndPointerToNode.push_back(make_pair(n4.vKeys.size(),&lNodes.front()));
                        lNodes.front().lit = lNodes.begin();
                    }
                }

                //当这个母节点expand之后就从列表中删除它了，能够进行分裂操作说明至少有一个子节点的区域中特征点的数量是>1的
                // 分裂方式是后加的节点先分裂，先加的后分裂
                lit=lNodes.erase(lit);

				//继续下一次循环，其实这里加不加这句话的作用都是一样的
                continue;
            }//判断当前遍历到的节点中是否有超过一个的特征点
        }//遍历列表中的所有提取器节点

        // Finish if there are more nodes than required features or all nodes contain just one point
        //停止这个过程的条件有两个，满足其中一个即可：
        //1、当前的节点数已经超过了要求的特征点数
        //2、当前所有的节点中都只包含一个特征点
        //prevSize中保存的是分裂之前的节点个数，如果分裂之前和分裂之后的总节点个数一样，说明当前所有的节点区域中只有一个特征点，已经不能够再细分了
        if((int)lNodes.size()>=N || (int)lNodes.size()==prevSize)	
        {
			//停止标志置位
            bFinish = true;
        }

        // Step 6 当再划分之后所有的Node数大于要求数目时,就慢慢划分直到使其刚刚达到或者超过要求的特征点个数
        //可以展开的子节点个数nToExpand x3，是因为一分四之后，会删除原来的主节点，所以乘以3
        /**
		 * //?BUG 但是我觉得这里有BUG，虽然最终作者也给误打误撞、稀里糊涂地修复了
		 * 注意到，这里的nToExpand变量在前面的执行过程中是一直处于累计状态的，如果因为特征点个数太少，跳过了下面的else-if，又进行了一次上面的遍历
		 * list的操作之后，lNodes.size()增加了，但是nToExpand也增加了，尤其是在很多次操作之后，下面的表达式：
		 * ((int)lNodes.size()+nToExpand*3)>N
		 * 会很快就被满足，但是此时只进行一次对vSizeAndPointerToNode中点进行分裂的操作是肯定不够的；
		 * 理想中，作者下面的for理论上只要执行一次就能满足，不过作者所考虑的“不理想情况”应该是分裂后出现的节点所在区域可能没有特征点，因此将for
		 * 循环放在了一个while循环里面，通过再次进行for循环、再分裂一次解决这个问题。而我所考虑的“不理想情况”则是因为前面的一次对vSizeAndPointerToNode
		 * 中的特征点进行for循环不够，需要将其放在另外一个循环（也就是作者所写的while循环）中不断尝试直到达到退出条件。 
		 * */
        else if(((int)lNodes.size()+nToExpand*3)>N)
        {
			//如果再分裂一次那么数目就要超了，这里想办法尽可能使其刚刚达到或者超过要求的特征点个数时就退出
			//这里的nToExpand和vSizeAndPointerToNode不是一次循环对一次循环的关系，而是前者是累计计数，后者只保存某一个循环的
			//一直循环，直到结束标志位被置位
            while(!bFinish)
            {
				//获取当前的list中的节点个数
                prevSize = lNodes.size();

				//保留那些还可以分裂的节点的信息, 这里是深拷贝
                vector<pair<int,ExtractorNode*> > vPrevSizeAndPointerToNode = vSizeAndPointerToNode;
				//清空
                vSizeAndPointerToNode.clear();

                // 对需要划分的节点进行排序，对pair对的第一个元素进行排序，默认是从小到大排序
				// 优先分裂特征点多的节点，使得特征点密集的区域保留更少的特征点
                //! 注意这里的排序规则非常重要！会导致每次最后产生的特征点都不一样。建议使用 stable_sort
                sort(vPrevSizeAndPointerToNode.begin(),vPrevSizeAndPointerToNode.end());

				//遍历这个存储了pair对的vector，注意是从后往前遍历
                for(int j=vPrevSizeAndPointerToNode.size()-1;j>=0;j--)
                {
                    ExtractorNode n1,n2,n3,n4;
					//对每个需要进行分裂的节点进行分裂
                    vPrevSizeAndPointerToNode[j].second->DivideNode(n1,n2,n3,n4);

                    // Add childs if they contain points
					//其实这里的节点可以说是二级子节点了，执行和前面一样的操作
                    if(n1.vKeys.size()>0)
                    {
                        lNodes.push_front(n1);
                        if(n1.vKeys.size()>1)
                        {
							//因为这里还有对于vSizeAndPointerToNode的操作，所以前面才会备份vSizeAndPointerToNode中的数据
							//为可能的、后续的又一次for循环做准备
                            vSizeAndPointerToNode.push_back(make_pair(n1.vKeys.size(),&lNodes.front()));
                            lNodes.front().lit = lNodes.begin();
                        }
                    }
                    if(n2.vKeys.size()>0)
                    {
                        lNodes.push_front(n2);
                        if(n2.vKeys.size()>1)
                        {
                            vSizeAndPointerToNode.push_back(make_pair(n2.vKeys.size(),&lNodes.front()));
                            lNodes.front().lit = lNodes.begin();
                        }
                    }
                    if(n3.vKeys.size()>0)
                    {
                        lNodes.push_front(n3);
                        if(n3.vKeys.size()>1)
                        {
                            vSizeAndPointerToNode.push_back(make_pair(n3.vKeys.size(),&lNodes.front()));
                            lNodes.front().lit = lNodes.begin();
                        }
                    }
                    if(n4.vKeys.size()>0)
                    {
                        lNodes.push_front(n4);
                        if(n4.vKeys.size()>1)
                        {
                            vSizeAndPointerToNode.push_back(make_pair(n4.vKeys.size(),&lNodes.front()));
                            lNodes.front().lit = lNodes.begin();
                        }
                    }

                    //删除母节点，在这里其实应该是一级子节点
                    lNodes.erase(vPrevSizeAndPointerToNode[j].second->lit);

					//判断是是否超过了需要的特征点数？是的话就退出，不是的话就继续这个分裂过程，直到刚刚达到或者超过要求的特征点个数
					//作者的思想其实就是这样的，再分裂了一次之后判断下一次分裂是否会超过N，如果不是那么就放心大胆地全部进行分裂（因为少了一个判断因此
					//其运算速度会稍微快一些），如果会那么就引导到这里进行最后一次分裂
                    if((int)lNodes.size()>=N)
                        break;
                }//遍历vPrevSizeAndPointerToNode并对其中指定的node进行分裂，直到刚刚达到或者超过要求的特征点个数

                //这里理想中应该是一个for循环就能够达成结束条件了，但是作者想的可能是，有些子节点所在的区域会没有特征点，因此很有可能一次for循环之后
				//的数目还是不能够满足要求，所以还是需要判断结束条件并且再来一次
                //判断是否达到了停止条件
                if((int)lNodes.size()>=N || (int)lNodes.size()==prevSize)
                    bFinish = true;				
            }//一直进行nToExpand累加的节点分裂过程，直到分裂后的nodes数目刚刚达到或者超过要求的特征点数目
        }//当本次分裂后达不到结束条件但是再进行一次完整的分裂之后就可以达到结束条件时
    }// 根据兴趣点分布,利用4叉树方法对图像进行划分区域

    // Retain the best point in each node
    // Step 7 保留每个区域响应值最大的一个兴趣点
    //使用这个vector来存储我们感兴趣的特征点的过滤结果
    vector<cv::KeyPoint> vResultKeys;

	//调整容器大小为要提取的特征点数目
    vResultKeys.reserve(nfeatures);

	//遍历这个节点链表
    for(list<ExtractorNode>::iterator lit=lNodes.begin(); lit!=lNodes.end(); lit++)
    {
		//得到这个节点区域中的特征点容器句柄
        vector<cv::KeyPoint> &vNodeKeys = lit->vKeys;

		//得到指向第一个特征点的指针，后面作为最大响应值对应的关键点
        cv::KeyPoint* pKP = &vNodeKeys[0];

		//用第1个关键点响应值初始化最大响应值
        float maxResponse = pKP->response;

		//开始遍历这个节点区域中的特征点容器中的特征点，注意是从1开始哟，0已经用过了
        for(size_t k=1;k<vNodeKeys.size();k++)
        {
			//更新最大响应值
            if(vNodeKeys[k].response>maxResponse)
            {
				//更新pKP指向具有最大响应值的keypoints
                pKP = &vNodeKeys[k];
                maxResponse = vNodeKeys[k].response;
            }
        }

        //将这个节点区域中的响应值最大的特征点加入最终结果容器
        vResultKeys.push_back(*pKP);
    }

    //返回最终结果容器，其中保存有分裂出来的区域中，我们最感兴趣、响应值最大的特征点
    return vResultKeys;
}
```


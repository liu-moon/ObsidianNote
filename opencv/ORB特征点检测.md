本文介绍如何通过opencv提取ORB的特征点

首先编写CMakeLists文件，引入OpenCV

```cpp
cmake_minimum_required(VERSION 3.10.2)
project(liuiu)

set(CMAKE_CXX_STANDARD 14)

find_package(OpenCV REQUIRED)
# find_package(Glog REQUIRED)

include_directories(${OpenCV_INCLUDE_DIRS})
# include_directories(${GLOG_INCLUDE_DIRS})

add_executable(liuiu main.cpp)

target_link_libraries(liuiu ${OpenCV_LIBS})
# target_link_libraries(liuiu glog::glog)
```

然后编写代码如下

```cpp
#include <iostream>
#include <fstream>
#include <opencv2/core/core.hpp>
#include <opencv2/features2d/features2d.hpp>
#include <opencv2/highgui/highgui.hpp>

using namespace std;
using namespace cv;

int main(int argc, char **argv)
{

    CommandLineParser parser(argc, argv, "{@input | ./../your_image.png | input image}");

    //-- 读取图像
    Mat img = imread(parser.get<String>("@input"));

    //-- 初始化
    // 定义关键点向量
    std::vector<KeyPoint> keypoints;
    // 定义描述子矩阵
    Mat descriptors;
    // 定义ORB特征点检测器和描述子提取器
    Ptr<FeatureDetector> detector = ORB::create(600);
    Ptr<DescriptorExtractor> descriptor = ORB::create(600);

    //-- 第一步:检测 Oriented FAST 角点位置
    detector->detect(img, keypoints);

    //-- 第二步:根据角点位置计算 BRIEF 描述子
    descriptor->compute(img, keypoints, descriptors);

    //-- 第三步:绘制特征点
    Mat outimg;
    drawKeypoints(img, keypoints, outimg, Scalar::all(-1), DrawMatchesFlags::DEFAULT);
    imshow("ORB特征点", outimg);

    cout << "pt = " << keypoints[0].pt << endl;
    cout << "angle = " << keypoints[0].angle << endl;
    cout << "size = " << keypoints[0].size << endl;
    cout << "response = " << keypoints[0].response << endl;
    cout << "octave = " << keypoints[0].octave << endl;
    cout << "class_id = " << keypoints[0].class_id << endl;

    waitKey(0);

    // 将cout的内容写入文件
    ofstream outputFile("output.txt");
    if (outputFile.is_open())
    {
        outputFile << "Number of keypoints: " << keypoints.size() << endl;
        outputFile << "Size of descriptors: " << descriptors.size() << endl;
        outputFile << "Descriptors: " << endl
                   << descriptors << endl;
        outputFile.close();
        cout << "Output written to output.txt" << endl;
    }
    else
    {
        cerr << "Unable to open the output file." << endl;
        return -1;
    }

    return 0;
}
```

这里主要介绍以下ORB::create构造函数：

```cpp
static Ptr<ORB> cv::ORB::create	(int 	nfeatures = 500,
								 float 	scaleFactor = 1.2f,
								 int 	nlevels = 8,
								 int 	edgeThreshold = 31,
								 int 	firstLevel = 0,
								 int 	WTA_K = 2,
								 int 	scoreType = ORB::HARRIS_SCORE,
								 int 	patchSize = 31,
								 int 	fastThreshold = 20 
								 )
```

| 参数 | 含义 |
| --- | --- |
| nfeatures | 要保留的最大的特征点的个数 |
| scaleFactor | 金字塔降采样比率，大于1。scaleFactor\==2 表示经典金字塔，其中每个下一级别的像素数量比前一级别少4倍，但如此大的尺度因子会大幅降低特征匹配分数。另一方面，尺度因子接近1会意味着为了涵盖特定的尺度范围，您将需要更多的金字塔级别，因此速度将受到影响。 |
| nlevels | 金字塔层数。最小级别的线性大小等于 input_image_linear_size/pow(scaleFactor, nlevels - firstLevel) |
| edgeThreshold | 这是未检测到特征的边界的大小。它应该大致匹配 patchSize 参数。 |
| firstLevel | 将源图像放入的金字塔级别。前面的图层充满了放大的源图像。 |
| WTA_K | 产生定向BRIEF描述符的每个元素的点的数量。默认值 2 表示我们采用随机点对并比较它们的亮度的 Brief，因此我们得到 0/1 响应。其他可能的值是 3 和 4。例如，3 表示我们随机取 3 个点（当然，这些点坐标是随机的，但它们是从预定义的种子生成的，因此，BRIEF 描述符的每个元素都是从像素矩形），找到最大亮度点和获胜者的输出索引（0、1或2）。这样的输出将占用 2 位，因此需要汉明距离的特殊变体，表示为 NORM_HAMMING2（每个 bin 2 位）。当 WTA_K=4 时，我们采用 4 个随机点来计算每个 bin（这也将占用 2 位，可能值为 0、1、2 或 3）。 |
| scoreType | 默认的HARRIS_SCORE表示使用Harris算法对特征进行排序（分数写入KeyPoint::score，用于保留最好的nfeatures特征）； FAST_SCORE 是参数的替代值，它产生的关键点稳定性稍差，但计算速度稍快一些。 |
| patchSize | 用于定向 BRIEF 描述符的图像块的大小。当然，在较小的金字塔层上，由特征覆盖的感知图像区域将更大。 |

以下是官方文档：

[OpenCV: cv::ORB Class Reference](https://docs.opencv.org/3.4/db/d95/classcv_1_1ORB.html)
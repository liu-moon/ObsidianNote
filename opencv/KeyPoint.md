详细说明：


## 公共成员函数
KeyPoint()
```c++
cv::KeyPoint::KeyPoint()
```
默认构造函数
KeyPoint()
```CPP
cv::KeyPoint::KeyPoint (Point2f 	pt,
					   float  		size,
					   float 	 	angle = -1,
					   float 	 	response = 0,
					   int 	 	 	octave = 0,
					   int 	 	 	class_id = -1 
					   )	
```

| 参数     | 含义                                  |
| -------- | ------------------------------------- |
| pt       | 关键点的x和y坐标                      |
| size     | 关键点的直径                          |
| angle    | 关键点的方向（默认值-1）              |
| response | 关键点的强度（默认值0）               |
| octave   | 关键点所在的图像金字塔层级（默认值0） |
| class_id | 物体id（默认值-1）                    | 

我们可以用class_id对每个特征点进行区分，未设定时为-1，需要靠自己设定

默认构造函数
```CPP
cv::KeyPoint::KeyPoint (float 	x,
						float 	y,
						float 	size,
						float 	angle = -1,
						float 	response = 0,
						int 	octave = 0,
						int 	class_id = -1 
						)
```

| 参数     | 含义                                  |
| -------- | ------------------------------------- |
| x        | 关键点的x坐标                         |
| y        | 关键点的y坐标                         |
| size     | 关键点的直径                          |
| angle    | 关键点的方向（默认值-1）              |
| response | 关键点的强度（默认值0）               |
| octave   | 关键点所在的图像金字塔层级（默认值0） |
| class_id | 物体id（默认值-1）                    | 

我们可以用class_id对每个特征点进行区分，未设定时为-1，需要靠自己设定

## 静态公共成员函数

```CPP
static void cv::KeyPoint::convert(const std::vector< KeyPoint > & 	keypoints,
								  std::vector< Point2f > & 	points2f,
								  const std::vector< int > & 	keypointIndexes = std::vector< int >() 
								  )	
```

这个方法将关键点的向量转换为点的向量，或者反过来，其中每个关键点都被赋予相同的大小和相同的方向。

| 参数            | 含义                                                               |
| --------------- | ------------------------------------------------------------------ |
| keypoints       | 从任何特征检测算法（如 SIFT/SURF/ORB）获得的关键点                 |
| points2f        | 每个关键点的 (x,y) 坐标数组                                        |
| keypointIndexes | 要转换为点的关键点索引数组。（类似于一个掩码，只转换指定的关键点） | 


```CPP

static void cv::KeyPoint::convert	(	const std::vector< Point2f > & 	points2f,
									 std::vector< KeyPoint > & 	keypoints,
									 float 	size = 1,
									 float 	response = 1,
									 int 	octave = 0,
									 int 	class_id = -1 
									 )	
```

| 参数      | 含义                                               |
| --------- | -------------------------------------------------- |
| points2f  | 每个关键点的 (x,y) 坐标数组                        |
| keypoints | 从任何特征检测算法（如 SIFT/SURF/ORB）获得的关键点 |
| size      | 关键点直径                                         |
| response  | 关键点检测器对关键点的响应（即关键点的强度）       |
| octave    | 检测到关键点的金字塔层级                           |
| class_id  | 物体id                                             | 

```cpp
static float cv::KeyPoint::overlap	(	const KeyPoint & 	kp1,
									 const KeyPoint & 	kp2 
									 )	
```

此方法计算关键点对的重叠。 重叠度是关键点区域交集面积与关键点区域并集面积之间的比率（将关键点区域视为圆）。 如果它们不重叠，我们就得到零。 如果它们在相同位置且大小相同，我们得到 1。

| 参数 | 含义         |
| ---- | ------------ |
| kp1  | 第一个关键点 |
| kp2  | 第二个关键点 | 



## 为什么要用齐次坐标
- 定义：齐次坐标就是在原有坐标的基础上加上一个维度，比如
		$[x,y]^T \to [x,y,1]^T$
		$[x,y,z]^T \to [x,y,z,1]^T$
### 能够非常方便地表达点在直线或平面上
- 直线方程的表达式
		$ax+by+c=0$
		向量表示：$l=[a,b,c]^T$
		二维点：$p=[x,y]^T$
		齐次坐标：$\tilde{p} = [x,y,1]^T$
		点在直线上：$ax+by+c=[x,y,1][a,b,c]^T=\tilde{p}^Tl=0$
- 平面$\pi$方程的表达式
		$ax+by+cz+d=0$
		向量表示：$\pi=[a,b,c,d]^T$
		三维点：$P=[x,y,z]^T$
		齐次坐标：$\tilde{P} = [x,y,z,1]^T$
		点在平面上：$ax+by+cz+d=[x,y,z,1][a,b,c,1]^T=\tilde{P}^T\pi=0$
### 方便表达直线之间的交点和平面之间的交线
- 已知两点坐标，表示出通过这两点的直线
- 已知两条直线的坐标，表示出两条直线的交点
	推导过程如下：
	![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/orbslam.png)
### 能够表达无穷远
- 结论：如果一个点的齐次坐标中，最后一个元素为0，则表示该点为无穷远点。
- 两条平行的直线
		$ax+by+c=0$
		$ax+by+d=0$
		向量表示：
		$m=[a,b,c]^T$
		$n=[a,b,d]^T$
		交点的齐次坐标$m \times n$
		$m \times n = [bd-bc,ac-ad,0]$
		表示交点是无穷远点
### 更简洁地表达空间变换
方便地表达空间中的旋转、平移和缩放。
- 向量$p$
- 旋转矩阵$R$
- 平移向量$t$
- 非齐次坐标表达变换
		${p}' = Rp+t$
- 齐次坐标表达变换
		${\tilde{p} }'  = \begin{bmatrix} {p}'\\1\end{bmatrix} = \begin{bmatrix} R & t \\  O  & 1\end{bmatrix} \begin{bmatrix} {p}\\1\end{bmatrix}=T \begin{bmatrix} {p}\\1\end{bmatrix}= T{\tilde{p} }$
- 变换矩阵$T$


## 三维空间中刚体旋转的几种表示方法
### 旋转矩阵
3x3的矩阵
- 有较强的约束条件：正交性
- 与本身的转置矩阵的乘积是单位矩阵，行列式值为1
- 可逆矩阵，逆矩阵（转置矩阵）表示反方向的旋转
- 用9个元素表示3个自由度的旋转，表达冗余
### 四元数
一个实部和三个虚部组成
- 紧凑、没有奇异的表达方式
- 注意：必须是单位四元数才能描述旋转，记得在使用四元数之前必须要**归一化**
- 在编程时，Eigen库中，四元数的构造及初始化的实部、虚部的顺序和内部系数的存储顺序不同
		```c++
		// 四元数构造及初始化时的顺序是 w,x,y,z 而内部系数 coeffs 的存储顺序是 x,y,z,w
		template<typename _Scalar, int _Options>
		Eigen::Quaternion< _Scalar, _Options>::Quaternion(const Scalar & w,
			const Scalar & x, const Scalar & y, const Scalar & z
		)
		```
### 旋转向量
用一个旋转轴$n$和旋转角$\theta$描述一个旋转。
- 因为旋转角度有周期性，这种表达方式具有奇异性
旋转向量->旋转矩阵
- 罗德里格斯公式
### 欧拉角
- 直观
- 有万向锁问题

### 矩阵线性代数运算库Eigen
- 安装
		```shell
		sudo apt-get install libeigen3-dev
		```
- 该库只有头文件，没有.so和.a的二进制库文件，在CMakeLists中只需要添加头文件路径即可
- 以矩阵为基本数据单元，所有矩阵和向量都是Matrix模板类的对象
	- 使用时一般需要指定3个参数
		- 数据类型
		- 行数
		- 列数
- 对于已知大小的矩阵，要直接指定矩阵的大小和类型，以便提高效率
- 如果不确定矩阵的大小，可以使用动态矩阵`Eigen::Dynamic`
		```c++
		// 动态矩阵
		Eigen::Matrix<double,Eigen::Dynamic,Eigen::Dynamic> matrix_dynamic;
		```
- Eigen中操作的数据类型必须完全一致，否则无法运算
- Eigen中还提供了矩阵分解、稀疏线性方程求解等函数
- Eigen中几种旋转表达方式的相互转换
		具体转换如下：
		![](https://cdn.jsdelivr.net/gh/liu-moon/pic@main/img/20230911181111.png)
具体的代码如下：
```c++
/*
 * 目标：已知旋转向量定义为沿着 z 轴旋转 45°
 * 下面按照该定义用 Eigen 实现旋转向量、旋转矩阵和四元数及其之间的相互转换
 * */

#include <iostream>
#include <cmath>
#include <Eigen/Core>
#include <Eigen/Geometry>
using namespace std;

int main() {
  // 旋转向量（轴角）：沿着 z 轴旋转 45°
  Eigen::AngleAxisd rotation_vector(M_PI / 4, Eigen::Vector3d(0, 0, 1));
  cout << "旋转向量的旋转轴 = \n" << rotation_vector.axis() << "\n 旋转向量角度 = " << rotation_vector.angle() << endl;

  // 旋转矩阵：沿着 z 轴旋转 45°
  Eigen::Matrix3d rotation_matrix = Eigen::Matrix3d::Identity();
  rotation_matrix << 0.707, -0.707, 0,
      0.707, 0.707, 0,
      0, 0, 1;
  cout << "旋转矩阵 = \n" << rotation_matrix << endl;

  // 四元数：沿着 z 轴旋转 45°
  Eigen::Quaterniond quat = Eigen::Quaterniond(0, 0, 0.383, 0.924);
  cout << "四元数的输出方法 1：四元数 = \n" << quat.coeffs() << endl;
  // 请注意coeffs的顺序是(x,y,z,w),w为实部，前三者为虚部
  cout << "四元数的输出方法 2：四元数 = \n" << quat.x() << " " << quat.y() << " " << quat.z() << " " << quat.w() << endl;

  // 1. 将旋转矩阵转化为其他形式
  rotation_vector.fromRotationMatrix(rotation_matrix);
  cout << "旋转矩阵转换为旋转向量方法 1：旋转轴 = \n" << rotation_vector.axis() << "\n 旋转角度 = "
       << rotation_vector.angle() << endl;
  // 注意：fromRotationMatrix 参数只适用于将旋转矩阵转换为旋转向量
  // 不适用于将旋转矩阵转换为四元数，因为四元数的旋转轴不唯一

  rotation_vector = rotation_matrix;
  cout << "旋转矩阵转换为旋转向量方法 2：旋转轴 = \n" << rotation_vector.axis() << "\n 旋转角度 = "
       << rotation_vector.angle() << endl;

  rotation_vector = Eigen::AngleAxisd(rotation_matrix);
  cout << "旋转矩阵转换为旋转向量方法 3：旋转轴 = \n" << rotation_vector.axis() << "\n 旋转角度 = "
       << rotation_vector.angle() << endl;

  quat = Eigen::Quaterniond(rotation_matrix);
  cout << "旋转矩阵转换为四元数方法 1：Q =\n" << quat.coeffs() << endl;

  quat = rotation_matrix;
  cout << "旋转矩阵转换为四元数方法 2：Q =\n" << quat.coeffs() << endl;

  // 2. 将旋转向量转化为其他形式
  cout << "旋转向量转换为旋转矩阵方法 1：旋转矩阵 R=\n" << rotation_vector.matrix() << endl;
  cout << "旋转向量转换为旋转矩阵方法 2：旋转矩阵 R=\n" << rotation_vector.toRotationMatrix() << endl;
  quat = Eigen::Quaterniond(rotation_vector);
  // 请注意 coeffs 的顺序是 (x,y,z,w),w 为实部，前三者为虚部 （这里书里写错了）
  cout << "旋转向量转换为四元数： Q=\n" << quat.coeffs() << endl;

  // 3. 将四元数转换为其他形式
  rotation_vector = quat;
  cout << "四元数转换为旋转向量：旋转轴 = \n" << rotation_vector.axis() << "\n 旋转角度 = "
       << rotation_vector.angle() << endl;

  rotation_matrix = quat.matrix();
  cout << "四元数转换为旋转矩阵方法 1：旋转矩阵 =\n" << rotation_matrix << endl;

  rotation_matrix = quat.toRotationMatrix();
  cout << "四元数转换为旋转矩阵方法 2：旋转矩阵 =\n" << rotation_matrix << endl;

  return 0;
}
```



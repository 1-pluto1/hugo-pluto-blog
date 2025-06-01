---
title: PnP问题
date: 2023-09-15T11:30:03+00:00
tags:
  - 3DV
categories:
  - Research
author: ZhaoYang
showToc: true
TocOpen: true
draft: false
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---
## 任务：学习后方交会(resection)原理，阅读 colmap 代码，回答问题

#### 一.后方交会(PnP)的目的是什么？

后方交会（PnP，Perspective-n-Point）的目的是确定相机的位置和方向（姿态）以及物体在相机坐标系下的三维坐标。具体而言，PnP问题的目标是从已知的物体特征点在图像中的投影位置（2D坐标）和这些特征点在物体坐标系下的三维坐标（3D坐标），计算出相机的旋转和平移参数，以及物体的三维姿态。因此，PnP问题的主要目标是从图像中的二维信息还原出相机和物体之间的三维空间关系。

**通俗的讲**，PnP问题就是在已知世界坐标系下N个空间点的真实坐标以及这些空间点在图像上的投影，如何计算相机所在的位姿。其中，已知量是空间点的真实坐标和图像坐标，未知量（求解量）是相机的位姿。

![image-20231120125945905.png](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/image-20231120125945905.png)






#### 二.**影响 PnP 的精度因素？colmap 选择视图的方法？结合代码分析**

##### （1）影响PnP的精度因素

**特征点的数量和准确性：** 使用更多的特征点可以提高PnP解的准确性。然而，如果特征点数量太少，解可能变得不稳定。从图像中提取的特征点的准确性直接影响PnP解的准确性。如果特征点提取不准确，那么计算的相机位置和方向也会不准确。窗体顶端

**噪声：** 图像中的噪声会导致特征点提取的不确定性，从而影响PnP的准确性。在图像处理之前，可以使用滤波器等技术来减少噪声的影响。

**算法选择：** 不同的PnP算法有不同的性能特点。某些算法可能对噪声更敏感，而其他算法可能在处理大量特征点时更为高效。选择适合特定情境的算法是重要的。

**相机参数的准确性：** PnP解依赖于相机的内部参数，如焦距、主点位置等，以及可能存在的径向和切向畸变参数。这些参数的准确性对PnP解的精度至关重要

**姿态变化：** 如果相机的姿态变化较大，即相机从一帧到另一帧的位置和方向发生显著变化，PnP问题可能会变得不稳定，解可能不唯一或不准确。

**初始估计：** 初始相机姿态的准确性对PnP解的性能也有影响。一些PnP算法可能对初始估计敏感，因此提供更好的初始估计可能会改善解的准确性。

**图像分辨率：** 图像的分辨率越高，可以提供更多的细节，有助于提高特征点的准确性。高分辨率图像可以提高PnP解的稳定性。

##### (2)colmap选择视图的方法

view的选择不但会影响pose估计的质量还会影响后续的三角化的精度和完整性

常规方案：

###### （1）方案一

只按照可视点的数量，即选择可视点数量最多的视图

###### （2）方案二

按照三角化点数与可视点数比值的大小，即选择三角化点数与可视点数比值最大的视图

colmap独有的方案：

###### （3）方案三

colmap提出了一种打分机制筛选：

宗旨：view视图中的可视点越多、分布越均匀，打分越高。

方法：打分金字塔，分三级网格，打分基数为2、2^2、2^3.

e.g.图score=66 = 2 * 1 + 22 * 4+23 * 6，即

一级网格中所有点只在1个格中

二级网格所有点分布在4个网格

三级网格中分布在6个网格
![image-20231120130331145.png](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/image-20231120130331145.png)


##### (3)结合代码分析



```c++
//代码路径：src/colmap/sfm/incremental_mapper.cc：
float RankNextImageMaxVisiblePointsNum(const Image& image) {
  return static_cast<float>(image.NumVisiblePoints3D());
}

float RankNextImageMaxVisiblePointsRatio(const Image& image) {
  return static_cast<float>(image.NumVisiblePoints3D()) /
         static_cast<float>(image.NumObservations());
}

float RankNextImageMinUncertainty(const Image& image) {
  return static_cast<float>(image.Point3DVisibilityScore());
}

//代码路径：src/colmap/sfm/incremental_mapper.cc：
  std::function<float(const Image&)> rank_image_func;
  switch (options.image_selection_method) {
    case Options::ImageSelectionMethod::MAX_VISIBLE_POINTS_NUM:
      rank_image_func = RankNextImageMaxVisiblePointsNum;
      break;
    case Options::ImageSelectionMethod::MAX_VISIBLE_POINTS_RATIO:
      rank_image_func = RankNextImageMaxVisiblePointsRatio;
      break;
    case Options::ImageSelectionMethod::MIN_UNCERTAINTY:
      rank_image_func = RankNextImageMinUncertainty;
      break;
  }

//代码路径：src/colmap/scene/image.h
//237
point2D_t Image::NumObservations() const { return num_observations_; }
//249
point2D_t Image::NumVisiblePoints3D() const { return num_visible_points3D_; }
//251
size_t Image::Point3DVisibilityScore() const {
  return point3D_visibility_pyramid_.Score();
}
```



#### 三.colmap P3P 算法

 ##### （一）学习给出的高小山算法的论文，理清其原理。

###### （1）算法理解		

PnP问题是在已知n 个 3D 空间点以及它们的投影位置时估计相机所在的位姿。那么 n 最小为多少时我们才能进行估算呢（最少需要几个3D-2D点对）？

我们可以设想以下场景，设相机位于点OC，P1、P2、P3……为特征点。

![image-20231120171247219.png](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/image-20231120171247219.png)


###### （2）算法数学推导



数学推导的一些变量设定，采用余弦定理得到如下方程：

![image-20231120171517847.png](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/image-20231120171517847.png)




![image-20231120171534997.png](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/image-20231120171534997.png)


作者设定的"reality conditions"，如果满足条件就称解集为"physical solutions"(物理解):


![image-20231120171604265.png](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/image-20231120171604265.png)




进行化简，通过变量的代换来将 （1）式化为（3）式，（3）式通过消元法转化为（4），（4）与（1）具有相同数量的物理解。现在，P3P问题被简化为寻找两个二次方程的正解。因此，我们得到了以下结果：P3P问题要么有无穷多个解，要么最多有四个物理解。

![image-20231120171947581.png](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/image-20231120171947581.png)




然后使用Wu-Ritt’s zero decomposition method来求解方程，得到X、Y、Z的值，P3P算法的总体计算流程如下：
![image-20231120195028189.png](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/image-20231120195028189.png)




得到X、Y、Z的值后，利用三角形相似的原理，得到A、B、C三点在相机坐标系下的坐标，过程如下：

![image-20231120195051211.png](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/image-20231120195051211.png)




得到A、B、C三点在世界坐标系和相机坐标系的坐标后就可以通过第四个点最小化重投影误差取得最合适的解，然后通过ICP方法就可以得到相机的位姿估计。







另外，对于夹角余弦值的求解如下：
https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/image-20231120195051211.png





P3P算法的过程和优缺点如下：

![image-20231120195130671.png](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/image-20231120195130671.png)


##### （二）colmap P3P 代码分析。结合论文，将其算法流程分析清楚。

```c++
#include "colmap/estimators/absolute_pose.h"

#include "colmap/estimators/utils.h"
#include "colmap/math/polynomial.h"
#include "colmap/util/logging.h"

#include <Eigen/Geometry>

namespace colmap {

std::vector<P3PEstimator::M_t> P3PEstimator::Estimate(
    const std::vector<X_t>& points2D, const std::vector<Y_t>& points3D) {
  CHECK_EQ(points2D.size(), 3);//检验向量大小是否为3
  CHECK_EQ(points3D.size(), 3);

  Eigen::Matrix3d points3D_world;
  points3D_world.col(0) = points3D[0];//将三维向量赋值到points3D_world的每一列
  points3D_world.col(1) = points3D[1];
  points3D_world.col(2) = points3D[2];
 ///取二维点，将其转换为齐次坐标表示，并将结果标准化为单位向量，最终存储在u中
  const Eigen::Vector3d u = points2D[0].homogeneous().normalized();									   
  const Eigen::Vector3d v = points2D[1].homogeneous().normalized();
  const Eigen::Vector3d w = points2D[2].homogeneous().normalized();

  // Angles between 2D points.
  const double cos_uv = u.transpose() * v;
  const double cos_uw = u.transpose() * w;
  const double cos_vw = v.transpose() * w;

  // Distances between 2D points.
  const double dist_AB_2 = (points3D[0] - points3D[1]).squaredNorm();
  const double dist_AC_2 = (points3D[0] - points3D[2]).squaredNorm();
  const double dist_BC_2 = (points3D[1] - points3D[2]).squaredNorm();

  const double dist_AB = std::sqrt(dist_AB_2);
//见论文（3）处
  const double a = dist_BC_2 / dist_AB_2;
  const double b = dist_AC_2 / dist_AB_2;

  // Helper variables for calculation of coefficients.
  const double a2 = a * a;
  const double b2 = b * b;
  const double p = 2 * cos_vw;
  const double q = 2 * cos_uw;
  const double r = 2 * cos_uv;
  const double p2 = p * p;
  const double p3 = p2 * p;
  const double q2 = q * q;
  const double r2 = r * r;
  const double r3 = r2 * r;
  const double r4 = r3 * r;
  const double r5 = r4 * r;

  // Build polynomial coefficients: a4*x^4 + a3*x^3 + a2*x^2 + a1*x + a0 = 0.
  Eigen::Matrix<double, 5, 1> coeffs;
  coeffs(0) = -2 * b + b2 + a2 + 1 + a * b * (2 - r2) - 2 * a;
  coeffs(1) = -2 * q * a2 - r * p * b2 + 4 * q * a + (2 * q + p * r) * b +
              (r2 * q - 2 * q + r * p) * a * b - 2 * q;
  coeffs(2) = (2 + q2) * a2 + (p2 + r2 - 2) * b2 - (4 + 2 * q2) * a -
              (p * q * r + p2) * b - (p * q * r + r2) * a * b + q2 + 2;
  coeffs(3) = -2 * q * a2 - r * p * b2 + 4 * q * a +
              (p * r + q * p2 - 2 * q) * b + (r * p + 2 * q) * a * b - 2 * q;
  coeffs(4) = a2 + b2 - 2 * a + (2 - p2) * b - 2 * a * b + 1;

  Eigen::VectorXd roots_real;
  Eigen::VectorXd roots_imag;
  if (!FindPolynomialRootsCompanionMatrix(coeffs, &roots_real, &roots_imag)) {
    return std::vector<P3PEstimator::M_t>({});
  }

  std::vector<M_t> models;
  models.reserve(roots_real.size());

  for (Eigen::VectorXd::Index i = 0; i < roots_real.size(); ++i) {
    const double kMaxRootImag = 1e-10;
    if (std::abs(roots_imag(i)) > kMaxRootImag) {
      continue;
    }

    const double x = roots_real(i);
    if (x < 0) {
      continue;
    }

    const double x2 = x * x;
    const double x3 = x2 * x;

    // Build polynomial coefficients: b1*y + b0 = 0.
    const double bb1 =
        (p2 - p * q * r + r2) * a + (p2 - r2) * b - p2 + p * q * r - r2;
    const double b1 = b * bb1 * bb1;
    const double b0 =
        ((1 - a - b) * x2 + (a - 1) * q * x - a + b + 1) *
        (r3 * (a2 + b2 - 2 * a - 2 * b + (2 - r2) * a * b + 1) * x3 +
         r2 *
             (p + p * a2 - 2 * r * q * a * b + 2 * r * q * b - 2 * r * q -
              2 * p * a - 2 * p * b + p * r2 * b + 4 * r * q * a +
              q * r3 * a * b - 2 * r * q * a2 + 2 * p * a * b + p * b2 -
              r2 * p * b2) *
             x2 +
         (r5 * (b2 - a * b) - r4 * p * q * b +
          r3 * (q2 - 4 * a - 2 * q2 * a + q2 * a2 + 2 * a2 - 2 * b2 + 2) +
          r2 * (4 * p * q * a - 2 * p * q * a * b + 2 * p * q * b - 2 * p * q -
                2 * p * q * a2) +
          r * (p2 * b2 - 2 * p2 * b + 2 * p2 * a * b - 2 * p2 * a + p2 +
               p2 * a2)) *
             x +
         (2 * p * r2 - 2 * r3 * q + p3 - 2 * p2 * q * r + p * q2 * r2) * a2 +
         (p3 - 2 * p * r2) * b2 +
         (4 * q * r3 - 4 * p * r2 - 2 * p3 + 4 * p2 * q * r - 2 * p * q2 * r2) *
             a +
         (-2 * q * r3 + p * r4 + 2 * p2 * q * r - 2 * p3) * b +
         (2 * p3 + 2 * q * r3 - 2 * p2 * q * r) * a * b + p * q2 * r2 -
         2 * p2 * q * r + 2 * p * r2 + p3 - 2 * r3 * q);

    // Solve for y.
    const double y = b0 / b1;
    const double y2 = y * y;

    const double nu = x2 + y2 - 2 * x * y * cos_uv;

    const double dist_PC = dist_AB / std::sqrt(nu);
    const double dist_PB = y * dist_PC;
    const double dist_PA = x * dist_PC;

    Eigen::Matrix3d points3D_camera;
    points3D_camera.col(0) = u * dist_PA;  // A'
    points3D_camera.col(1) = v * dist_PB;  // B'
    points3D_camera.col(2) = w * dist_PC;  // C'

    // Find transformation from the world to the camera system.
    const Eigen::Matrix4d transform =
        Eigen::umeyama(points3D_world, points3D_camera, false);
    models.push_back(transform.topLeftCorner<3, 4>());
  }

  return models;
}
//函数的作用是计算投影矩阵对应的三维点到二维点的重投影误差的平方，并将结果存储在 residuals 向量中。
void P3PEstimator::Residuals(const std::vector<X_t>& points2D,
                             const std::vector<Y_t>& points3D,
                             const M_t& proj_matrix,
                             std::vector<double>* residuals) {
  ComputeSquaredReprojectionError(points2D, points3D, proj_matrix, residuals);
}
```

求解多项式的解的代码

```c++
//求解多项式的解的代码
#include "colmap/math/polynomial.h"

#include "colmap/util/logging.h"

#include <Eigen/Eigenvalues>

namespace colmap {
namespace {

// Remove leading zero coefficients.
Eigen::VectorXd RemoveLeadingZeros(const Eigen::VectorXd& coeffs) {
  Eigen::VectorXd::Index num_zeros = 0;
  for (; num_zeros < coeffs.size(); ++num_zeros) {
    if (coeffs(num_zeros) != 0) {
      break;
    }
  }
  return coeffs.tail(coeffs.size() - num_zeros);
}

// Remove trailing zero coefficients.
Eigen::VectorXd RemoveTrailingZeros(const Eigen::VectorXd& coeffs) {
  Eigen::VectorXd::Index num_zeros = 0;
  for (; num_zeros < coeffs.size(); ++num_zeros) {
    if (coeffs(coeffs.size() - 1 - num_zeros) != 0) {
      break;
    }
  }
  return coeffs.head(coeffs.size() - num_zeros);
}

}  // namespace

bool FindLinearPolynomialRoots(const Eigen::VectorXd& coeffs,
                               Eigen::VectorXd* real,
                               Eigen::VectorXd* imag) {
  CHECK_EQ(coeffs.size(), 2);

  if (coeffs(0) == 0) {
    return false;
  }

  if (real != nullptr) {
    real->resize(1);
    (*real)(0) = -coeffs(1) / coeffs(0);
  }

  if (imag != nullptr) {
    imag->resize(1);
    (*imag)(0) = 0;
  }

  return true;
}

bool FindQuadraticPolynomialRoots(const Eigen::VectorXd& coeffs,
                                  Eigen::VectorXd* real,
                                  Eigen::VectorXd* imag) {
  CHECK_EQ(coeffs.size(), 3);

  const double a = coeffs(0);
  if (a == 0) {
    return FindLinearPolynomialRoots(coeffs.tail(2), real, imag);
  }

  const double b = coeffs(1);
  const double c = coeffs(2);
  if (b == 0 && c == 0) {
    if (real != nullptr) {
      real->resize(1);
      (*real)(0) = 0;
    }
    if (imag != nullptr) {
      imag->resize(1);
      (*imag)(0) = 0;
    }
    return true;
  }

  const double d = b * b - 4 * a * c;

  if (d >= 0) {
    const double sqrt_d = std::sqrt(d);
    if (real != nullptr) {
      real->resize(2);
      if (b >= 0) {//利用了韦达定理 x1*x2=c/a
        (*real)(0) = (-b - sqrt_d) / (2 * a);
        (*real)(1) = (2 * c) / (-b - sqrt_d);
      } else {//利用了韦达定理 x1*x2=c/a
        (*real)(0) = (2 * c) / (-b + sqrt_d);
        (*real)(1) = (-b + sqrt_d) / (2 * a);
      }
    }//虚部赋零
    if (imag != nullptr) {
      imag->resize(2);
      imag->setZero();
    }
  } else {
    if (real != nullptr) {
      real->resize(2);
      real->setConstant(-b / (2 * a));
    }
    if (imag != nullptr) {
      imag->resize(2);
      (*imag)(0) = std::sqrt(-d) / (2 * a);
      (*imag)(1) = -(*imag)(0);
    }
  }

  return true;
}
//伴随矩阵法求根
bool FindPolynomialRootsCompanionMatrix(const Eigen::VectorXd& coeffs_all,
                                        Eigen::VectorXd* real,
                                        Eigen::VectorXd* imag) {
  CHECK_GE(coeffs_all.size(), 2);

  Eigen::VectorXd coeffs = RemoveLeadingZeros(coeffs_all);

  const int degree = coeffs.size() - 1;

  if (degree <= 0) {
    return false;
  } else if (degree == 1) {
    return FindLinearPolynomialRoots(coeffs, real, imag);
  } else if (degree == 2) {
    return FindQuadraticPolynomialRoots(coeffs, real, imag);
  }

  // Remove the coefficients where zero is a solution.
  coeffs = RemoveTrailingZeros(coeffs);

  // Check if only zero is a solution.
  if (coeffs.size() == 1) {
    if (real != nullptr) {
      real->resize(1);
      (*real)(0) = 0;
    }
    if (imag != nullptr) {
      imag->resize(1);
      (*imag)(0) = 0;
    }
    return true;
  }

  // Fill the companion matrix.
  Eigen::MatrixXd C(coeffs.size() - 1, coeffs.size() - 1);
  C.setZero();
  for (Eigen::MatrixXd::Index i = 1; i < C.rows(); ++i) {
    C(i, i - 1) = 1;
  }
  C.row(0) = -coeffs.tail(coeffs.size() - 1) / coeffs(0);

  // Solve for the roots of the polynomial.
  //使用Eigen库中的EigenSolver类来求解伴随矩阵的特征值
  Eigen::EigenSolver<Eigen::MatrixXd> solver(C, false);
  if (solver.info() != Eigen::Success) {
    return false;
  }
  // If there are trailing zeros, we must add zero as a solution.
  //这段代码的主要目的是处理特征值的结果，并将其存储在输出参数real和imag中
  const int effective_degree =
      coeffs.size() - 1 < degree ? coeffs.size() : coeffs.size() - 1;

  if (real != nullptr) {
    real->resize(effective_degree);
    real->head(coeffs.size() - 1) = solver.eigenvalues().real();
    if (effective_degree > coeffs.size() - 1) {
      (*real)(real->size() - 1) = 0;
    }
  }
  if (imag != nullptr) {
    imag->resize(effective_degree);
    imag->head(coeffs.size() - 1) = solver.eigenvalues().imag();
    if (effective_degree > coeffs.size() - 1) {
      (*imag)(imag->size() - 1) = 0;
    }
  }

  return true;
}

}  // namespace colmap
```



```c++
//计算重投影误差
void ComputeSquaredReprojectionError(
    const std::vector<Eigen::Vector2d>& points2D,
    const std::vector<Eigen::Vector3d>& points3D,
    const Eigen::Matrix3x4d& cam_from_world,
    std::vector<double>* residuals) {
  const size_t num_points2D = points2D.size();
  CHECK_EQ(num_points2D, points3D.size());
  residuals->resize(num_points2D);
  for (size_t i = 0; i < num_points2D; ++i) {
      //将三维点变换到相机坐标系中
    const Eigen::Vector3d point3D_in_cam =
        cam_from_world * points3D[i].homogeneous();
    // Check if 3D point is in front of camera.
    //如果在相机前方，则计算重投影点与实际二维点之间的欧几里德距离的平方，并将结果存储在 residuals 中。
    //如果不在相机前方，则将对应的 residuals 设为一个较大的值
    if (point3D_in_cam.z() > std::numeric_limits<double>::epsilon()) {
      (*residuals)[i] =
          (point3D_in_cam.hnormalized() - points2D[i]).squaredNorm();
    } else {
      (*residuals)[i] = std::numeric_limits<double>::max();
    }
  }
}
```



![image-20231123140518128.png](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/image-20231123140518128.png)

![image-20231123140536083.png](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/image-20231123140536083.png)



![image-20231123140549415.png](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/image-20231123140549415.png)

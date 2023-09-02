---
comments: true
---

# 翻译：OpenCV相机标定[^1]

## 本文目标

通过本文，我们将会学习与如何使用OpenCV进行相机标定相关的知识。具体的：

1. 由相机导致的图像畸变有哪些类型；
2. 如何通过计算得到相机的内参矩阵和外参矩阵；
3. 如何利用相机的内参矩阵和外参矩阵对图像进行消畸变操作。

## 基础知识

基于小孔成像模型的相机会为所拍摄的图像引入图像畸变问题。其中，最主要的畸变类型是：径向畸变（radial distortion）和切向畸变（tangential distortion）。





一些基于小孔成像原理的相机会给所拍摄的图像引入明显的图像畸变。其中，最主要的畸变类型是：径向畸变(Radial distortion)和切向畸变(Tangential distortion)。

径向畸变会导致场景中的直线在图像中变为曲线。径向畸变具有越远离图像中心越大的特点。例如，下图中，棋盘格两侧的边缘使用红线标记。但是，你可以看到，棋盘格的边缘不再是一条直线，并不严格符合红色的线条。所有理论上的直线都向外膨胀了。想要了解更多相关的信息，可以参考[光学畸变](https://en.wikipedia.org/wiki/Distortion_%28optics%29)的维基百科。

<div align=center>
  <img src="https://docs.opencv.org/4.8.0/calib_radial.jpg">
  <br>
  <label><b>image</b></label>
</div>

径向畸变可以使用下面的式子表示：

$$
x_{distorted} = x(1 + k_{1} r^2 + k_{2} r^4 + k_{3} r^6) \\
y_{distorted} = y(1 + k_{1} r^2 + k_{2} r^4 + k_{3} r^6)
$$

类似的，切向畸变是由相机的镜头和像平面非严格平行引起的。所以，图像中的有些区域看着会比理论上更近些。图像的切向畸变可以使用下面的式子表示：

$$
x_{distorted} = x + [2p_{1}xy + p_{2}(r^2+2x^2)] \\
y_{distorted} = y + [p_{1}(r^2+2y^2) + 2p_{2}xy]
$$

简而言之，我们需要五个参数表示图像的畸变。这些参数也被称为畸变参数：

$$
DistortionCoefficients = (k_1 \quad k_2 \quad p_1 \quad p_2 \quad k_3)
$$

除畸变参数外，我们还需要一些额外的信息，如相机的内部参数和外部参数。内部参数因相机的不同而不同，通常包含相机的焦距($fx,fy$)，和光心位置($cx,cy$)。焦距和光心位置可以用于创建相机矩阵，而相机矩阵可以被用于移除由于相机镜头引起的畸变。由于相机的矩阵是每个相机独有的，所以一旦确定，就可以被所有该相机拍摄的图像复用。相机矩阵可以使用一个$3 \times 3$的矩阵表示：

$$
CameraMatrix = \begin{bmatrix} f_x \quad 0 \quad c_x \\ 0 \quad f_y \quad c_y \\ 0 \quad 0 \quad 1\end{bmatrix}
$$

外部参数相对的，表示用于从3D坐标到相机坐标的旋转和平移等向量。

对于立体视觉应用，这些畸变是首先需要被矫正的。为了得到矫正参数，我们必须提供一些通过精心设计场景的图像，例如棋盘格。我们能够发现一些已知相对关系的特殊点，列入，棋盘格图像中的方块交点。我们知道他们在真实世界空间中的坐标，也知道了他们在图像中的坐标。所以我们能够通过这些信息解得用于畸变参数。为了得到更好的结果，我们需要至少10张测试图像。

## 代码

正如上面所说，我们需要至少10张测试图像用于相机标定。OpenCV包含了一组棋盘格图像，所以我们会利用他们。考虑其中一张棋盘格图像。对于相机标定，最重要的输入数据是一组3维世界点的坐标和对应的二维图像中的坐标。二维图像中的坐标十分容易得到。（这些图像坐标位于棋盘格中任意两个黑色方块相接触的地方。）

那三维世界中的坐标呢？这些图像使用了一个静态的相机拍摄，棋盘格被以不同的位置和方向摆放。所以，我们需要知道$(x,y,z)$坐标值。但为了简单起见，我们可以认为棋盘格始终位于XY平面内，(即$Z$始终为0)，然后相机产生了相对的移动。这样的思考只需要我们得到X，Y即可。现在，考虑$x$和$y$，我们可以简化这些点为$(0,0)$，$(1,0)$，$(2,0)$...这些值就可以表示点的坐标位置。在这种情况下，结果将是棋盘格方块的缩放值。但是，如果我们已经知道了方块的实际大小，例如30mm，我们就可以使用简化的点坐标。并且最终结果的单位是毫米。（在这种情况下，我们并不知道方块的大小，因为我们并未拍摄图像，所以我们略过了方块大小的参数。）

3维的点被称为物体点，二维图像中的点被称为图像点

### 准备

为了能够在棋盘格图像中搜寻到图案，我们使用`cv.findChessboardCorners()`函数。同时，我们需要传递一个参数，用于表示我们需要搜寻什么样的图案。例如，$8 \times 8$的方格，$5 \times 5$的方格等。在本文的例子中，我们使用$7 \times 6$的方格。（通常情况下，一个棋盘格拥有$8 \times 8$的方块和$7 \times 7$个内部交点）。这个函数将会返回相应的交点和一个用于表示图案是否找到的返回值`retval`。如果找了相应的图案，`retval`为`True`。这些图案将会按序排列，例如，从左到右从上到下。

> 需要注意：这个函数有可能在所有的图像中都无法找到需要的图案。因此，一个比较好的做法是：启动相机，然后拍摄一帧图像进行所需图案的检测。一旦在所拍摄的图案中找到了所需的图案，就进行交点的搜索并将结果存储在一个数组中。此外，在拍摄一下帧图像前，预留一些间隔，这样可以调整棋盘格图像的位置和方向。重复上述的过程，知道满足所需的图像数量。即便是本文中提供的例子，我们也无法百分百保证14张图像中有多少张图像是能够找到对应图案的。因此，我们需要读取所有的图像，然后只选取其中能够检测出图案的那些。除了棋盘格，我们也可以选择使用圆形阵列。在这种情况下，我们可以使用函数`cv.findCirclesGrid()`搜索图案。使用圆形阵列可以使用更少的图像来满足进行相机标定的条件。

Once we find the corners, we can increase their accuracy using cv.cornerSubPix(). We can also draw the pattern using cv.drawChessboardCorners(). All these steps are included in below code：

一旦能够找到棋盘格中的交点，我们就可以使用`cv.cornerSubPix()`函数来提高他们的精度。同时，我们还可以使用`cv.drawChessboardCorners()`在图像中绘制找到的交点。上述的所有步骤都可以用下面的代码完成：

```python
import numpy as np
import cv2 as cv
import glob
# termination criteria
criteria = (cv.TERM_CRITERIA_EPS + cv.TERM_CRITERIA_MAX_ITER, 30, 0.001)
# prepare object points, like (0,0,0), (1,0,0), (2,0,0) ....,(6,5,0)
objp = np.zeros((6*7,3), np.float32)
objp[:,:2] = np.mgrid[0:7,0:6].T.reshape(-1,2)
# Arrays to store object points and image points from all the images.
objpoints = [] # 3d point in real world space
imgpoints = [] # 2d points in image plane.
images = glob.glob('*.jpg')
for fname in images:
 img = cv.imread(fname)
 gray = cv.cvtColor(img, cv.COLOR_BGR2GRAY)
 # Find the chess board corners
 ret, corners = cv.findChessboardCorners(gray, (7,6), None)
 # If found, add object points, image points (after refining them)
 if ret == True:
 objpoints.append(objp)
 corners2 = cv.cornerSubPix(gray,corners, (11,11), (-1,-1), criteria)
 imgpoints.append(corners2)
 # Draw and display the corners
 cv.drawChessboardCorners(img, (7,6), corners2, ret)
 cv.imshow('img', img)
 cv.waitKey(500)
cv.destroyAllWindows()
```

One image with pattern drawn on it is shown below:

其中一张包含搜寻到的图案的图像如下图所示。

<div align=center>
  <img src="https://docs.opencv.org/4.8.0/calib_pattern.jpg">
  <br><b>image</b></br>
</div>

### 标定

Now that we have our object points and image points, we are ready to go for calibration. We can use the function, cv.calibrateCamera() which returns the camera matrix, distortion coefficients, rotation and translation vectors etc.

现在，我们有了物体点和图像点，就可以进行相机标定了。我们可以使用`cv.calibrateCamera()`完成。这个函数将会返回相机矩阵，即便参数，旋转矩阵和平移矩阵。

```python
ret, mtx, dist, rvecs, tvecs = cv.calibrateCamera(objpoints, imgpoints, gray.shape[::-1], None, None)
```

### 消畸变

Now, we can take an image and undistort it. OpenCV comes with two methods for doing this. However first, we can refine the camera matrix based on a free scaling parameter using cv.getOptimalNewCameraMatrix(). If the scaling parameter alpha=0, it returns undistorted image with minimum unwanted pixels. So it may even remove some pixels at image corners. If alpha=1, all pixels are retained with some extra black images. This function also returns an image ROI which can be used to crop the result.

现在，我们可以使用拍摄一张图像，然后对该图像进行消畸变操作。OpenCV提供了两种方式来完成。但在此之前，我们需要利用函数`cv.getOptimalNewCameraMatrix`对相机参数进行优化。这个优化基于一个自由度参数。如果自由度参数`alpha=0`，那么该函数将会返回一个包含最少不理想像素的消畸变图像。所以，使用这个函数优化后的相机矩阵进行消畸变可能会在图像的四周移除部分像素。如果`alpha=0`，那么所有的像素都会被返回，也会包含一些额外的黑色像素。此外，该函数还返回了一个图像的ROI，用于裁剪结果。

现在，我们将使用一张新的图像进行消畸变（本文例子中使用的是left12.jpg，也就是本文的第一张图像）。

```python
img = cv.imread('left12.jpg')
h, w = img.shape[:2]
newcameramtx, roi = cv.getOptimalNewCameraMatrix(mtx, dist, (w,h), 1, (w,h))
```

#### 方式1 使用`cv.undistort()`函数

This is the easiest way. Just call the function and use ROI obtained above to crop the result.

这是最简单的方式。调用这个函数，然后使用上文提到的ROI裁剪图像即可。

```python
# undistort
dst = cv.undistort(img, mtx, dist, None, newcameramtx)
# crop the image
x, y, w, h = roi
dst = dst[y:y+h, x:x+w]
cv.imwrite('calibresult.png', dst)
```

#### 方式2 使用`remapping`函数

这种方式有一点复杂。首先，找到一个从含畸变图像到消畸变图像的映射函数。然后使用这个映射函数。

```python
# undistort
mapx, mapy = cv.initUndistortRectifyMap(mtx, dist, None, newcameramtx, (w,h), 5)
dst = cv.remap(img, mapx, mapy, cv.INTER_LINEAR)
# crop the image
x, y, w, h = roi
dst = dst[y:y+h, x:x+w]
cv.imwrite('calibresult.png', dst)
```

不管怎样，两种方式都会给出相同的结果。结果如下。

<div align=center>
  <img src="https://docs.opencv.org/4.8.0/calib_result.jpg"></img>
  <br><b>image</b></br>
</div>

可以返现，结果中所有的边缘都变直了。

现在，你可以使用Numpy中的函数（如np.savez，np.savetxt）来保存相机矩阵和畸变参数以便将来使用。

### 重投影误差

重投影误差可以很好的估计用于消畸变的参数的质量。重投影误差越接近于0，说明消畸变参数越精确。在得到相机的内部参数、即便参数、旋转和平移矩阵后，我们首先需要使用函数`cv.projectPoints()`将物体点转换到图像点。然后，我们就可以计算转换得到的图像点和实际图像点之间的曼哈顿距离。为了得到平均的误差，我们会对所有图像的重投影误差求算术平均作为最终的结果。

```python
mean_error = 0
for i in range(len(objpoints)):
 imgpoints2, _ = cv.projectPoints(objpoints[i], rvecs[i], tvecs[i], mtx, dist)
 error = cv.norm(imgpoints[i], imgpoints2, cv.NORM_L2)/len(imgpoints2)
 mean_error += error
print( "total error: {}".format(mean_error/len(objpoints)) )
```

## 额外的资源

### 练习

1. 尝试使用圆形阵列进行相机标定。

[^1]: [OpenCV: Camera Calibration](https://docs.opencv.org/4.8.0/dc/dbb/tutorial_py_calibration.html)

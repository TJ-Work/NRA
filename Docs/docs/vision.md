## 机器人视觉

### ALPhotoCapture模块

`ALPhotoCapture`模块允许你使用机器人相机拍摄照片，并保存到磁盘上。

#### `ALPhotoCapture.getCameraID() `

返回用于拍照的摄像头ID

> ***Returns***: 
>
> `camera ID` - 摄像头ID

#### `ALPhotoCapture.getCaptureInterval(delay) `

返回当前两幅图片之间的延迟(以毫秒为单位)。

> ***Returns***: 
>
> `delay(ms)` - 延时

#### `ALPhotoCapture.setCameraID (cameraID) `

设置用于拍照的摄像头ID

> ***Parameters***: 
>
> `cameraID`： Id of the camera

#### `ALPhotoCapture.setCaptureInterval (captureInterval) `

设置两次连续拍照之间的间隔时间。

> ***Parameters***:  
>
> `captureInterval`：以毫秒为单位，默认值200，相当于每秒拍摄5张图像。

#### `ALPhotoCapture.takePicture(folderPath，fileName) `

以给定的分辨率拍摄一张照片并将其存储在磁盘上。文件格式由扩展名决定(参见ALPhotoCapture.setPictureFormat支持的扩展名列表)。如果目标文件已经存在，它将被覆盖。

> ***Parameters***: 
>
> `folderPath`：- 文件保存路径          
>
> `fileName`：- 文件名字
>
> ***Returns***:  
>
> 一个元素的数组 形式为   [fileName]

#### `ALPhotoCapture.setResolution (resolution) `

设置拍照的分辨率。

> ***Parameters***:  
>
> `resolution`：Nao机器人支持的分辨率

#### `ALPhotoCapture.setPictureFormat(pictureFormat) `

设置用于保存图片的图像格式, 这对应于添加到文件名中的文件扩展名。

> ***Parameters***:  
>
> `pictureFormat`：- 文件格式 ： “bmp”, “dib”, “jpeg”, “jpg”, “jpe”, “png”, “pbm”, “pgm”, “ppm”, “sr”, “ras”, “tiff”, “tif”

#### `ALPhotoCapture.takePictures(nubmerOfPictures, folderPath，fileName) `

连续拍摄几张照片并存储在磁盘上。如果目标文件已经存在，它将被覆盖。

> ***Parameters***:  
>
> `numberOfPictures` - 需要拍照的数量
>
> `folderPath`：- 文件保存路径
>
> `fileName`：- 文件名字
>
> ***Returns***:  
>
> 一个元素的数组 形式为   [fileName]

#### 实例

本教程解释了如何使用Python运行ALPhotoCapture模块，在一些初始化步骤之后，我们首先向ALPhotoCapture模块实例化一个代理。

```python
# This test demonstrates how to use the ALPhotoCapture module.
# Note that you might not have this module depending on your distribution
import os
import sys
import time
from naoqi import ALProxy

# Replace this with your robot's IP address
IP = "10.0.252.91"
PORT = 9559

# Create a proxy to ALPhotoCapture
try:
  photoCaptureProxy = ALProxy("ALPhotoCapture", IP, PORT)
except Exception, e:
  print "Error when creating ALPhotoCapture proxy:"
  print str(e)
  exit(1)
```

调用函数进行拍照

```python
# Take 3 pictures in VGA and store them in /home/nao/recordings/cameras/

photoCaptureProxy.setResolution(2)
photoCaptureProxy.setPictureFormat("jpg")
photoCaptureProxy.takePictures(3, "/home/nao/recordings/cameras/", "image")

# This call returns ['/home/nao/recordings/cameras/image_0.jpg', '/home/nao/recordings/cameras/image_1.jpg', '/home/nao/recordings/cameras/image_2.jpg']
```

### ALVideoDevice模块

`ALVideoDevice`模块负责将视频源(如机器人的摄像头、模拟器)的图像高效地提供给所有处理模块，如`ALFaceDetection`或`ALVisionRecognition`。由于`ALVideoDevice`模块随时知道需要图像的模块列表及其需求，因此可以设置视频设备的最小配置，既能满足各个模块的需求，又能节省处理资源。

#### `ALVideoDevice.subscribeCamera(Name, CameraIndex, Resolution, ColorSpace, Fps)`

订阅一个ALVideoDevice，当视频模块注册到ALVideoDevice时，请求的图像格式的缓冲区被添加到缓冲区列表中。该函数返回一个句柄，该句柄标识ALVideoDevice中的模块。(此句柄基于name参数，例如，第三个句柄在其名称中添加_3来构建此句柄)。

> ***Parameters***:  
>
> `Name` - 订阅的模块名称
>
> `CameraIndex` ：- 视频系统中摄像机的索引。
>
> `Resolution` ：- 分辨率
>
> `ColorSpace`：- 颜色空间
>
> `Fps`: -帧率 
>
> ***Returns***: 
>
> 字符串句柄，该句柄下的模块是ALVideoDevice已知的，如果发生错误，返回空字符串

#### `ALVideoDevice.getImageLocal(Handle)`

从视频源获取最新的图像，对图像进行最终转换，以提供视觉模块所要求的格式，并返回一个指向已锁定的AL.ALImage图像模块的指针。

> ***Parameters***:  
>
> `Handle`- 订阅函数返回的句柄
>
> ***Returns***:  
>
> 指向锁定的图像缓冲区的指针，如果错误返回NULL(例如，如果以前的图像没有被释放)。

#### `ALVideoDevice.getImageRemote(Handle)`

从视频源获取最新的图像，对图像应用最终的转换，以提供vision模块所要求的格式，并将其作为一个值通过网络发送。

> ***Parameters***:  
>
> `Handle`- 订阅函数返回的句柄
>
> ***Returns***:  
>
> 包含图像的类

#### `ALVideoDevice.releaseImage(Handle)`

释放被`ALVideoDevice.getImageLocal()`锁定的图像缓冲区。如果模块没有锁定的图像缓冲区，什么都不做。

> ***Parameters***:  
>
> `Handle`- 订阅函数返回的句柄
>
> ***Returns***:  
>
> 成功返回True

#### `ALVideoDevice.unsubscribe(Handle)`

从`ALVideoDevice`注销一个模块

> ***Parameters***:  
>
> `Handle`- 订阅函数返回的句柄
>
> ***Returns***:  
>
> 成功返回True

#### 用法

| Step | Action                                                       |
| :--- | :----------------------------------------------------------- |
| 1    | Make your vision module subscribe to the **ALVideoDevice** proxy by calling [`ALVideoDeviceProxy::subscribeCamera `](https://developer.softbankrobotics.com/nao6/naoqi-developer-guide/naoqi-apis/naoqi-vision/alvideodevice/alvideodevice-api#ALVideoDeviceProxy::subscribeCamera__ssCR.iCR.iCR.iCR.iCR)and passing it parameters such as resolution, color space and frame rate. |
| 2    | In the main process loop, get an image by calling [`ALVideoDeviceProxy::getImageLocal `](https://developer.softbankrobotics.com/nao6/naoqi-developer-guide/naoqi-apis/naoqi-vision/alvideodevice/alvideodevice-api#ALVideoDeviceProxy::getImageLocal__ssCR)or [`ALVideoDeviceProxy::getImageRemote `](https://developer.softbankrobotics.com/nao6/naoqi-developer-guide/naoqi-apis/naoqi-vision/alvideodevice/alvideodevice-api#ALVideoDeviceProxy::getImageRemote__ssCR)(depending on whether your module is local or remote). |
| 3    | Release the image calling [`ALVideoDeviceProxy::releaseImage `](https://developer.softbankrobotics.com/nao6/naoqi-developer-guide/naoqi-apis/naoqi-vision/alvideodevice/alvideodevice-api#ALVideoDeviceProxy::releaseImage__ssCR), |
| 4    | When you stop your module, call [`ALVideoDeviceProxy::unsubscribe `](https://developer.softbankrobotics.com/nao6/naoqi-developer-guide/naoqi-apis/naoqi-vision/alvideodevice/alvideodevice-api#ALVideoDeviceProxy::unsubscribe__ssCR)after exiting the main loop. |

### ALVideoRecorder模块

`ALVideoRecorder`模块允许您使用机器人摄像机录制视频序列，并将它们保存在磁盘上。

#### `ALVideoRecorder.setCameraID(cameraID)`

设置用于录制的摄像头ID

> ***Parameters***:  
>
> `cameraID`： Id of the camera

#### `ALVideoRecorder.setColorSpace(colorSpace)`

设置使用的颜色空间

> ***Parameters***:  
>
> `colorSpace`：它可以是kBGRColorSpace(颜色)或kYuvColorSpace(灰度)。

#### `ALVideoRecorder.setFrameRate(frameRate)`

设置每秒帧数

> ***Parameters***:  
>
> `frameRate`：帧速率在1到30之间(VGA分辨率最大为15帧/秒)。

#### `ALVideoRecorder.setResolution(resolution)`

设置分辨率

> ***Parameters***:  
>
> `resolution`：分辨率为 ： 0 (QQVGA)， 1 (QVGA)或2 (VGA)

#### `ALVideoRecorder.setVideoFormat(videoFormat)`

设置用于编码视频的方式。

> ***Parameters***:  
>
> `videoFormat`：“IYUV”(院士 avi，仅限彩色视频)      “MJPG”(压缩avi，灰度和彩色视频)

#### `ALVideoRecorder.startRecording(folderPath, fileName)`

以输入参数开始录像，直到`ALVideoRecorderProxy::stopRecording`被调用。如果目标文件已经存在，它将被覆盖，同一时间只能录制一个视频。

> ***Parameters***:  
>
> `folderPath`：视频保存的位置
>
> `filename`： 视频保存的名称

#### `ALVideoRecorder.stopRecording()`

停止用`ALVideoRecorderProxy::startRecording`启动的视频录制。该函数返回所记录的帧数以及视频的绝对路径。

> ***Returns***:   
>
> `[numRecordedFrames, recordAbsolutePath]`

#### 实例

在一些初始化步骤之后，我们首先实例化`ALVideoRecorder`模块的代理。

```python
# This test demonstrates how to use the ALVideoRecorder module.
# Note that you might not have this module depending on your distribution
import os
import sys
import time
from naoqi import ALProxy

# Replace this with your robot's IP address
IP = "10.0.252.91"
PORT = 9559

# Create a proxy to ALVideoRecorder
try:
  videoRecorderProxy = ALProxy("ALVideoRecorder", IP, PORT)
except Exception, e:
  print "Error when creating ALVideoRecorder proxy:"
  print str(e)
  exit(1)
```

开始记录一个视频

```python
videoRecorderProxy.setFrameRate(10.0)
videoRecorderProxy.setResolution(2) # Set resolution to VGA (640 x 480)
# We'll save a 5 second video record in /home/nao/recordings/cameras/
videoRecorderProxy.startRecording("/home/nao/recordings/cameras", "test")
print "Video record started."

time.sleep(5)

videoInfo = videoRecorderProxy.stopRecording()
print "Video was saved on the robot: ", videoInfo[1]
print "Total number of frames: ", videoInfo[0]
```

### ALRedBallDetection模块

`ALRedBallDetection`是一个模块，提供了一个基于快速视觉的红球检测器。`ALRedBallDetection`是基于相机图像中红色像素进行检测，根据这些像素与YUV色彩空间中的红色值的距离使用一个阈值进行过滤，计算该阈值实现了在光照条件变化时进行检测。然后，从所有检测到的红色像素集合中，只保留那些定义圆形的像素。当在当前图像上找到一组合适像素时，`ALMemory`键`redBallDetected`被更新，并返回如下的关键字：

```guess
[
  TimeStamp,
  BallInfo,
  CameraPose_InTorsoFrame,
  CameraPose_InRobotFrame,
  Camera_Id
]
```

`TimeStamp`：该字段是用于执行检测的图像的时间戳。

```guess
TimeStamp [
  TimeStamp_Seconds,
  Timestamp_Microseconds
]
```

`BallInfo`

```guess
BallInfo [
  centerX,
  centerY,
  sizeX,
  sizeY
]
```

- `centerX` 和 `centerY` 是球中心的角度坐标，以角度(弧度)表示。
- `sizeX` 和 `sizeY` 是球的“水平和垂直半径”的角度(弧度)。

坐标系的原点在图像的中间`centerX` 对应于沿Z轴的正(逆时针)旋转， `centerY` 对应于沿Y轴的正旋转，如下图所示:

![/sites/default/files/repository/70_html_nao/_images/vision_cameraangles.png](https://developer.softbankrobotics.com/sites/default/files/repository/70_html_nao/_images/vision_cameraangles.png)

`CameraPose_InTorsoFrame` : 在躯干坐标系中描述了图像被拍摄时相机的位置

`CameraPose_InRobotFrame` : 在机器人坐标系中描述了图像被拍摄时相机的位置

`Camera_Id` : 给出用于检测的摄像机的Id(0为顶部摄像机，1为底部摄像机)。

### ALVisionRecognition模块

`ALVisionRecognition`是一个视觉模块，机器人可以在这个模块中尝试识别不同的图片、物体、侧面甚至是之前学习过的位置。该模块以视觉关键点的识别为基础，仅针对之前学习过的特定对象进行识别。

#### 检测结果

像所有其他提取模块一样，识别结果被放在`ALMemory`中。当某个东西被识别时，你会看到一个`ALValue`字段，其结构如下:

“`PictureDetected`” 关键字结构如下

```guess
[
  TimeStamp,
  PictureInfo[N]
]
```

`PictureInfo`标签数量与当前识别到物体的数量一样多。

`TimeStamp`

该字段是用于执行检测的图像的时间戳。

```guess
TimeStamp =
[
  TimeStamp_Seconds,
  Timestamp_Microseconds
]
```

`PictureInfo`

对于每个检测到的图片，我们有一个`PictureInfo`字段:

```guess
PictureInfo =
[
  Label[N],
  MatchedKeypoints,
  Ratio,
  BoundaryPoint[N]
]
```

- `Label` : 有组织的图片名称 (e.g. [“cover”, “my book”], or [“fridge corner”, “kitchen”, “my flat”]).
- `MatchedKeypoints` 在当前帧中为对象检索到的关键点数量。
- `Ratio` 是当前帧中检索到的对象的关键点数除以对象在学习阶段中找到的关键点数。
- `BoundaryPoint` 是表示学习阶段选择的边界在当前图像中重新投影的角度值(弧度)中的点坐标列表。

```guess
BoundaryPoint =
[
  x,
  y
]
```

#### `ALVisionRecognition.detectFromFile (filename)`

在给定图像文件中搜索已知对象。

> ***Parameters***:   
>
> `filename` – 要搜索的图像的绝对文件名。

#### `ALVisionRecognition.learnFromFile (filename, name, tags, isWholeImage, forced)`

从给定的文件名加载图像，并学习它与给定的信息，名称，标签，边界。

> ***Parameters***:   
>
> `image` – 要搜索的图像的绝对文件名。
>
> `name` - 对象的名称，必须是非空字符串。
>
> `tags` - 与对象一起存储的对象标签，可以为空。
>
> `isWholeImage` - Bool变量，默认为False。 True表示物体占据了整个图像，False表示自动检测物体的边界。仅当物体从单色背景中突出时使用。
>
> `forced` - Bool变量，默认为False。True表示学习对象将替换可能存在的对象(条件:具有相同的原始文件名)。False:如果一个对象已经以相同的文件名存在，它将不会被替换，并且方法将中止。
>
> ***Return***：
>
> 成功返回True，否则返回False

#### 实例

本教程介绍从 Python 访问 `ALVisionRecognition`模块的两种方法， 订阅包含 `ALVisionRecognition`的`AlMemory`密钥。

import一些环境和变量

```python
import os
import sys
import time
import naoqi
from naoqi import *

# A global counter of the number of loops
count = 10

PC_IP = "127.0.0.1"  # Replace this with your computer's IP address
NAO_IP = "nao.local" # Replace this with your NAOqi's IP address

# The name of the event generated by ALVisionRecognition
event_name = "PictureDetected"
```

我们使用回调方法创建 python 模块并实例化它

```python
# The name of our local python module
module_name = "python_module"

class myModule(ALModule):
"""python class myModule test auto documentation"""

  def dataChanged(self, strVarName, value, strMessage):
    """callback when data change"""

    print "datachanged", strVarName, " ", value, " ", strMessage
    global count
    count = count - 1

# Create a local broker, connected to the remote naoqi
broker = ALBroker("pythonBroker", PC_IP, 9999, NAO_IP, 9559)

# Create a python module
pythonModule = myModule(module_name)
```

订阅事件，以便可以调用我们的回调

```python
try:
  # Create a proxy to ALMemory
  memoryProxy = ALProxy("ALMemory", ROBOT_IP, PORT)

  # Subscribe to the event, saying where we want to be called back
  memoryProxy.subscribeToEvent(event_name, module_name, "dataChanged")

  # Let the picture recognition run
  while count > 0:
    time.sleep(5)

  # Unsubscribe
  memoryProxy.unsubscribeToEvent(event_name, module_name)

except RuntimeError, e:
  print e
  exit(1)

print 'end'
```

### ALLocalization模块

`ALLocalization`是一个模块，致力于在室内环境中对机器人*进行*定位。

#### `ALLocalization.learnHome()`

学习机器人的主节点

> ***Returns***:  
>
> 无错误返回0，否则返回错误代码

#### `ALLocalization.getRobotPosition(active)`

学习机器人的主节点

> ***Parameters***: 
>
> `active` - 默认为False， 定义里程计不可靠是的重定位过程，如果为True, 机器人会重新扫描来定位，如果为假，机器人将不会移动。
>
> ***Returns***:  
>
> 机器人2D的坐标(x, y)。

#### `ALLocalization.goToHome ()`

如果需要,将机器人移动到执行`ALLocalization.learnHome`的地方。准确性取决于环境。

> ***Returns***:  
>
> 如果无错误返回0， 否则返回错误代码

#### 实例

```python
motionProxy  = ALProxy("ALMotion", robotIP, PORT)
localizationProxy = ALProxy("ALLocalization", robotIP, PORT)

# Learning home.
ret = localizationProxy.learnHome()
# Check that no problem occurred.
if ret == 0:
  print "Learning OK"
else:
  print "Error during learning " + str(ret)

# Make some moves.
motionProxy.moveTo(0.5, 0.0, 0.2)

# Go back home.
ret = localizationProxy.goToHome()
# Check that no problem occurred.
if ret == 0:
  print "go to home OK"
else:
  print "error during go to home " + str(ret)

# Save the data for later use.
ret = localizationProxy.save("example")
# Check that no problem occurred.
if ret == 0:
  print "saving OK"
else:
  print "error during saving" + str(ret)
```


## 机器人视觉

### `ALPhotoCapture `

`ALPhotoCapture`模块允许你使用机器人相机拍摄照片，并保存到磁盘上。

#### `ALPhotoCapture.getCameraID() `

返回用于拍照的摄像头ID

> ***Returns***: camera ID

#### `ALPhotoCapture.getCaptureInterval(delay) `

返回当前两幅图片之间的延迟(以毫秒为单位)。

> ***Returns***: delay(ms)

#### `ALPhotoCapture.setCameraID (cameraID) `

设置用于拍照的摄像头ID

> ***Parameters***:  *cameraID*： Id of the camera

#### `ALPhotoCapture.setCaptureInterval (captureInterval) `

设置两次连续拍照之间的间隔时间。

> ***Parameters***:  captureInterval：以毫秒为单位，默认值200，相当于每秒拍摄5张图像。

#### `ALPhotoCapture.takePicture(folderPath，fileName) `

以给定的分辨率拍摄一张照片并将其存储在磁盘上。文件格式由扩展名决定(参见ALPhotoCapture.setPictureFormat支持的扩展名列表)。如果目标文件已经存在，它将被覆盖。

> ***Parameters***:  folderPath：- 文件保存路径
>
> ​						 *fileName*：- 文件名字
>
> ***Returns***:  一个元素的数组 形式为   [fileName]

#### `ALPhotoCapture.setResolution (resolution) `

设置拍照的分辨率。

> ***Parameters***:  *resolution：*Nao机器人支持的分辨率

#### `ALPhotoCapture.setPictureFormat(pictureFormat) `

设置用于保存图片的图像格式, 这对应于添加到文件名中的文件扩展名。

> ***Parameters***:  *pictureFormat*：- 文件格式 ： “bmp”, “dib”, “jpeg”, “jpg”, “jpe”, “png”, “pbm”, “pgm”, “ppm”, “sr”, “ras”, “tiff”, “tif”

#### `ALPhotoCapture.takePictures(nubmerOfPictures, folderPath，fileName) `

连续拍摄几张照片并存储在磁盘上。如果目标文件已经存在，它将被覆盖。

> ***Parameters***:  *numberOfPictures* - 需要拍照的数量
>
> ​						 folderPath：- 文件保存路径
>
> ​						 *fileName*：- 文件名字
>
> ***Returns***:  一个元素的数组 形式为   [fileName]

#### Examples

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

### `ALVideoDevice`

`ALVideoDevice`模块负责将视频源(如机器人的摄像头、模拟器)的图像高效地提供给所有处理模块，如`ALFaceDetection`或`ALVisionRecognition`。由于`ALVideoDevice`模块随时知道需要图像的模块列表及其需求，因此可以设置视频设备的最小配置，既能满足各个模块的需求，又能节省处理资源。

`ALVideoDevice.subscribeCamera(Name, CameraIndex, Resolution, ColorSpace, Fps)`

订阅一个ALVideoDevice，当视频模块注册到ALVideoDevice时，请求的图像格式的缓冲区被添加到缓冲区列表中。该函数返回一个句柄，该句柄标识ALVideoDevice中的模块。(此句柄基于name参数，例如，第三个句柄在其名称中添加_3来构建此句柄)。

> ***Parameters***:  *Name* - 订阅的模块名称
>
> ​						 CameraIndex ：- 视频系统中摄像机的索引。
>
> ​						 Resolution ：- 分辨率
>
> ​						 ColorSpace：- 颜色空间
>
> ​	     				 Fps: -帧率 
>
> ***Returns***:  字符串句柄，该句柄下的模块是ALVideoDevice已知的，如果发生错误，返回空字符串

`ALVideoDevice.getImageLocal(Handle)`

从视频源获取最新的图像，对图像进行最终转换，以提供视觉模块所要求的格式，并返回一个指向已锁定的AL.ALImage图像模块的指针。

> ***Parameters***:  Handle- 订阅函数返回的句柄
>
> ***Returns***:  指向锁定的图像缓冲区的指针，如果错误返回NULL(例如，如果以前的图像没有被释放)。

`ALVideoDevice.getImageRemote(Handle)`

从视频源获取最新的图像，对图像应用最终的转换，以提供vision模块所要求的格式，并将其作为一个值通过网络发送。

> ***Parameters***:  Handle- 订阅函数返回的句柄
>
> ***Returns***:  包含图像的类

`ALVideoDevice.releaseImage(Handle)`

释放被`ALVideoDevice.getImageLocal()`锁定的图像缓冲区。如果模块没有锁定的图像缓冲区，什么都不做。

> ***Parameters***:  Handle- 订阅函数返回的句柄
>
> ***Returns***:  成功返回True

`ALVideoDevice.unsubscribe(Handle)`

从`ALVideoDevice`注销一个模块

> ***Parameters***:  Handle- 订阅函数返回的句柄
>
> ***Returns***:  成功返回True

#### Get Started

| Step | Action                                                       |
| :--- | :----------------------------------------------------------- |
| 1    | Make your vision module subscribe to the **ALVideoDevice** proxy by calling [`ALVideoDeviceProxy::subscribeCamera `](https://developer.softbankrobotics.com/nao6/naoqi-developer-guide/naoqi-apis/naoqi-vision/alvideodevice/alvideodevice-api#ALVideoDeviceProxy::subscribeCamera__ssCR.iCR.iCR.iCR.iCR)and passing it parameters such as resolution, color space and frame rate. |
| 2    | In the main process loop, get an image by calling [`ALVideoDeviceProxy::getImageLocal `](https://developer.softbankrobotics.com/nao6/naoqi-developer-guide/naoqi-apis/naoqi-vision/alvideodevice/alvideodevice-api#ALVideoDeviceProxy::getImageLocal__ssCR)or [`ALVideoDeviceProxy::getImageRemote `](https://developer.softbankrobotics.com/nao6/naoqi-developer-guide/naoqi-apis/naoqi-vision/alvideodevice/alvideodevice-api#ALVideoDeviceProxy::getImageRemote__ssCR)(depending on whether your module is local or remote). |
| 3    | Release the image calling [`ALVideoDeviceProxy::releaseImage `](https://developer.softbankrobotics.com/nao6/naoqi-developer-guide/naoqi-apis/naoqi-vision/alvideodevice/alvideodevice-api#ALVideoDeviceProxy::releaseImage__ssCR), |
| 4    | When you stop your module, call [`ALVideoDeviceProxy::unsubscribe `](https://developer.softbankrobotics.com/nao6/naoqi-developer-guide/naoqi-apis/naoqi-vision/alvideodevice/alvideodevice-api#ALVideoDeviceProxy::unsubscribe__ssCR)after exiting the main loop. |

### `ALVideoRecorder`

`ALVideoRecorder`模块允许您使用机器人摄像机录制视频序列，并将它们保存在磁盘上。

`ALVideoRecorder.setCameraID(cameraID)`

设置用于录制的摄像头ID

> ***Parameters***:  *cameraID*： Id of the camera

`ALVideoRecorder.setColorSpace(colorSpace)`

设置使用的颜色空间

> ***Parameters***:  *colorSpace*：它可以是kBGRColorSpace(颜色)或kYuvColorSpace(灰度)。

`ALVideoRecorder.setFrameRate(frameRate)`

设置每秒帧数

> ***Parameters***:  *frameRate*：帧速率在1到30之间(VGA分辨率最大为15帧/秒)。

`ALVideoRecorder.setResolution(resolution)`

设置分辨率

> ***Parameters***:  *resolution*：分辨率为 ： 0 (QQVGA)， 1 (QVGA)或2 (VGA)

`ALVideoRecorder.setVideoFormat(videoFormat)`

设置用于编码视频的方式。

> ***Parameters***:  *videoFormat*：“IYUV”(院士 avi，仅限彩色视频)
>
> ​													“MJPG”(压缩avi，灰度和彩色视频)

`ALVideoRecorder.startRecording(folderPath, fileName)`

以输入参数开始录像，直到`ALVideoRecorderProxy::stopRecording`被调用。如果目标文件已经存在，它将被覆盖，同一时间只能录制一个视频。

> ***Parameters***:  *folderPath*：视频保存的位置
>
> ​						 *filename*： 视频保存的名称

`ALVideoRecorder.stopRecording()`

停止用`ALVideoRecorderProxy::startRecording`启动的视频录制。该函数返回所记录的帧数以及视频的绝对路径。

> ***Returns***:   [numRecordedFrames, recordAbsolutePath]

#### Example

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
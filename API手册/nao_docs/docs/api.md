# Nao机器人API手册

**上海意赋教育公司(ifcreate)**

**修订日期：** *2020-08-26*

## 适用环境

- Win10&Mac&Linux
- Python2.7

## 特征

Python API允许用户：

- 在同一局域网内远程调用Nao机器人的所有C++ API
- 适用Python建立可以远程工作的机器人app

## 使用ALProxy代理连接

### 获取机器人ip地址

1. [使用网线进行连接后，从电脑端的app中设置机器人连接无线网](https://developer.softbankrobotics.com/nao6/nao-documentation/nao-user-guide/first-steps-nao/get-robot-settings#robot-settings) 
2. ![/sites/default/files/repository/70_html_nao/_images/note_ipaddress.png](E:/my_blogs/source/images/note_ipaddress-1598428326016.png)
3. 快速按下**NAO**的胸部按钮一次。NAO会复述他的IP地址的四个数字，用**点分隔**。

### ALProxy

`ALProxy `是一个代理对象，它允许用户通过机器人Nao在局域网内的IP地址和端口（port）访问连接到的所有方法或模块（如：`ALTextToSpeech`模块等等。

> class(module_name, ip_address, port)
>
> - module_name: 需要连接到的功能模块等
> - ip_address: 机器人nao的ip地址
> - port：Nao机器人侦听的端口（默认为9559）

模块的每个方法都可以通过对象直接访问，例如以下例程通过`ALTextToSpeech`的`say`方法实现远程控制Nao机器人说Hello World：

```PYTHON
from naoqi import ALProxy
tts = ALProxy("ALTextToSpeech", "<IP of your robot>", 9559)
tts.say("Hello, world!")
```

## 并行任务执行

用户初始化的每一个`ALProxy`对象都有一个名为`post`(代码中第五行)的属性，这能让每一个代理调用的方法同步=执行，而不需要按照正常python的执行顺序顺序执行，如以下实例实现了Nao机器人行走同时并说话：

```Python
from naoqi import ALProxy
motion = ALProxy("ALMotion", "<IP of your robot>", 9559)
tts    = ALProxy("ALTextToSpeech", "<IP of your robot>", 9559)
motion.setStiffnesses("Body", 1.0)
motion.moveInit()
motion.post.moveTo(0.5, 0, 0)
tts.say("I'm walking")
```

> 1. 除非您将关节的刚度设置为非0 的，否则机器人不会移动。为此，只需调用`ALMotion.setStiffnesses`方法， 如代码中第四行所示
> 2. `ALMotion.moveInit()` API 将机器人初始化在一个合适的位置（准备移动）
> 3. `ALMotion.moveTo() ` API 是机器人朝制定方向移动一段距离，单位（m) 
> 4. `ALMotion.post`使并行执行能

如果您需要等待直到给定任务完成，您可以使用ALProxy的wait方法，参数为post方法返回的任务id，示例如下：

```python
from naoqi import ALProxy
motion = ALProxy("ALMotion", "<IP of your robot>", 9559)
motion.moveInit()
id = motion.post.moveTo(0.5, 0, 0)
motion.wait(id, 0)
```

## Naoqi接口模块

### 机器人运动

#### ALRobotPosture模块

##### `ALRobotPosture.getPostureList()`

> <u>***Return：***</u>所有预定义的机器人姿势的vector

```python
from naoqi import ALProxy
posture = ALProxy("ALRobotPosture", "<IP of your robot>", 9559)
posture.getPostureList()
```

```shell
['Crouch', 'LyingBack', 'LyingBelly', 'Sit', 'SitOnChair', 'SitRelax', 'Stand', 'StandInit', 'StandZero']
```

##### `ALRobotPosture.getPosture()`

返回当前预定义姿势的名称。如果当前姿势不在预定义姿势中，则返回"未知"

> **<u>*Return:*</u>** 当前姿势的字符串

```python
from naoqi import ALProxy
posture = ALProxy("ALRobotPosture", "<IP of your robot>", 9559)
posture.getPosture()
```

##### `ALRobotPosture.goToPosture (postureName，speed)`

使机器人进入参数中要求的预定义姿态，可以修改移动的速度。机器人动作是“智能”的:从机器人开始的姿态开始，选择所有应该的步骤到达所要求的姿态。

> <u>***Parameters:***</u>  *postureName*：目标预定义姿势名称字符串
>
> ​                        *speed*：相对速度，大小为0.0-1.0
>
> <u>***Return：***</u> 达到目标姿势后返回True

```python
from naoqi import ALProxy
posture = ALProxy("ALRobotPosture", "<IP of your robot>", 9559)
posture.goToPosture(postureName，speed)
```

##### `ALRobotPostureProxy.applyPosture(postureName,speed)`

设定机器人的预定姿势的所有关节。操纵机器人的动态行为时使用这个命令，如果需要机器人快速达到一个姿势，则需要操作者的帮助。谨慎使用此功能，命令的作用是即时、无“智能”的，如果机器人正坐着，运用此命令要求机器人站起来，则对机器人可能跌到。

> <u>***Parameters:***</u>  *postureName*：目标预定义姿势名称字符串
>
> ​                        *speed*：相对速度，大小为0.0-1.0
>
> <u>***Return：***</u> 达到目标姿势后返回True

```python
from naoqi import ALProxy
posture = ALProxy("ALRobotPosture", "<IP of your robot>", 9559)
posture.goToPosture(postureName,speed)
```

##### `ALRobotPostureProxy.stopMove( )`

停止目前的姿势

```python
from naoqi import ALProxy
posture = ALProxy("ALRobotPosture", "<IP of your robot>", 9559)
posture.stop()
```

##### `ALRobotPostureProxy.getPostureFamily( )`

获取当前姿势所在的分类

> <u>***Return：***</u> 当前姿势所在的类别

##### `ALRobotPostureProxy.getPostureFamilyList( )`

获取所有预定义的姿势分类

> <u>***Return：***</u> 所有预定义的姿势分类的vector

```python
from naoqi import ALProxy
posture = ALProxy("ALRobotPosture", "<IP of your robot>", 9559)
posture.getPosturFamilyeList()
```

```python
['Belly', 'Crouching', 'Left', 'LyingBack', 'LyingBelly', 'LyingLeft', 'LyingRight', 'Right', 'Sitting', 'SittingOnChair', 'Standing', 'Unknown']
```

##### `ALRobotPostureProxy.setMaxTryNumber(maxTryNumber)`

> 设置最大尝试次数
>
> <u>***Parameters:***</u>  *maxTryNumber*：尝试次数，默认值为3

#### ALNavigation模块

##### `ALNavigation.navigateTo(x,y)`

该函数使得Nao机器人导航到距离机器人以x,y描述的相对距离，在此过程中，机器人会自动的计算路径以避免障碍物。机器人可能会执行任何安全动作，以确保不与环境发生碰撞。例如抬头看，停下来重新规划一条新路径。
因此，在导航期间不能运行任何占用头部资源的运动时间线。

与`ALMotion`不同，机器人可以在移动的同时选择自己的路径和速度。机器人越接近障碍物，速度越慢，如果避障变得过于危险，机器人就会像在`ALMotion`中一样停止。` ALNavigationProxy::moveTo`

目标必须离机器人近 3 米，否则将忽略该命令并提示警告。

> <u>***Parameters:***</u>   *x* = 沿 X 轴的距离（以米为单位）
>
> ​						 *y* = 沿 Y 轴的距离（以米为单位）
>
> <u>***Return：***</u> 抵达目的点返回True，找不到目标点或被障碍物停止返回False

```python
from naoqi import ALProxy
nav = ALProxy("ALNavigation", "<IP of your robot>", 9559)
nav.navigateTo(2.0,0.0)
```

##### `ALNavigation.moveAlong(trajectory)`

该函数使得Nao机器人沿某条具体的轨迹前进。

> <u>***Parameters:***</u>   trajectory ： (描述直接轨迹的ALValue类，可以是直接轨迹如[[“Holonomic”, pathXY, finalTheta, finalTime], 也可以是复合轨迹，如 [“Composed”, 多个直接轨迹])
>
> *pathXY* 描述了一个2D轨迹，可以是直接轨迹，也可是复合轨迹[“Composed”, direct paths].
>
> *direct path*可为直线，圆弧：[“Line”, [finalX, finalY]], [“Circle”, [centerX, centerY], spanAngle].
>
> <u>***Return：***</u> 抵达目的点返回True，找不到目标点或被障碍物停止返回False

以下命令使机器人在5秒内向前移动1米，然后在10秒内向后移动1米，且不停止:

```python
from naoqi import ALProxy
nav = ALProxy("ALNavigation", "<IP of your robot>", 9559)
nav.moveAlong(["Composed", ["Holonomic", ["Line", [1.0, 0.0]], 0.0, 5.0], ["Holonomic", ["Line", [-1.0, 0.0]], 0.0, 10.0]])
```

##### `ALNavigation.findFreeZone( )`

需要改动

该函数使得Nao机器人导航到距离机器人以x,y描述的相对距离，在此过程中，机器人会自动的计算路径以避免障碍物。机器人可能会执行任何安全动作，以确保不与环境发生碰撞。例如抬头看，停下来重新规划一条新路径。
因此，在导航期间不能运行任何占用头部资源的运动时间线。

与`ALMotion`不同，机器人可以在移动的同时选择自己的路径和速度。机器人越接近障碍物，速度越慢，如果避障变得过于危险，机器人就会像在`ALMotion`中一样停止。` ALNavigationProxy::moveTo`

目标必须离机器人近 3 米，否则将忽略该命令并提示警告。

> <u>***Parameters:***</u>   *x* = 沿 X 轴的距离（以米为单位）
>
> ​						 *y* = 沿 Y 轴的距离（以米为单位）
>
> <u>***Return：***</u> 抵达目的点返回True，找不到目标点或被障碍物停止返回False

```python
from naoqi import ALProxy
nav = ALProxy("ALNavigation", "<IP of your robot>", 9559)
nav.navigateTo(2.0,0.0)
```


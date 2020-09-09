~

**上海意赋教育公司(ifcreate)**

**修订日期：** *2020-08-26*

**适用环境**：Win10&Mac&Linux；Python2.7

**特征**： 在同一局域网内远程调用Nao机器人的所有C++ API

  	      适用Python建立可以远程工作的机器人app

## 使用ALProxy代理连接

### 获取机器人ip地址

1. 使用网线进行连接后，从电脑端的app中设置机器人连接无线网
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

## 机器人运动

### ALMotion模块

它包括四组主要的控制方法: 关节刚度(电机基本开断)；关节位置(插补、反应控制)；步行(距离和速度控制，世界位置等)；笛卡尔空间中的机器人执行器(逆运动学，全身约束)。ALMotion模块还实现了自碰撞避免、外部碰撞避免、跌落控制、智能调节刚度和诊断效果等功能。ALMotion运行在50Hz(周期20ms)。在ALMotion中，每当您调用一个公共方法来请求一个动作时，就会创建一个“动作任务”来处理该任务。

#### 基本参数

X轴正向机器人的前方，Y轴从右到左，Z轴垂直向上。

- ![ ](https://developer.softbankrobotics.com/sites/default/files/repository/70_html_nao/_images/axis_def.png)

#### 函数方法

ALMotion提供帮助机器人移动的方法。它包含允许您操作关节刚度、关节角度的命令，以及允许您控制行走的更高级别API。

##### 刚度控制API

###### `ALMotion.wakeUp()`

唤醒机器人，打开电机，如果需要，恢复到初始位置。如果机器人已经激活，调用立即返回True。

> 成功返回True, 否则返回False

###### `ALMotion.rest()`

使机器人休眠: 并进入一个放松和安全的位置，并关闭电机。例如，进入蹲姿，并关闭刚度。

###### `ALMotion.setStiffnesses(names, stiffnesses)`

设置一个或多个关节的刚度，这是一个非阻塞调用。

> ***Parameters:***  *names*- 关节名称, “Body”, “JointActuators”, “Joints” or “Actuators”.
>
> ​					   *stiffness* - 在0和1之间的一个或多个刚度参数。

###### `ALMotion.getStiffnesses(names)`

获取一个或多个关节的刚度。

> ***Parameters:***  *names*- 关节名称, “Body”, “JointActuators”, “Joints” or “Actuators”.
>
> ***Return***：*stiffness* - 在0和1之间的一个或多个刚度参数。

##### 关节控制

###### `ALMotion.angleInterpolation (names,angleLists,timeLists,isAbsolute )`

插入一个或多个关节到目标角度或沿时间轨迹。这是一个阻塞调用。

> ***Parameters:***  *names*-关节名称, “Body”, “JointActuators”, “Joints” or “Actuators”.
>
> ​                       *angleLists*-以弧度表示的角、角列表或角列表
>
> ​                       *timeLists* - 时间，时间列表或时间列表，以秒为单位
>
> ​                       *isAbsolute* - 如果为真，运动被描述为绝对角度，否则角度是相对于当前的角度

###### `ALMotion.angleInterpolationWithSpeed(names,targetAngles,maxSpeedFraction)`

插值使一个或多个关节到一个目标角度，使用一个最大关节速度的百分比。每个关节只允许一个目标角。这是一个阻塞调用。

> ***Parameters:***  *names*-关节名称, “Body”, “JointActuators”, “Joints” or “Actuators”.
>
> ​                       targetAngles-以弧度表示的角、角列表或角列表
>
> ​                       *maxSpeedFraction* - 一个百分比

###### `ALMotion.setAngles(names, angles, fractionMaxSpeed)`

设置角度，这是一个非阻塞调用。

> ***Parameters:***  *names*-关节名称, “Body”, “JointActuators”, “Joints” or “Actuators”.
>
> ​                       *targetAngles*-以弧度表示的角、角列表或角列表
>
> ​                       *fractionMaxSpeed*- 占最大速度的百分比

###### `ALMotion.changeAngles(names, angles, fractionMaxSpeed)`

改变角度，这是一个非阻塞调用。

> ***Parameters:***  *names*-关节名称, “Body”, “JointActuators”, “Joints” or “Actuators”.
>
> ​                       *targetAngles*-以弧度表示的角、角列表或角列表
>
> ​                       *fractionMaxSpeed*- 占最大速度的百分比

###### `ALMotion.getAngles(names, useSensors)`

获得所有关节的角度

> ***Parameters:***  *names*-关节名称, “Body”, “JointActuators”, “Joints” or “Actuators”.
>
> ​                       useSensors-如果设置为True， 传感器角度会被返回
>
> ***Return：*** 以弧度为单位的关节角度

##### 运动控制

###### `ALMotion.move(x, y, theta)`

使机器人以给定的速度运动，这是一个非阻塞调用。

> ***Parameters:***  *x*-沿x轴的速度，单位是米每秒。向后运动为负值
>
> ​                       *y*-沿y轴的速度，单位是米每秒。向左移动为正值
>
> ​                       *theta*- 绕z轴的速度，单位是弧度/秒。顺时针旋转为负值。

###### `ALMotion.moveToward(x, y, theta)`

使机器人以给定的归一化速度运动，这是一个非阻塞调用。

> ***Parameters:***  *x*-沿x轴的归一化速度，单位是米每秒。向后运动为负值，+1为最大值
>
> ​                       *y*-沿y轴的归一化速度，单位是米每秒。向左移动为正值，+1为最小值
>
> ​                       *theta*- 绕z轴的归一化速度，单位是弧度/秒。顺时针旋转为负值。

###### `ALMotion.moveTo(x, y, theta)`

使机器人以给定的归一化速度运动，这是一个非阻塞调用。

> ***Parameters:***  *x*-沿x轴的距离，单位是米
>
> ​                       *y*-沿y轴的距离，单位是米
>
> ​                       *theta*- 绕z轴的旋转，单位是弧度， 范围为[-3.1415 to 3.1415]。
>
> ***Return：*** 如果抵达返回True, 被中断则返回False

###### `ALMotion.stopMove()`

停止运动

##### 笛卡尔控制

###### `ALMotion.setPositions(effectorNames, frame, position, fractionMaxSpeed, axisMask)`

随着时间的推移，移动末端执行器到给定的位置和方向。这是一个阻塞调用。

> ***Parameters:***  *effectorNames*- Name or names of effector. Could be: “Torso” or chain name.
>
> ​                       *frame*-任务坐标系
>
> ​                       *position*-  Position6D array (x,y,z,wx,wy,wz) in meters and radians
>
> ​                       *fractionMaxSpeed*- 最大速度的百分比
>
> ​                       *axisMask*- The Axis Mask or Axis Mask list. True for axes that you wish to control. e.g. 7 for position only, 56 for rotation only and 63 for both

```python
#! /usr/bin/env python
# -*- encoding: UTF-8 -*-

"""Example: Use setPositions Method"""

import qi
import argparse
import sys
import time
import motion


def main(session):
    """
    This example uses the setPositions method.
    """
    # Get the services ALMotion & ALRobotPosture.

    motion_service  = session.service("ALMotion")
    posture_service = session.service("ALRobotPosture")

    # Wake up robot
    motion_service.wakeUp()

    # Send robot to Pose Init
    posture_service.goToPosture("StandInit", 0.5)

    # Example showing how to set LArm Position, using a fraction of max speed
    chainName = "LArm"
    frame     = motion.FRAME_TORSO
    useSensor = False

    # Get the current position of the chainName in the same frame
    current = motion_service.getPosition(chainName, frame, useSensor)

    target = [
        current[0] + 0.05,
        current[1] + 0.05,
        current[2] + 0.05,
        current[3] + 0.0,
        current[4] + 0.0,
        current[5] + 0.0]

    fractionMaxSpeed = 0.5
    axisMask         = 7 # just control position

    motion_service.setPositions(chainName, frame, target, fractionMaxSpeed, axisMask)

    time.sleep(1.0)

    # Example showing how to set Torso Position, using a fraction of max speed
    chainName        = "Torso"
    frame            = motion.FRAME_ROBOT
    position         = [0.0, 0.0, 0.25, 0.0, 0.0, 0.0] # Absolute Position
    fractionMaxSpeed = 0.2
    axisMask         = 63
    motion_service.setPositions(chainName, frame, position, fractionMaxSpeed, axisMask)

    time.sleep(4.0)

    # Go to rest position
    motion_service.rest()


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--ip", type=str, default="127.0.0.1",
                        help="Robot IP address. On robot or Local Naoqi: use '127.0.0.1'.")
    parser.add_argument("--port", type=int, default=9559,
                        help="Naoqi port number")

    args = parser.parse_args()
    session = qi.Session()
    try:
        session.connect("tcp://" + args.ip + ":" + str(args.port))
    except RuntimeError:
        print ("Can't connect to Naoqi at ip \"" + args.ip + "\" on port " + str(args.port) +".\n"
               "Please check your script arguments. Run with -h option for help.")
        sys.exit(1)
    main(session)
```

###### `ALMotion.getPositions(name, frame, useSensorValues)`

获取相对于坐标系的位置。x轴为正方向，y轴从右到左，z轴为垂直方向。Position6D的角度约定是Rot_z(wz). rot_y (wy). rot_x (wx)。

> ***Parameters:***  *name*- Name or names of effector. Could be: “Torso” or chain name.
>
> ​                       *frame*-任务坐标系
>
> ​                       *useSensorValues*-  如果为真，传感器值将用于确定位置。

### ALRobotPosture模块

`ALRobotPosture`模块可以让机器人切换到不同的预定义姿势。您可以在`ALRobot.goToPosture`和`ALRobotPosture.applyPosture`之间选择。

如果你想创建一个机器人自动执行的应用程序，请始终选择`ALRobotPosture.goToPosture`。
在操作机器人时，如果你只是想要快速到达姿势的捷径，你可以使用`ALRobotPosture.applyPosture`(你将不得不帮助机器人)。

机器人检测到它当前的姿态，并自动计算从当前姿态到目标姿态的路径，然后执行，在此过程中用户可以选择执行速度的快慢。

#### 预定义姿势

机器人的姿态是其关节和惯性传感器的(独特的)结构。由于姿势是由一组实数(比如浮点数)定义的，所以姿势的数量是无限的。

以下是预先定义的姿势名称列表:

- **Crouch** ,
- **LyingBack** ,
- **LyingBelly** ,
- **Sit** ,
- **SitRelax** ,
- **Stand** ,
- **StandInit** ,
- **StandZero** .

**Posture Family:**

| Posture Family   | Description                          |
| :--------------- | :----------------------------------- |
| “Standing”       | 重量由脚支撑，躯干直立，腿直立。     |
| “Crouching”      | 重量由脚支撑，躯干直立，腿弯曲。     |
| “Sitting”        | 臀部与地面接触，躯干直立。           |
| “SittingOnChair” | 臀部与椅子(10厘米高)接触，躯干直立。 |
| “LyingBelly”     | 躺下，脸朝下。                       |
| “LyingBack”      | 躺下，脸朝上。                       |
| “LyingLeft”      | 躺下，脸朝左。                       |
| “LyingRight”     | 躺下，脸朝右。                       |
| “Belly”          | 脸朝下，躯干抬起。                   |
| “Back”           | 脸朝上，躯干抬起。                   |
| “Left”           | 向左倾斜，手触地。                   |
| “Right”          | 向右倾斜，手触地。                   |

#### 函数方法

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

##### `ALRobotPosture.applyPosture(postureName,speed)`

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

##### `ALRobotPosture.stopMove( )`

停止目前的姿势

```python
from naoqi import ALProxy
posture = ALProxy("ALRobotPosture", "<IP of your robot>", 9559)
posture.stop()
```

##### `ALRobotPosture.getPostureFamily( )`

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

##### `ALRobotPosture.setMaxTryNumber(maxTryNumber)`

> 设置最大尝试次数
>
> <u>***Parameters:***</u>  *maxTryNumber*：尝试次数，默认值为3

### ALTracker模块

ALTracker模块允许机器人用不同的方式(头部、全身、移动等)跟踪不同的目标(红色的球、脸、地标等)。这个模块的主要作用是在目标检测和运动之间建立一个桥梁，使机器人一直注视着摄像机中间的目标。

#### 跟踪模式

**ALTracker** 跟踪模式如下所示：

| 模式          | 描述               | 解释                                                         |
| :------------ | :----------------- | :----------------------------------------------------------- |
| **Head**      | 默认模式（只动头） | 两个头的关节被控制来跟踪目标。<br/>任何控制头部关节的用户命令将优先于跟踪命令。 |
| **WholeBody** | 动整个身体来跟踪   | 机器人自动保持平衡，并调整姿态跟踪目标。                     |
| **Move**      | 移动跟踪           | 机器人移动以保持与目标的固定距离。<br/>在标准配置中，任何移动用户命令将优先于跟踪。 |

#### 跟踪的物体种类

**ALTracker** 可以跟踪以下物体：

| 目标          | 参数                                                   | 解释                                                         |
| :------------ | :----------------------------------------------------- | :----------------------------------------------------------- |
| **RedBall**   | 球直径(米)                                             | 用于计算机器人与球之间的距离。                               |
| **Face**      | 脸宽(米)                                               | 用于计算机器人与人脸之间的距离。                             |
| **LandMark**  | [size, [LandMarkId, ...]]                              | size用于计算机器人与地标之间的距离。<br/>LandMarkId指定要跟踪的地标。 |
| **LandMarks** | [[size, [LandMarkId, ...]], [size, [LandMarkId, ...]]] | 与**LandMark**相同的参数。<br/>一个由地标组成的数组。        |
| **People**    | [peopleId, ...]                                        | 用来追踪指定的人                                             |
| **Sound**     | [distance, confidence]                                 | 利用distance估计声音位置，利用confidence过滤声音位置。       |

#### 物体位置识别坐标系

- ![abc](https://developer.softbankrobotics.com/sites/default/files/repository/70_html_nao/_images/tracker_red_ball.png)

- 跟踪模块识别机器人看到的目标的位置
- 它可以给出目标在目标坐标系中的位置。
- 函数`ALTracker.getTargetPosition`返回活动目标的[x, y, z]位置。

#### 前提条件

根据所选择的`模式`，所使用的身体部位的`Stiffness`必须设置为1.0，否则不能移动。此外，对于`move`和全身`WholeBody`，机器人必须处于站立姿势，准备移动。

要设置刚度，使用motion api `ALMotion.stiffnessInterpolation`。要到达预定义的姿势，请参考: `ALRobotPosture::goToPosture`。

#### 函数方法

##### `ALTracker.getActiveTarget()`

返回`ALTracker::track`检测到的目标名字

> ***Return***：Tracked target name

##### `ALTracker.getAvailableModes()`

返回可行的跟踪模式

> ***Return***：跟踪模式名字列表

##### `ALTracker.getEffector()`

返回可行的跟踪模式，获取当前执行器名称。使用`ALTrackerProxy::setEffector`设置该值。

> ***Return***：当前执行器名称。可能是 `“Arms”, “LArm”, “RArm” or “None”`

##### `ALTracker.getMaximumAcceleration()`

获得头部最大加速度。使用`ALTracker.setMaximumAcceleration`设置此值。

> ***Return***: 以rad.s^-2为单位返回最大加速度

##### `ALTracker.getMaximumVelocity ()`

获得头部最大速度。使用`ALTracker.setMaximumVelocity`来设置这个值。

> ***Return***: 以rad.s^-1为单位返回最大速度

##### `ALTracker.getMode ()`

获取使用用`ALTracker.setMode`定义的当前模式。

> ***Return***: 当前跟踪器预定义模式。参见:跟踪模式。

##### `ALTracker.getRelativePosition ()`

获取移动模式下机器人相对于目标的位置。使用ALTracker.setRelativePosition设置该值。

> ***Return***: 返回:跟踪的最终目标:`[coordX, coordY, coordWz, thresholdX, thresholdY, thresholdWz]`。

##### `ALTracker.registerTarget(TargetName, Param)`

用参数`(RedBall, 红球直径)`注册预定义的目标。如果跟踪正在运行，则订阅相应的提取器和存储最后位置。如果目标已经注册，则只更新参数。

> ***Parameters:***  *Targetname* - 预先定义的目标名字
>
> ​					   *Param* - 目标参数

##### `ALTracker.track(TargetName)`

设定预先设定的目标进行跟踪，并开始跟踪过程。目标名称之前需要注册`ALTracker.registerTarget`。

> ***Parameters:***  *Targetname* - 预先定义的目标名字

##### `ALTracker.setEffector (Effector)`

设置一个末端执行器移动以进行跟踪。追踪器总是用头部, 使用ALTracker.getEffector获取该值。

> ***Parameters:***  *Effector* - 执行器的名字。可以是:" Arms "， " LArm "， " RArm "或" None " 

##### `ALTracker.setMaximumAcceleration (MaxAcceleration)`

设置头部的最大绝对加速度。

> ***Parameters:***  *MaxAcceleration* - 以rad.s^-2为单位的非负最大加速度

##### `ALTracker.setMaximumDistanceDetection (MaxDistance)`

设置目标检测的最大绝对距离。如果到目标的距离大于这里给出的距离，则认为目标丢失了。

> ***Parameters:*** *MaxDistance * - 距离，正，单位为米。

##### `ALTracker.setMaximumVelocity (MaxVelocity )`

设置头的最大绝对速度。

> ***Parameters:*** *MaxVelocity* – 速度，非负，单位为rad.s^-2

##### `ALTracker.setMode (MaxVelocity)`

将跟踪器设置为预定义模式。使用`ALTracker.getMode`获取该值。

> ***Parameters:***  *Mode* - 预先定义的模式

##### `ALTrackerset.RelativePosition (Target)`

在移动模式下，设置机器人相对于目标的位置。 使用`ALTracker.getRelativePosition`来获得这个值。

> ***Parameters:***  *Target* - 跟踪的最终目标:`[coordX, coordY, coordWz, thresholdX, thresholdY, thresholdWz]`

##### `ALTracker.stopTracker()`

停止追踪

#### 脸部追踪实例

```python
#! /usr/bin/env python
# -*- encoding: UTF-8 -*-

"""Example: Use Tracking Module to Track a Face"""

import qi
import argparse
import sys
import time


def main(session, faceSize):
    """
    This example shows how to use ALTracker with face.
    """
    # Get the services ALTracker and ALMotion.

    motion_service = session.service("ALMotion")
    tracker_service = session.service("ALTracker")

    # First, wake up.
    motion_service.wakeUp()

    # Add target to track.
    targetName = "Face"
    faceWidth = faceSize
    tracker_service.registerTarget(targetName, faceWidth)

    # Then, start tracker.
    tracker_service.track(targetName)

    print "ALTracker successfully started, now show your face to robot!"
    print "Use Ctrl+c to stop this script."

    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        print
        print "Interrupted by user"
        print "Stopping..."

    # Stop tracker.
    tracker_service.stopTracker()
    tracker_service.unregisterAllTargets()
    motion_service.rest()

    print "ALTracker stopped."


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--ip", type=str, default="127.0.0.1",
                        help="Robot IP address. On robot or Local Naoqi: use '127.0.0.1'.")
    parser.add_argument("--port", type=int, default=9559,
                        help="Naoqi port number")
    parser.add_argument("--facesize", type=float, default=0.1,
                        help="Face width.")

    args = parser.parse_args()
    session = qi.Session()
    try:
        session.connect("tcp://" + args.ip + ":" + str(args.port))
    except RuntimeError:
        print ("Can't connect to Naoqi at ip \"" + args.ip + "\" on port " + str(args.port) +".\n"
               "Please check your script arguments. Run with -h option for help.")
        sys.exit(1)
    main(session, args.facesize)
```

## 机器人情绪感知

### ALMood模块

ALMood模块能够预测机器前方行人的表情、对机器人的行为目的和机器人周围环境氛围。

使用这项服务，你可以查询以下情绪感知的基本表现形式，获得一组如积极、消极、谨慎等情绪的描述符

连接到ALMood模块的信号和属性，获知机器人前面的人是积极的、消极的还是中立的，用户对机器人是不投入的，半投入的还是完全投入的，周围的环境是平静的还是兴奋的。

#### `ALMood.currentPersonState()`

该函数得到一个描述当前用户的情绪状态的结构体PersonState

> <u>***Returns:***</u>   一个详细描述一个人的情绪的结构体

```python
from naoqi import ALProxy
mood = ALProxy("ALMood", "<IP of your robot>", 9559)
mood.currentPersonState()
```

PersonState结构体的组成如下。其中valence的值域是-1（消极）~1（积极），其余的值的值域是0~1

```python
PersonData =
{
   "valence" : { value, confidence },
   "attention" : { value, confidence },
   "bodyLanguageState" :
   {
     "ease" : { level, confidence }
   },
   "smile" : { value, confidence },
   "expressions" :
   {
    "calm" : { value, confidence },
    "anger" : { value, confidence },
    "joy" : { value, confidence },
    "sorrow" : { value, confidence },
    "laughter" : { value, confidence },
    "excitement" : { value, confidence },
    "surprise" : { value, confidence }
   }
}
```

#### `ALMood.currentPersonStateFromPeoplePerception(ID)`

该函数得到某个ID的用户的情绪状态

> <u>***Parameters:***</u>   ALUserSession的辨识符，具体参考ALUserSession-API
>
> <u>***Returns:***</u>   一个详细描述一个人的情绪的结构体,同上

#### `ALMood.persons()`

该函数得到一个列表，包含在模块内存中的所有用户

> <u>***Returns:***</u>   一个Person结构体包含ALMood检测到的用户的ID及其情绪状态

Person结构体如下：

```python
Person =
{
   "userSessionID" : usid,
   "personState" :
   {
     ...
     *See above*
     ...
   }
}
```

#### `ALMood.ambianceState()`

该函数得到一个表示周围环境状态的结构体AmbianceState

> <u>***Returns:***</u>   一个描述环境状态的结构体AmbianceState

AmbianceState结构体如下，值域为0~1

```python
AmbianceData =
{
   "agitationLevel" : value,
   "calmLevel" : value
}
```

#### `ALMood.getEmotionalReaction()`

获取用户的情感反应。该函数寻找用户的首次情感反应，当出现以下任一情况时返回：

（1）找到一个积极或者消极的情绪反应

（2）超出设定时间

> <u>***Returns:***</u>   检测到的情绪反应

## 机器人语音

### 语音管理

#### ALAnimatedSpeech模块

该模块使机器人以一种有表现力的形式来说话，包括语言和动作等。

原理：模块可以接收用指令标注的文本，文本被分成一个个语句和动作小块，可以自定义每个动作和语句执行的同步性和异步性

指令文本：一个包含语句和动作指令的文本，如

```python
"Hello! ^start(animations/Stand/Gestures/Hey_1) Nice to meet you!"
```

执行了这个文本之后，机器人的执行顺序是：

（1）机器人说“Hello”

（2）同时执行

​			a.执行命令animations/Stand/Gestures/Hey_1

​			b.说“Nice to meet you!”	

如果想继续运行动作，可以在后面加一个**^wait**指令，如

```python
"Hello! ^start(animations/Stand/Gestures/Hey_1) Nice to meet you ^wait(animations/Stand/Gestures/Hey_1)"
```

具体的指令如下：

**说话动作模式**

| 指令                                      | 作用                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| **^mode(** *speaking_movement_mode* **)** | 改变说话的时候的动作，*speaking_movement_mode*可以是“disabled”"random""contextual" |

```python
"Hello. Look I can stop moving ^mode(disabled) and after I can resume moving ^mode(contextual), you see ?"
```

**动态动作**

| 指令                                    | 作用                                   |
| --------------------------------------- | -------------------------------------- |
| **^run(** *animation_full_name* **)**   | 中止语音，进行动态动作，再重新开始语音 |
| **^start(** *animation_full_name* **)** | 开始动作                               |
| **^stop(** *animation_full_name* **)**  | 停止动作                               |
| **^wait(** *animation_full_name* **)**  | 中止语音，等到动作结束再重新开始语音   |

```python
"^start(animations/Stand/Gestures/Enthusiastic_4) Look what I can do while speaking!"
"^start(animations/Stand/Gestures/Enthusiastic_4) Look what I can do while speaking! ^stop(animations/Stand/Gestures/Enthusiastic_4) Now I am animated by the ALSpeakingMovement module"
"^start(animations/Stand/Gestures/Hey_1)Hi, guys! ^wait(animations/Stand/Gestures/Hey_1)"
```

**运行标记动作**

| 指令                                                         | 作用                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **^runTag(** *tag_name* **)** 或 **^runTag(** *path* , *tag_name* **)** | 中止语音→运行有标记的动态动作，如果指定路径，则只有该路径的动作被运行→重新开始语音 |
| **^startTag(** *tag_name* **)** 或 **^runTag(** *path* , *tag_name* **)** | 运行有标记的动态动作，如果指定路径，则只有该路径的动作被运行 |
| **^stopTag(** *tag_name* **)** 或**^stopTag(** *path* , *tag_name* **)** | 停止有标记的动态动作，如果指定路径，则只有该路径的动作被停止 |
| **^waitTag(** *tag_name* **)** 或**^waitTag(** *path* , *tag_name* **)** | 中止语音→等待有标记的动态动作结束，如果指定路径，则只等待该路径指定的动作→重新开始语音 |

```python
"^startTag(me) My name is Nao."
"^startTag(hello) Hello. ^waitTag(hello)"
"^startTag(hello) Hello Paul, nice to meet you. ^stopTag(hello) My name is Nao."
"^startTag(animations/Stand, hello) Hello. ^waitTag(animations/Stand, hello)"

```

**声音**

| 指令                                                         | 作用                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **^runSound(** *soundSet/soundFile* **)**或**^runSound(** *soundSet/soundFile* , *soundVolume* **)** | 中断语音→播放声音→重新启动语音。*soundVolume*是0~100的整数，默认值为100 |
| **^startSound(** *soundSet/soundFile* **)**或**^startSound(** *soundSet/soundFile* , *soundVolume* **)** | 播放声音作为背景音乐。*soundVolume*是0~100的整数，默认值为100 |
| **^stopSound(** *soundSet/soundFile* **)**                   | 停止播放                                                     |
| **^waitSound(** *soundSet/soundFile* **)**                   | 中止语音→等待播放结束→重新启动语音                           |

```python
"^startSound(my_sound_set/my_sound) That's cool."

"^startSound(my_sound_set/my_sound, 50) That's cool. ^waitSound(my_sound_set/my_sound)"

"That's cool. ^runSound(my_sound_set/my_sound) "
```

#### `ALAnimatedSpeech.say(text)`

机器人语音输出给定的语句，并且对内嵌在语句中的动态动作进行演示

> <u>***Parameters:***</u>   一个给定的文本语句，例如“Hello. ^start(animations/Stand/Gestures/Hey_1) My name is John Doe. Nice to meet you!”)

```python
from naoqi import ALProxy
speech = ALProxy("ALAnimatedSpeech", "<IP of your robot>", 9559)
speech.say("Hello! ^start(animations/Stand/Gestures/Hey_1) Nice to meet you!")
```



### ALAudioDevice模块

该模块提供给其它Naoqi模块到NAO的声音输入（麦克风）和声音输出（喇叭）的接口，每个需要处理声音信号的Naoqi模块都要使用该模块来传递信号。

ALAudioDevice可以以下列的格式来表示由麦克风输入的声音：

- 四个通道交织，48000Hz，170ms缓冲（默认）
- 四个通道分选，48000Hz，170ms缓冲
- 一个通道（前，后，左或者右），16000Hz，170ms缓冲

ALAudioDevice可以以下列的帧数来发送数据到喇叭：

- 两个通道交织，16000Hz（亚洲语言默认该值）
- 两个通道交织，22050Hz（非亚洲语言默认该值）
- 两个通道交织，44100Hz
- 两个通道交织，48000Hz

#### `ALAudioDevice.disableEnergyComputation()`

关闭计算每个输入通道的能量

```python
from naoqi import ALProxy
EngCom = ALProxy("ALAudioDevice", "<IP of your robot>", 9559)
EngCom.disableEnergyComputation()
```

#### `ALAudioDevice.enableEnergyComputation()`

使能计算每个输入通道的能量（默认是关闭的），计算结果可以通过以下函数调取：

- `ALAudioDevice.getFrontMicEnergy()`
- `ALAudioDevice.getRearMicEnergy()`
- `ALAudioDevice.getLeftMicEnergy()`
- `ALAudioDevice.getRightMicEnergy()`

#### `ALAudioDevice.flushAudioOutputs()`

清空所有将要发送给扬声器的样本

##### `ALAudioDevice.getFrontMicEnergy()`

返回前方麦克风在170ms缓冲区的平均信号能量，该功能运行前首先要用enableEnergyComputation()使能

> <u>***Returns:***</u>   能量值，范围在0~32768之间

```python
from naoqi import ALProxy
Eng = ALProxy("ALAudioDevice", "<IP of your robot>", 9559)
Eng.enableEnergyComputation()
leftEng = Eng.getFrontMicEnergy()
```

其余麦克风的能量计算同理

##### `ALAudioDevice.getOutputVolume()`

获取系统音量大小

> <u>***Returns:***</u>   音量大小0~100

##### `ALAudioDevice.isAudioOutMuted()`

判断是否声音输出设备被禁止

> <u>***Returns:***</u>   声音被禁止则返回True，否则返回False

##### `ALAudioDevice.muteAudioOut(mute)`

禁止声音设备

> <u>***Parameters:***</u>   True表示禁止，False表示不禁止

##### `ALAudioDevice.playSine(frequency，gain,pan,duration)`

输出一个给定特性的正弦波

> <u>***Parameters:***</u>   
>
> frequency：频率，单位赫兹
>
> gain：音量0~100
>
> pan：立体声像，取-1，0，1
>
> duration：持续时间，单位为秒

##### `ALAudioDevice.sendLocalBufferToOutput(nbOfFrames,buffer)`

一个本地的模块使用该函数发送信号到机器人的扬声器，信号存在十六位的缓存器中，缓存器的大小不超过16384

> <u>**Parameters:**</u>   
>
> nbOfFrames：缓存器中的立体帧数
>
> buffer：缓存器的在内存空间中的地址
>
> <u>**Returns:**</u>   
>
> 操作成功返回True，否则返回False

##### `ALAudioDevice.setClientPreferences(name,sampleRate,channels,deinterleaved)`

模块可以通过这个函数来设定ALAudioDevice传递的信号的格式。如果没有调用这个函数，则默认发送给这个模块的格式是四个交织通道，48000Hz。下列的格式组合是可行的：

- 四通道交织，48000Hz（默认设置）
- 四通道分选，48000Hz
- 一通道（前，后，左或右），16000Hz

> <u>**Parameters:**</u>   
>
> name：模块名字
>
> sampleRte：取样率，16000Hz或48000Hz
>
> channels：通道配置，AL.ALLCHANNELS，AL.FRONTCHANNEL，AL.LEFTCHANNEL或者AL.REARCHANNEL
>
> denterleaved：只有当通道配置为AL.ALLCHANNELS时才有效

##### `ALAudioDevice.setFileAsInput(fileName)`

该函数将特定文件设定为声音输入，声音文件以.wav为后缀，包含48000Hz，16位，四通道交织信号

> <u>**Parameters:**</u>  
>
>  fileName：文件的绝对路径

##### `ALAudioDevice.setOutputVolume(volume)`

该函数设定系统的输入音量，音量可以由ALAudioDevice.getOutputVolume获取

#### `ALAudioDevice.startMicrophonesRecording(fileName)`

该函数将麦克风收集到的信号直接记录在特定的文件中。如果文件后缀为.wav
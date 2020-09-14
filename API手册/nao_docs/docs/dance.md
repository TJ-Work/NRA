## Nao机器人跳舞实例讲解

### 导入相应的库和`naoqi`框架

```python
from naoqi import ALProxy
import motion
import almath
```

### 设置机器人IP

```python
#改为你自己的IP地址
robotIP="192.168.2.115"
```

### 初始化代理

```python
motionProxy  = ALProxy("ALMotion", robotIP, 9559)
postureProxy = ALProxy("ALRobotPosture", robotIP, 9559)
ttsProxy     = ALProxy("ALTextToSpeech", robotIP, 9559)
audioProxy   = ALProxy("ALAudioPlayer", robotIP, 9559)
#isAbsolute variable is used wherever functions require a boolean argument
isAbsolute   = True
```

### CARTESIAN CONTROL

```PYTHON
space      = motion.FRAME_ROBOT
axisMask   = almath.AXIS_MASK_ALL

#isAbsolute variable is used wherever functions require a boolean argument
isAbsolute = False
```

### 舞蹈动作一

主要用到的函数方法为[ALMotion.angelInterpolation](./motion.md#41)方法控制机器人运动：

```python
def move1():
    names      = ["RHipRoll","LHipRoll", "LKneePitch"
                  , "LAnklePitch", "LHipPitch"]
    angleLists = [[0.2] ,[0.2]  ,[1]   ,[-0.5],[-0.5]]
    times      = [[1.0] ,[ 1.0] ,[1.0] ,[1.0] , [1.0]]
    isAbsolute = True
    motionProxy.angleInterpolation(names, angleLists, times, isAbsolute)

    names      = ["RShoulderRoll" ,"RHand","HeadYaw","LWristYaw"]
    angleLists = [[-1.5],[1,0,1,0]        ,[1.5] ,[-1.0]]
    times      = [[2.0] ,[0.5,1.0,1.5,2.0],[2.0] ,[1.0] ]
    isAbsolute = True
    motionProxy.angleInterpolation(names, angleLists, times, isAbsolute)
```
### 舞蹈动作二

```python
def move2():
	names      = ["RHipRoll","LHipRoll" ,"LKneePitch"
                  ,"LAnklePitch" ,"LHipPitch"]
    angleLists = [[0],[0],[0],[0],[0]]
    times      = [[1.0],[ 1.0],[1.0],[1.0],[1.0]]
    isAbsolute = True
    motionProxy.angleInterpolation(names, angleLists, times, isAbsolute)


    motors  = ["RElbowRoll" ,"LElbowRoll","LShoulderRoll" ,"RShoulderPitch"
               ,"RHand", "LHand", "HeadYaw" ]
    angles  = [[0.5],[-0.5] ,[0.8],[-1] ,[1.0]  ,[1.0] , [0.7,0.7] ]
    times   = [[0.5],[0.5]  ,[0.5],[0.5],[0.5]  ,[0.5] , [0.5,1.0] ]
    motionProxy.angleInterpolation(motors, angles, times,isAbsolute)


    motors  = ["RElbowRoll","LElbowRoll","RShoulderRoll" ,"LShoulderPitch"
               ,"RHand", "LHand", "HeadYaw"   ]
    angles  = [[-0.5],[0.5] ,[-0.8] ,[1]    ,[1.0]  ,[1.0] , [-0.7,-0.7] ]
    times   = [[0.5] ,[0.5] ,[0.5]  ,[0.5]  ,[0.5]  ,[0.5] , [0.5,1]     ]
    motionProxy.angleInterpolation(motors, angles, times,isAbsolute)


    motors  = ["RElbowRoll","LElbowRoll","RShoulderPitch","LShoulderPitch"
               ,"RHand" ,"LHand" , "HeadYaw" ]
    angles  = [[0]  ,[0]    ,[-1]   ,[-1]   ,[-1.0]  ,[-1.0]  , [0] ]
    times   = [[0.3],[0.3]  ,[0.3]  ,[0.3]  ,[0.3]   ,[0.3]   , [0.3]]
    motionProxy.angleInterpolation(motors, angles, times,isAbsolute)
```

### 舞蹈动作三

```python
def move3():

    
    Head_motors       = ["HeadPitch","HeadYaw"    ]
    Head_angles       = [[0.5,-0.5,0.5,-0.5,0.5,-0.5,0.5,-0.5,0.5,-0.5]
                         ,[1.0, -1.0]  ]
    Head_times        = [[0.5,1.0,1.5,2.0,2.5,3.0,3.5,4.0,4.5,5.0 ]
                         ,[ 2.5, 5.0]  ]
 
    UpperBody_motors  = ["RElbowRoll" ,"LElbowRoll"
                         ,"LShoulderPitch"  ,"RShoulderPitch"   ]
    UpperBody_angles  = [[0.5],[-0.5],[0.8, -0.8, 0.0, -0.8, 0.8]
                         ,[-0.8, 0.8, 0.0, 0.8, -0.8] ]
    UpperBody_times   = [[1.0],[1.0] ,[1.0,  2.0, 3.0,  4.0, 5.0]
                         ,[ 1.0, 2.0, 3.0, 4.0,  5.0] ]

        
    LowerBody_motors  = ["LKneePitch","RKneePitch","RAnklePitch","LAnklePitch"
                         ,"RHipPitch","LHipPitch"]
    LowerBody_angles  = [[1,0]  ,[1,0]  ,[-0.5,0] ,[-0.5,0]
                         ,[-0.5,0]   ,[-0.5,0]]
    LowerBody_times   = [[3,5]  ,[3,5]  ,[3,5]    ,[3,5]
                         ,[3,5]      ,[3,5]  ]

    #The motornames_names list has the various MOTOR_NAMES which are used
    #The angles list has the respective MOTOR_ANGLES which are used
    #The times list has the recpejtwctive TIMEs at which
    #the motor holds a particular angle value

    motor_names       = Head_motors + UpperBody_motors + LowerBody_motors
    times             = Head_times  + UpperBody_times  + LowerBody_times
    angles            = Head_angles + UpperBody_angles + LowerBody_angles
    isAbsolute = True

     
    motionProxy.angleInterpolation(motor_names, angles, times, isAbsolute)

    names =      ["RHand","RShoulderPitch"]
    angleLists = [[1.0],[-1.0]]
    times      = [[0.5],[0.5]] 
    motionProxy.angleInterpolation(names, angleLists, times, isAbsolute)



    #SET NAO TO INITIAL POSE
    postureProxy.goToPosture("StandInit", 0.5)

```

### 舞蹈动作四

```python
def move4():    

    #LOWER THE TORSO AND MOVE FROM SIDE TO SIDE
    effector   = "Torso"
    path       = [0.0, -0.07, -0.03, 0.0, 0.0, 0.0]
    time       = 1.5                
    motionProxy.positionInterpolation(effector, space, path,axisMask
                                      , time, isAbsolute)

    effector   = "Torso"
    path       = [0.0, 0.07, 0.03, 0.0, 0.0, 0.0]
    time       = 1.5                
    motionProxy.positionInterpolation(effector, space, path,axisMask
                                      , time, isAbsolute)

    effector   = "Torso"
    path       = [0.0, 0.07, -0.03, 0.0, 0.0, 0.0]
    time       = 1.5                
    motionProxy.positionInterpolation(effector, space, path,axisMask
                                      , time, isAbsolute)
```

### 舞蹈动作五

```python
def move5():
        
    Head_motors       = ["HeadPitch" ,"HeadYaw"    ]
    Head_angles       = [[0.5 ]      ,[1.0, -1.0]  ]
    Head_times        = [[0.5 ]      ,[ 2.5, 5.0]  ]

    UpperBody_motors  = ["RElbowRoll"   ,"LElbowRoll"
                         ,"LShoulderPitch"  ,"RShoulderPitch"]
    UpperBody_angles  = [[0.5]  ,[-0.5] ,[0.8,  0.8, 0.8,  0.8, 0]
                         ,[-0.8, -0.8, -0.8, -0.8,  0 ] ]
    UpperBody_times   = [[1.0]  ,[1.0]  ,[1.0,  2.0, 3.0,  4.0, 5.0]
                         ,[ 1.0, 2.0, 3.0, 4.0,  5.0] ]
  
    #The motornames_names list has the various MOTOR_NAMES which are used
    #The angles list has the respective MOTOR_ANGLES which are used
    #The times list has the recpective TIMEs at which
    #the motor holds a particular angle value

    motor_names       = UpperBody_motors + Head_motors
    times             = UpperBody_times  + Head_times
    angles            = UpperBody_angles + Head_angles

    isAbsolute = True
    motionProxy.angleInterpolation(motor_names, angles, times, isAbsolute)
```

### 主函数

主函数中完成整个跳舞的逻辑, 需要将`audioProxy.post.playFile("/home/nao/Ficher.wav")`中的音频文件地址替换为自己的文件地址。远程传输文件可使用scp传输。

```python
def main(robotIP):    
    #ROBOT SPEAKS OUT MESSAGE
    ttsProxy.say("I LIKE TO DANCE!")

    #WE SET THE MOTOR STIFFNESS TO 1 AND BRING NAO
    #TO INITIAL POSTURE BEFORE START OF DANCE
    #PLAY THE MUSIC
    motionProxy.setStiffnesses("Body", 1.0)
    postureProxy.goToPosture("StandZero", 0.5)
    audioProxy.post.playFile("/home/nao/Ficher.wav")



    #ALL THE DANCE MOVE FUNCTIONS ARE CALLED ONE BY ONE
    move1()
    move2()
    move3()    
    move4()
    move5()
    
    #STOP THE SONG
    audioProxy.stopAll()

    #SET NAO TO INITIAL POSE
    postureProxy.goToPosture("StandInit", 0.5)

    #ROBOT SPEAKS OUT MESSAGE
    ttsProxy.say("THANK YOU!")

    #WE SET THE MOTOR STIFFNESS TO 0 AT THE END OF DANCE
    motionProxy.setStiffnesses("Body", 0.0)
    
```


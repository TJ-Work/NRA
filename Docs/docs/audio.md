## 机器人语音

### ALAnimatedSpeech模块

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

该模块提供给其它`Naoqi`模块到NAO的声音输入（麦克风）和声音输出（喇叭）的接口，每个需要处理声音信号的`Naoqi`模块都要使用该模块来传递信号。

`ALAudioDevice`可以以下列的格式来表示由麦克风输入的声音：

- 四个通道交织，48000Hz，170ms缓冲（默认）
- 四个通道分选，48000Hz，170ms缓冲
- 一个通道（前，后，左或者右），16000Hz，170ms缓冲

`ALAudioDevice`可以以下列的帧数来发送数据到喇叭：

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

返回前方麦克风在170ms缓冲区的平均信号能量，该功能运行前首先要用`enableEnergyComputation`()使能

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
> `frequency`：频率，单位赫兹
>
> `gain`：音量0~100
>
> `pan`：立体声像，取-1，0，1
>
> `duration`：持续时间，单位为秒

##### `ALAudioDevice.sendLocalBufferToOutput(nbOfFrames,buffer)`

一个本地的模块使用该函数发送信号到机器人的扬声器，信号存在十六位的缓存器中，缓存器的大小不超过16384

> <u>**Parameters:**</u>   
>
> `nbOfFrames`：缓存器中的立体帧数
>
> `buffer`：缓存器的在内存空间中的地址
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
> `name`：模块名字
>
> `sampleRte`：取样率，16000Hz或48000Hz
>
> `channels`：通道配置，AL.ALLCHANNELS，AL.FRONTCHANNEL，AL.LEFTCHANNEL或者AL.REARCHANNEL
>
> `denterleaved`：只有当通道配置为AL.ALLCHANNELS时才有效

##### `ALAudioDevice.setFileAsInput(fileName)`

该函数将特定文件设定为声音输入，声音文件以.wav为后缀，包含48000Hz，16位，四通道交织信号

> <u>**Parameters:**</u>  
>
> `fileName`：文件的绝对路径

##### `ALAudioDevice.setOutputVolume(volume)`

该函数设定系统的输入音量，音量可以由ALAudioDevice.getOutputVolume获取

#### `ALAudioDevice.startMicrophonesRecording(fileName)`

该函数将麦克风收集到的信号直接记录在特定的文件中。如果文件后缀为.wav

#### **ALAudioPlayer** 模块

`ALAudioPlayer` 为多种音频文件格式及其关联的常用功能（播放，暂停，停止，循环等）提供播放服务。在所有情况下，最终的音频流都将发送到机器人的扬声器。并使用其绝对路径调用单个声音文件(mp3格式文件支持不是很好)：

```python
fileId = aup.post.playFile("/usr/share/naoqi/wav/filename.wav")
```

#### `ALAudioPlayer.loadFile(filename)`

预载文件，但不播放它。当play函数被调用时，预加载文件是一种减少实际启动播放所需时间的方法。加载文件的播放可以通过调用ALAudioPlayer.play开始。 

> **Parameters:**     `fileName`-文件的绝对路径
>
> ***Returns:***     播放任务的ID

#### `ALAudioPlayer.pause (taskID)`

暂停文件的播放，可以通过调用ALAudioPlayer.play开始。 

> **Parameters:**     `taskID` -播放任务的ID

#### `ALAudioPlayer.play(taskID, volume，pan)`

开始播放指定的任务，具有特定的音量和立体声。 

> **Parameters:**     
>
> `taskID` -播放任务的ID
>
> `volume` -  音量[0.0-1.0]
>
> `pan` - 立体声(-1.0(左) - 1.0(右))

#### `ALAudioPlayer.playFile (fileName, volume, pan)`

启动对指定文件的回放, 这个调用相当于依次调用ALAudioPlayer.loadFile和ALAudioPlayer.play。 

> **Parameters:**      
>
> `fileName`-文件的绝对路径
>
> `volume` -  音量[0.0-1.0]
>
> `pan` - 立体声(-1.0(左) - 1.0(右))

#### `ALAudioPlayer.setVolume (taskID, volume)`

设置播放指定任务的音量级别。

> **Parameters:**     
>
> `taskID` -播放任务的ID
>
> `volume` -  音量[0.0-1.0]

#### `ALAudioPlayer.stopAll ()`

停止所有播放文件
## 机器人情绪感知

### ALMood模块

`ALMood`模块能够预测机器前方行人的表情、对机器人的行为目的和机器人周围环境氛围。

使用这项服务，你可以查询以下情绪感知的基本表现形式，获得一组如积极、消极、谨慎等情绪的描述符

连接到`ALMood`模块的信号和属性，获知机器人前面的人是积极的、消极的还是中立的，用户对机器人是不投入的，半投入的还是完全投入的，周围的环境是平静的还是兴奋的。

#### `ALMood.currentPersonState()`

该函数得到一个描述当前用户的情绪状态的结构体`PersonState`

> <u>***Returns:***</u>   一个详细描述一个人的情绪的结构体

```python
from naoqi import ALProxy
mood = ALProxy("ALMood", "<IP of your robot>", 9559)
mood.currentPersonState()
```

`PersonState`结构体的组成如下。其中`valence`的值域是-1（消极）~1（积极），其余的值的值域是0~1

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

> <u>***Parameters:***</u>   `ALUserSession`的辨识符，具体参考`ALUserSession`-API
>
> <u>***Returns:***</u>   一个详细描述一个人的情绪的结构体,同上

#### `ALMood.persons()`

该函数得到一个列表，包含在模块内存中的所有用户

> <u>***Returns:***</u>   一个`Person`结构体包含ALMood检测到的用户的ID及其情绪状态

`Person`结构体如下：

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

该函数得到一个表示周围环境状态的结构体`AmbianceState`

> <u>***Returns:***</u>   一个描述环境状态的结构体`AmbianceState`

`AmbianceState`结构体如下，值域为0~1

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
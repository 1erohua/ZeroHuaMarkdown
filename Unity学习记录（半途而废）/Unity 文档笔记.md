# Unity 文档笔记

*前言：此笔记包含英文*



---

## [Important Classes - Time - Unity 手册](https://docs.unity.cn/cn/current/Manual/TimeFrameManagement.html)

为了更好地了解Unity中的时间序列，在此概括并解释一下Unity中的时间问题

> 在Unity中，物理系统使用固定的时间步长来工作，这对于模拟的准确性和一致性非常重要。在每一帧的开始，Unity会执行尽可能多的固定更新，以便赶上当前时间。你可以在执行顺序图中查看固定更新周期期间发生了什么。
>
> 你也可以在必要时同步执行自己的代码。这通常用于执行自己的物理相关代码，例如对刚体施加力。Unity提供了`FixedUpdate`方法作为一个入口点，让你在每个固定时间步长中执行自己的代码。
>
> `fixedDeltaTime`属性控制Unity固定时间步长循环的间隔，并以秒为单位指定。例如，值为0.01表示每个固定时间步长持续一百分之一秒，因此每秒将有100个固定时间步长。
>
> 如果你的游戏或应用程序的帧率高于每秒固定时间步长的数量，则意味着每帧持续时间小于单个固定时间步长的持续时间。在这种情况下，Unity每帧执行零或一个固定物理更新。例如，如果你的固定时间步长值为0.02，则每秒将有50次固定更新。如果你的游戏或应用程序以每秒60帧的速度运行，则大约十分之一的帧不会有固定更新。
>
> 如果你的游戏或应用程序的帧率低于固定时间步长值，则意味着每帧持续时间大于单个固定时间步长。为了解决这个问题，Unity将在每一帧中执行一次或多次固定更新，以便物理模拟赶上自上一帧以来经过的时间量。例如，如果你的固定时间步长值为0.01，则每秒将有100次固定更新。如果你的应用程序以每秒25帧的速度运行，则直到每一帧都执行四次固定更新。你可能希望这样一个场景，在其中更重要的是建模更精确的物理学而不是高帧率。(概括你妈，句句都是精华，怎么概括)

可以在Unity编辑器中的Time Manager窗口中设置`fixedDeltaTime`的值。打开Time Manager窗口的方法是：在Unity编辑器中，依次点击Edit（编辑）> Project Settings（项目设置）> Time（时间），然后在Fixed Timestep（固定时间步长）字段中输入你想要的值。

你也可以在脚本中通过修改`Time.fixedDeltaTime`属性来设置`fixedDeltaTime`的值。例如，你可以在脚本的Awake或Start方法中添加以下代码来设置`fixedDeltaTime`的值为0.02秒：

```csharp
void Awake()
{
    Time.fixedDeltaTime = 0.02f;
}
```

需要注意的是，修改`fixedDeltaTime`的值会影响物理模拟的准确性和性能。一般来说，较小的`fixedDeltaTime`值会提高物理模拟的准确性，但会更吃设备；而较大的`fixedDeltaTime`值则会降低物理模拟的准确性，但可以不那么吃性能。因此，在设置`fixedDeltaTime`的值时，需要在准确性和性能之间进行权衡。

**以下流程图说明了 Unity 用于计算单个帧中时间的逻辑，以及 [time](https://docs.unity.cn/cn/current/ScriptReference/Time-time.html)、[deltaTime](https://docs.unity.cn/cn/current/ScriptReference/Time-deltaTime.html)、[fixedDeltaTime](https://docs.unity.cn/cn/current/ScriptReference/Time-fixedDeltaTime.html) 和 [maximumDeltaTime](https://docs.unity.cn/cn/current/ScriptReference/Time-maximumDeltaTime.html) 属性如何相互关联。**

![img](https://docs.unity.cn/cn/current/uploads/Main/time-flowchart.png)

be capped to 被限制

`Time.time` 表示从游戏开始到当前帧所经过的时间，而 `deltaTime` 表示两帧之间的时间差。因此，`Time.time` 可以看作是由很多个 `deltaTime` 累加而成的。每一帧，`Time.time` 的值都会增加一个 `deltaTime`，以便计算从游戏开始到当前帧所经过的总时间。

关于后面的那个fixedTime和Update还有FixedUpdate的那个流程图，是New Bing这样解释的

> 当 `fixedTime` 落后于 `Time.time` 至少一个固定时间步长时，意味着游戏中已经经过了一个固定时间步长，此时需要进行一次物理计算。因此，Unity 引擎会调用 `FixedUpdate()` 函数来进行物理计算。
>
> 在 `FixedUpdate()` 函数执行完毕后，Unity 引擎会再次检查 `fixedTime` 是否仍然落后于 `Time.time` 至少一个固定时间步长。如果是，那么就会再次执行 `FixedUpdate()` 函数，直到 `fixedTime` 不再落后于 `Time.time` 至少一个固定时间步长为止。这样做的目的是为了保证物理计算能够以固定的时间步长进行，以便保证物理模拟的准确性。
>
> 总之，当 `fixedTime` 落后于 `Time.time` 至少一个固定时间步长时，Unity 引擎会调用 `FixedUpdate()` 函数来进行物理计算，并且会不断重复这个过程，直到 `fixedTime` 不再落后于 `Time.time` 至少一个固定时间步长为止。

----

这些属性都有各自的脚本 API 文档页面，但是查看它们的描述和输出以及它们之间的关系，有助于理解它们适当的用途。

- [Time.time](https://docs.unity.cn/cn/current/ScriptReference/Time-time.html)
- [Time.unscaledTime](https://docs.unity.cn/cn/current/ScriptReference/Time-unscaledTime.html)
- [Time.deltaTime](https://docs.unity.cn/cn/current/ScriptReference/Time-deltaTime.html)
- [Time.unscaledDeltaTime](https://docs.unity.cn/cn/current/ScriptReference/Time-unscaledDeltaTime.html)
- [Time.smoothDeltaTime](https://docs.unity.cn/cn/current/ScriptReference/Time-smoothDeltaTime.html)
- [Time.timeScale](https://docs.unity.cn/cn/current/ScriptReference/Time-timeScale.html)
- [Time.maximumDeltaTime](https://docs.unity.cn/cn/current/ScriptReference/Time-maximumDeltaTime.html)

`Time.time` 表示自玩家开始游戏以来经过的时间，因此通常会持续稳定地增加。`Time.deltaTime` 表示自上一帧以来经过的时间，因此理想情况下应该保持相对恒定。

这两个值都是应用程序或游戏中经过时间的主观度量。这意味着它们考虑了您应用的任何时间缩放。例如，您可以将 `Time.timeScale` 设置为 0.1 以实现慢动作效果（表示正常播放速度的 10%）。在这种情况下，`Time.time` 的值以“真实”时间的 10% 的速率增加。经过 10 秒钟后，`Time.time` 的值将增加 1。除了在应用程序中减慢或加快时间外，**您还可以将 `Time.timeScale` 设置为零以暂停游戏，在这种情况下，仍然会调用 `Update()` 方法，但 `Time.time` 不会增加，而 `Time.deltaTime` 为零。** 

这些值(`Time.time`、`Time.timeUpdate`、`Time.time`和`Time.deltaTime`)也受到 `Time.maximumDeltaTime` 属性值的限制。这意味着这些属性报告的任何暂停或帧率变化的长度都不会超过`maximumDeltaTime`。例如，如果发生一秒钟的延迟，但 `Time.maximumDeltaTime` 设置为默认值 0.333，则 `Time.time` 只会增加 0.333，而 `Time.deltaTime` 等于 0.333，尽管实际上在现实世界中经过了更多时间。**注意，`Time.maximumDeltaTime`并不能改变帧与帧之间实际经过的时间，它只能限制`Time.deltaTime`报告的值。**

**这些属性的未缩放版本（`Time.unscaledTime` 和 `Time.unscaledDeltaTime`）忽略了这些主观变化和限制**，并在两种情况下都报告实际经过的时间。这对于任何“即使在游戏以慢动作播放时也应以固定速度响应”的内容都很有用。一个例子是 UI 交互动画。

**下表显示了一个例子**，其中 16 帧依次经过，其中一半经过一个大延迟，在单个帧上发生。这些数字说明了 Time 类属性如何报告和响应帧率的大变化。（第8帧发生大延迟）

| Frame     | unscaledTime  | time          | unscaledDeltaTime | deltaTime     | smoothDeltaTime |
| :-------- | :------------ | :------------ | :---------------- | :------------ | :-------------- |
| 1         | 0.000         | 0.000         | 0.018             | 0.018         | 0.018           |
| 2         | 0.018         | 0.018         | 0.018             | 0.018         | 0.018           |
| 3         | 0.036         | 0.036         | 0.018             | 0.018         | 0.018           |
| 4         | 0.054         | 0.054         | 0.018             | 0.018         | 0.018           |
| 5         | 0.071         | 0.071         | 0.017             | 0.017         | 0.018           |
| 6         | 0.089         | 0.089         | 0.018             | 0.018         | 0.018           |
| 7         | 0.107         | 0.107         | 0.018             | 0.018         | 0.018           |
| 8 (**a**) | 1.123 (**b**) | 0.440 (**c**) | 1.016 (**d**)     | 0.333 (**e**) | 0.081 (**f**)   |
| 9         | 1.140         | 0.457         | 0.017             | 0.017         | 0.066           |
| 10        | 1.157         | 0.474         | 0.017             | 0.017         | 0.056           |
| 11        | 1.175         | 0.492         | 0.018             | 0.018         | 0.049           |
| 12        | 1.193         | 0.510         | 0.018             | 0.018         | 0.042           |
| 13        | 1.211         | 0.528         | 0.018             | 0.018         | 0.038           |
| 14        | 1.229         | 0.546         | 0.018             | 0.018         | 0.034           |
| 15        | 1.247         | 0.564         | 0.018             | 0.018         | 0.031           |
| 16        | 1.265         | 0.582         | 0.018             | 0.018         | 0.028           |

- **在第8帧上，尽管实际经过了一秒钟的时间，但deltaTime仅报告了0.333秒。这是因为deltaTime的值被限制在maximumDeltaTime值内。**maximumDeltaTime是Unity中的一个变量，用于限制deltaTime报告的值和增加当前time的数量。当发生大于maximumDeltaTime值的帧延迟时，Unity会限制deltaTime报告的值，并增加当前time的数量。这样做的目的是避免在帧速率峰值期间发生不良副作用(unscaledDeltaTime不受主管变化和限制)

![img](https://docs.unity.cn/cn/current/uploads/Main/time-delta-unscaled.png)

- **在第8帧上，unscaledTime增加了大约一秒钟，因为已经添加了真实（未限制）的值，而time仅增加了较小的限制值。**time值并没有赶上实际经过的时间，而是表现得好像延迟只持续了maximumDeltaTime。（(unscaledTime不受主管变化和限制)）

![img](https://docs.unity.cn/cn/current/uploads/Main/time-and-unscaled.png)

- Time.smoothDeltaTime属性报告了近期deltaTime值的近似值，**所有变化都根据算法平滑处理。这是另一种避免在运动或其他基于时间的计算中出现不合意的大步长或波动的技术。特别是那些低于maximumDeltaTime设置的阈值的波动**。**平滑算法无法预测未来的变化，但它逐渐调整其报告的值**，以平滑近期经过的delta time值的变化，使平均报告时间仍然大致等于实际经过的时间。（Time.smoothDeltaTime和Time.deltaTime都不会对实际帧数造成影响。它们只是用来控制游戏世界中物体的运动和动画。）

![img](https://docs.unity.cn/cn/current/uploads/Main/time-delta-smoothed.png)

### Unity中时间变化和物理学系统

假设您正在玩一个游戏，游戏中有一个物理对象在空中运动。在正常情况下，物理引擎会在每一帧之间模拟物理对象的运动，使其看起来像是在实时运动。

现在，假设在某一帧，游戏需要执行一些重型计算，导致这一帧的处理时间超过了maximumDeltaTime。在这种情况下，物理引擎不会尝试模拟超出maximumDeltaTime的任何时间，而是暂停物理模拟，等待帧处理完成。这意味着，在这一帧之间，物理对象不会像通常那样移动。**（*假设maximumDeltaTime设置为0.1秒。如果在某一帧，帧处理需要0.2秒才能完成，那么物理引擎会检测到这一点，并模拟maximumDeltaTime的那0.1秒，然后停止模拟。*）**

一旦帧更新完成，物理就会恢复，好像自停止以来没有经过任何时间。这意味着，在下一帧之间，物理对象将继续按照正常速度移动，就好像它从未停止过一样。

这样做的目的是避免物理系统落后太远，导致需要大量步骤才能赶上当前时间。虽然这可能会导致物理对象的运动稍微减慢，但这种减慢通常不太明显，并且通常是对游戏性能的可接受折衷。

 ### Unity中的时间尺度

Unity中的时间缩放属性（Time Scale），它可以控制游戏时间相对于现实时间的流逝速度。

如果将时间缩放设置为1.0，则游戏时间与现实时间相匹配。如果将其设置为2.0，则游戏时间将以两倍于现实时间的速度流逝，而设置为0.5则会使游戏速度减半。设置为零则会使游戏时间完全停止。

请注意，时间缩放并不会减慢执行速度，而是改变了**报告给Update和FixedUpdate函数的时间步长（通过Time.deltaTime和Time.fixedDeltaTime）**。您可以使用ScriptRef:Time-timeScale属性从脚本中设置全局时间缩放值。

> The [Time](https://docs.unity.cn/cn/current/Manual/class-TimeManager.html) window has a property to let you set the time scale globally **but it’s usually more useful to set the value from a script using the [ScriptRef:Time-timeScale](https://docs.unity.cn/cn/current/ScriptReference/Time-timeScale.html) property**.

- 对于Update来说，它是每帧调用一次，**修改Time.deltaTime的值不会影响Update函数的调用**。Update函数仍然会在每一帧被调用，但是由于Time.deltaTime的值改变了，所以基于时间的计算也会受到影响。
- 但是**修改Time.fixedDeltaTime会影响FixedUpdate函数的调用**，FixedUpdate函数以固定的时间间隔被调用，这个间隔由Time.fixedDeltaTime确定。如果您更改了Time.fixedDeltaTime的值，则FixedUpdate函数调用之间的时间间隔也会相应更改。(注意：修改Time.scaleTime会影响FixedUpdate的调用，但不会影响Time.fixedDeltaTime的值)
- - Update函数不受此影响，但是Time.deltaTime会受到此影响（成正比）                                                                   
  - FixedUpdate的调用会受到它的影响，即调用频率成正比；但fixedDeltaTime不会因此受到影响
  - 根本原因在于，Time.scaleTime影响了Time.time，而FixedUpdate是根据Time.time的流逝进行调用的

### Unity中的捕获帧率（Capture Framerate）

> A special case of time management is where you want to record gameplay as a video. Since the task of saving screen images takes considerable time, the game’s normal frame rate is reduced, and the video doesn’t reflect the game’s true performance. To improve the video’s appearance, use the Capture Framerate property. The property’s default value is 0, for unrecorded gameplay. For recording. When you set the property’s value to anything other than zero, game time is slowed and the frame updates are issued at precise regular intervals. The interval between frames is equal to , so if you set the value to 5.0 then updates occur every fifth of a second. With the demands on frame rate effectively reduced, you have time in the Update function to save screenshots or take other actions:1 / Time.captureFramerate

这段话讲的是Unity中的“捕获帧率”（Capture Framerate）属性，它用于在录制游戏视频时改善视频的外观。由于保存屏幕图像需要花费相当长的时间，因此游戏的正常帧率会降低，视频无法反映游戏的真实性能。

使用“捕获帧率”属性(即`Time.captureFramerate`)可以改善视频的外观。该属性的默认值为0，表示未录制游戏。当您将该属性的值设置为非零值时，游戏时间会减慢，并且帧更新会以精确的固定间隔发出。**帧之间的间隔等于1 / Time.captureFramerate**，因此如果您将该值设置为5.0，则每五分之一秒更新一次。由于对帧率的要求有效降低，您可以在Update函数中保存屏幕截图或执行其他操作。

为了进一步了解这个捕获帧率属性(`Time.captureFramerate`)，我们问了bing，假如想让视频录制为60帧

> 如果**您希望录制的视频以60帧每秒的速度呈现，那么您应该将Time.captureFramerate设置为60**。这样，帧之间的间隔将等于1 / 60 = 0.01667秒，即每秒更新60次。
>
> 您可以在脚本中这样设置Time.captureFramerate的值：
>
> ```C#
> void Start()
> {
>     Time.captureFramerate = 60;
> }
> 
> ```
>
> 

**关于这为什么会让游戏更难玩，bing是这么说的**

> 使用Time.captureFramerate属性可能会使游戏更难玩，因为它会减慢游戏时间，并且帧更新会以精确的固定间隔发出。这意味着游戏的响应速度可能会降低，从而影响游戏的可玩性。
>
> 例如，如果您将Time.captureFramerate设置为一个较低的值，则游戏时间会减慢，帧更新间隔会增加。这可能会导致游戏画面卡顿，操作反应迟钝，从而影响游戏体验。
>
> 当然，这并不是绝对的。您可以通过尝试不同的Time.captureFramerate值来找到一个既能改善视频外观，又不会对游戏性能产生太大影响的平衡点。
>
> 希望这能帮助您理解为什么使用Time.captureFramerate属性可能会使游戏更难玩。

----



 ##   [Unity - Manual: Variables and the Inspector](https://docs.unity.cn/2021.3/Documentation/Manual/VariablesAndTheInspector.html)

- A script makes its connection with the internal workings of Unity by implementing a class which derives from the built-in class called MonoBehaviour.

- The class name and file name must be the same to enable the script component to be attached to a GameObject.

- The main things to note, however, are the two functions defined inside the class. **The Update function is the place to put code that will handle the frame update for the GameObject. This might include movement, triggering actions and responding to user input, basically anything that needs to be handled over time during gameplay.** To enable the Update function to do its work, it is often useful to be able to set up variables, read preferences and make connections with other GameObjects before any game action takes place. **The Start function will be called by Unity before gameplay begins **(ie, before the Update function is called for the first time) and is an ideal place to do any initialization.

- Note to experienced programmers: you may be surprised that **initialization of an object is not done using a constructor function(构造函数)**. This is because the construction of objects is handled by the editor and does not take place at the start of gameplay as you might expect. If you attempt to define a constructor for a script component, it will interfere（干扰） with the normal operation of Unity and can cause major problems with the project.

- Once attached, the script will start working when you press Play and run the game. You can check this by adding the following code in the Start function:（There, we learn a new method "Debug.Log(string)"）

  Debug.Log is a simple command that just prints a message to Unity’s console output. If you press Play now, you should see the message at the bottom of the main Unity editor window and in the Console window (menu: Window > General > Console).

  ```c#
  // Use this for initialization 
  void Start ()
  { 
      Debug.Log("I am alive!");
  }
  ```

- In C#, the simplest way to see a variable in the Inspector is to declare it as public. An alternative method is to use SerializeField(This is an attribute). Conversely, you can use HideInInspector to prevent a public variable from being displayed in the Inspector.

- Serialize是序列化，它是一个把数据对象转换成二进制流保存为本地文件的过程，方便将对象从一个地方移动到另一个地方，使游戏数据不容易被直接篡改

- When Unity serializes your scripts, it only serializes public fields. If you also want Unity to serialize your private fields you can add the SerializeField attribute to those fields.

  ```C#
  	[SerializeField]
      private Rigidbody rb;
  ```

- Unity serializes all your script components, reloads the new assemblies, and recreates your script components from the serialized versions. This serialization is done with an internal Unity serialization system; not with .NET's serialization functionality.

  The serialization system can do the following:

  - CAN serialize public non-static fields (of serializable types)
  - CAN serialize nonpublic non-static fields marked with the [SerializeField](https://docs.unity.cn/ScriptReference/SerializeField.html) attribute.
  - CANNOT serialize static fields.
  - CANNOT serialize properties.

- Unity can serialize fields of the following types:
  - All classes inheriting from UnityEngine.Object, for example GameObject, Component, MonoBehaviour, Texture2D, AnimationClip.
  - All basic data types, such as int, string, float, bool.
  - Some built-in types, such as Vector2, Vector3, Vector4, Quaternion, Matrix4x4, Color, Rect, LayerMask.
  - Arrays of a serializable type
  - Lists of a serializable type
  - Enums
  - Structs

-  If you put one element in a list (or array) twice, when the list gets serialized, you'll get two copies of that element, instead of one copy being in the new list twice.（实际上就是说，你会得到两份拷贝而不是一个数据的两个引用）

- If you want to serialize a custom Struct field, you must give the Struct the [System.Serializable] attribute.

- **Hint:** Unity won't serialize Dictionary, however you could store a List<> for keys and a List<> for values, and sew them up in a non serialized dictionary on Awake(). This doesn't solve the problem of when you want to modify the dictionary and have it "saved" back, but it is a handy trick in a lot of other cases.（字典不能被序列化，但你可以在脚本里面使用字典的）

- 你可以使用一些方法来绕过这个限制，比如：

  - [使用List模拟字典的键值对，在Awake或Start方法中将键值复制到字典中](https://zhuanlan.zhihu.com/p/397879702)[3](https://zhuanlan.zhihu.com/p/397879702)。
  - [使用[**SerializableDictionary**\]插件，它可以让你自定义你需要序列化的字典类型，并在Inspector窗口中显示](https://blog.csdn.net/JianZuoGuang/article/details/102536605)[4](https://blog.csdn.net/JianZuoGuang/article/details/102536605)。
  - [使用[**OdinSerializer**\]插件，它可以让你序列化任何类型的数据，并提供了更多的功能和优化](https://kuroha.vip/unity/unity_jsonutility.html)[5](https://kuroha.vip/unity/unity_jsonutility.html)。

- cnblog关于序列化的解释：

  序列化 (Serialization)是将对象的状态信息转换为可以存储或传输的形式的过程。在序列化期间，对象将其当前状态写入到临时或持久性存储区。以后，可以通过从存储区中读取或反序列化对象的状态，重新创建该对象。

  序列化使其他代码可以查看或修改，那些不序列化便无法访问的对象实例数据。

- [Unity - Scripting API: SerializeField](https://docs.unity.cn/ScriptReference/SerializeField.html)

- Serialization is the automatic process of transforming data structures or object states into a format that Unity can store and reconstruct later.(序列化的定义)
- The Inspector window does not communicate with the Unity Scripting API when it displays the values of a field. If you use properties in your script, any of the property getters and setters are never called when you view or change values in the Inspector windows as Unity serializes the Inspector window fields directly.

- When you change and save a script, Unity reloads all the currently loaded script data. It first stores all serializable variables in all loaded scripts, and after loading the scripts restores them. All data that is not serializable is lost after the script is reloaded.（注意保存和恢复是两个概念）This affects all Editor windows, as well as all MonoBehaviours in the project. Unlike other cases of serialization in Unity, private fields are serialized by default when reloading, even if they don’t have the ‘SerializeField’ attribute.

- **HideInInspector**: Makes a variable not show up in the inspector but be serialized.

  Use:

  ```c#
  using UnityEngine;
  
  public class Example : MonoBehaviour
  {
      // Make the variable p not show up in the inspector
      // but be serialized.
      [HideInInspector]
      int p = 5;
  }
  ```

  Unity - Scripting API: HideInInspector](https://docs.unity.cn/2021.3/Documentation/ScriptReference/HideInInspector.html)

- Unity will actually let you change the value of a script’s variables while the game is running. This is very useful for seeing the effects of changes directly without having to stop and restart. **When gameplay ends, the values of the variables will be reset to whatever they were before you pressed Play**. This ensures that you are free to tweak your object’s settings without fear of doing any permanent damage.

---

## [Unity - Manual: Instantiating Prefabs at run time](https://docs.unity.cn/2021.3/Documentation/Manual/InstantiatingPrefabs.html)

Compared with creating GameObjects from scratch using code, instantiating Prefabs using code has many advantages because you can:

- Instantiate a Prefab using one line of code. Creating equivalent GameObjects from scratch requires many more lines of code.
- Set up, test, and modify the Prefab quickly and easily using the **Scene**,**view, Hierarchy** and **Inspector**
- Change which Prefab is instantiated without changing the code. You can make a simple rocket into a super-charged rocket, without any code changes.

You can drag a different Prefab into the **My Prefab** field in the Inspector to change which Prefab is instantiated, without having to change the script.

Because this first example is very simple, it may not seem to provide any advantage over(超过) just placing a Prefab into the Scene yourself. However, being able to instantiate Prefabs using code provides you with powerful abilities to dynamically（动态地） create complex configurations of GameObjects while your game or app is running, as shown in the following examples.

学点英语：

replace sth with sth： In this scenario, the example script deletes and replaces the complete, operational robot Prefab with a wrecked robot Prefab. 

### Basics of instantiating a Prefab

详见请看语法笔记，这只讲一些我遇到的东西

讲一讲**Input Manager**

> [Unity - Manual: Input Manager](https://docs.unity.cn/2021.3/Documentation/Manual/class-InputManager.html)

*to access it, from Unity’s main menu, go to **Edit > Project Settings**, then select **Input Manager** from the navigation on the right.*

> The **Input Manager** uses the following types of controls:
>
> - **Key** refers to any key on a physical keyboard, such as W, Shift, or the space bar.
> - **Button** refers to any button on a physical controller (for example, gamepads), such as the X button on a remote control.
> - A **virtual axis** (plural: **axes**) is mapped to a control, such as a button or a key. When the user activates the control, the axis receives a value in the range of [–1..1]. You can use this value in your **scripts**
>   

cnm，map怎么还有映射的意思

- 概念一，Physical Keys

  > The Physical keys option allows you to map key codes to the physical keyboard layout, rather than to the language-specific layout that may vary between users in different regions.
  >
  > For example, on some keyboards the first row of letters reads “QWERTY”, and on others it reads “AZERTY”. This means if you scripted specific controls to use the well known “WASD” keys for movement, they would not be in the correct physical arrangement (like the arrow-key arrangement) on an AZERTY-layout keyboard.
  >
  > With Physical Keys enabled, Unity uses a generic ANSI/ISO “Qwerty” layout to represent the **physical location** of the keys regardless of the user’s actual layout. This means if you specify the “Q” key, it will always be the left-most letter on the first row of letter keys, even if the user’s keyboard has a different letter in that position.

​		简单来讲就是由于不同地方的键盘是不一样的，咱们就用做这个功能，不以字母来，而是以按键所在键盘位		置来

- virtual axes

  下面这个就是一些属性，没错就是edit>project settings到Input Manager的解释

  > | **Property**                                                 | **Function**                                                 |
  > | :----------------------------------------------------------- | :----------------------------------------------------------- |
  > | **Name**                                                     | Axis name. You can use this to access the axis from scripts. |
  > | **Descriptive Name, Descriptive Negative Name**（Descriptive弃用的） | These values are deprecated and do not work. Previously, they displayed for the user on the Rebind Controls screen at startup, but this screen has also been deprecated. |
  > | **Negative Button, Positive Button**（Negative反向，Positive正向） | The controls to push the axis in the negative and positive direction respectively. These can be keys on a keyboard, or buttons on a joystick or mouse. |
  > | **Alt Negative Button, Alt Positive Button**                 | Alternative controls to push the axis in the negative and positive direction respectively. |
  > | **Gravity**（重力）                                          | Speed in units per second that the axis falls toward neutral when no input is present.（用人话来说，当你不按下按钮的时候，该方向上的衰减程度；gravity最大为1000，即此时） |
  > | **Dead**                                                     | How far the user needs to move an analog stick before your application registers the movement. At runtime, input from all analog devices that falls within this range will be considered null.（指游戏手柄或其他输入设备的摇杆或轴的中心位置周围的区域。在这个区域内，输入值被认为是零，即使实际上输入设备的轴并没有完全回到中心位置。Deadzone的目的是为了减少输入设备的误差，使玩家可以更容易地控制游戏中的角色或物体。） |
  > | **Sensitivity**（灵敏度）                                    | Speed in units per second that the axis will move toward the target value. This is for digital devices only.（例如，如果Sensitivity的值为1，则输入设备的轴的完整范围将映射到游戏中的完整范围。如果Sensitivity的值为2，则输入设备的轴的完整范围将映射到游戏中的一半范围。这意味着，如果玩家将输入设备的轴移动到其完整范围的一半位置，则游戏中的角色或物体将只移动到其完整范围的四分之一位置） |
  > | **Snap**                                                     | If enabled, the axis value will reset to zero when pressing a button that corresponds to the opposite direction.（这个我解释不明白，不知道）（这是通义千问的解释：根据 Unity 的文档，当启用 InputManager 的 Snap 功能时，如果您按下了某个按钮，那么系统将会自动检测该按钮所代表的方向，并将轴心位置重置为零。这意味着，如果您按下了一个向左的按钮，那么系统将会将轴心位置重置到左侧，以便您可以通过移动屏幕来实现向左的移动。同样地，如果您按下了一个向右的按钮，那么系统将会将轴心位置重置到右侧，以便您可以通过移动屏幕来实现向右的移动。 因此，当您使用 InputManager 的 Snap 功能时，您可以通过移动屏幕来实现小球的各种移动方向和轨迹，而不仅仅是简单地让小球随机地左右移动。不过需要注意的是，在某些情况下，例如当您需要实现小球的碰撞检测或者其他一些特殊的功能时，您需要在代码中手动控制小球的位置和方向，而不是完全依赖于 InputManager 的 Snap 功能。） |
  > | **Type**                                                     | The type of input that controls the axis. Select from these values:  - Key or Mouse button - Mouse Movement - Joystick Axis |
  > | **Axis**                                                     | The axis of a connected device that controls this axis.      |
  > | **JoyNum**                                                   | The connected Joystick that controls this axis. You can select a specific joystick, or query input from all joysticks. |

(这个英语太简单了，我就不讲了)![image-20230520112425139](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234500.png)

---

### 为了说明在运行时实例化预制件的优势，下面介绍了预制件非常有用的几种基本情况：

#### 1. 通过在不同位置（例如在网格或圆形结构中）多次复制单个预制件来构建一个结构。

> 以下示例将创建一堵墙的实例。
>
> 要尝试该示例，请创建以下脚本，将其命名为 **Wall**，并将其放置在场景中的空游戏对象上。
>
> ```c#
> using UnityEngine;
> public class Wall : MonoBehaviour
> {
>    public GameObject block;
>    public int width = 10;
>    public int height = 4;
>   
>    void Start()
>    {
>        for (int y=0; y<height; ++y)
>        {
>            for (int x=0; x<width; ++x)
>            {
>                Instantiate(block, new Vector3(x,y,0), Quaternion.identity);
>            }
>        }       
>    }
> }
> ```

下面是创建圆形形式的实例化部件（使用三角函数的办法）

> ```C#
> using UnityEngine;
> public class CircleFormation : MonoBehaviour
> {
>    // 以圆形形式实例化预制件
>    public GameObject prefab;
>    public int numberOfObjects = 20;
>    public float radius = 5f;
>    void Start()
>    {
>        for (int i = 0; i < numberOfObjects; i++)
>        {
>            //2Π平均分为n个物体的角度，比如有6个物体，那么每个物体的角度分别是6等分的360°，即60，120，180，210，240，270···注意这里是弧度制
>            float angle = i * Mathf.PI * 2 / numberOfObjects;
>            
>            //利用三角函数计算的坐标就一定是原周围的坐标
>            float x = Mathf.Cos(angle) * radius;
>            float z = Mathf.Sin(angle) * radius;
>            //这个是计算相对于主元件的坐标，妙啊
>            Vector3 pos = transform.position + new Vector3(x, 0, z);
>            
>            //这个没看懂
>            //Mathf.Rad2Deg换算成度
>            float angleDegrees = -angle*Mathf.Rad2Deg;
>            //返回一个Quaternion，它围绕 z 轴旋转 z 度、围绕 x 轴旋转 x 度、围绕 y 轴旋转 y 度（按该顺序应用）。（因为它是角度制的）
>            Quaternion rot = Quaternion.Euler(0, angleDegrees, 0);
>            Instantiate(prefab, pos, rot);
>        }
>    }
> }
> ```
>
> 



####  2. 从发射器发射飞弹预制件。飞弹预制件可能是一个复杂的配置，其中包含**网格**、**刚体**、**碰撞体**、**音频源**、**动态光源**以及一个具有自己的轨迹**粒子系统**的子游戏对象。

我是真他妈吐了，我的首先就没有爆炸效果，**

而且最郁闷的是，它一接触就消失了，我属实是没绷住

不过后面我就把预制件的例子效果改为生成一个立方体，笑死我了

> ### 实例化飞弹和爆炸
>
> 在此情况中：
>
> 1.玩家按下发射按钮时，“Launcher”游戏对象将实例化一个飞弹预制件。该预制件包含一个网格、一个刚体和一个碰撞体，因此它可以在空中飞行并检测何时发生碰撞。
>
> 2.飞弹与物体碰撞，然后实例化爆炸预制件。爆炸预制件包含粒子系统效果，以及一个对周围游戏对象施力的脚本。
>
> 按照与上面的 Block 预制件相同的方式，无论飞弹预制件有多复杂，仅用一行代码即可实例化飞弹。在实例化预制件之后，还可以修改实例化的游戏对象的任何属性。例如，可以设置飞弹的刚体的速度。
>
> 除了易于使用之外，还可以稍后修改预制件，而无需改动代码。因此，如果飞弹为火箭，那么稍后可以添加粒子系统，从而产生云迹。完成此操作后，所有实例化的火箭都会具有粒子轨迹。
>
> 以下脚本显示了如何使用 **Instantiate()** 函数来发射飞弹。
>
> ```
> using UnityEngine;
> public class FireProjectile : MonoBehaviour
> {
>     public Rigidbody projectile;
>     public float speed = 4;
>     void Update()
>     {
>         if (Input.GetButtonDown("Fire1"))
>         {
>             Rigidbody p = Instantiate(projectile, transform.position, transform.rotation);
>             p.velocity = transform.forward * speed;
>         }
>     }
> }
> ```
>
> 在代码中，预制件变量类型是刚体，而不是游戏对象。这样有两个有用的效果：
>
> 1.只能为此变量分配具有刚体组件的游戏对象。这很有用，因为它有助于确保您为变量分配了正确的游戏对象。
>
> 2.Instantiate 方法返回对新实例上的刚体组件的引用。这很有用，因为这样可以轻松地在实例化刚体之后立即设置刚体的速度。
>
> 生成公共预制件变量时，变量类型可以是游戏对象，也可以是任何有效的组件类型（内置的 Unity 组件或您自己的 MonoBehaviour 脚本之一）。
>
> 对于游戏对象类型的变量，可以将任何游戏对象分配给该变量，并且 Instantiate 函数将返回对新游戏对象实例的引用。
>
> 对于组件类型变量（例如刚体、碰撞体和光源），只能将该组件类型的游戏对象分配给变量，并且 Instantiate 函数将返回新游戏对象实例上对该特定组件的引用。
>
> 以下脚本（放置在飞弹预制件上）执行以下操作：在飞弹碰撞物体后，在飞弹的当前位置实例化爆炸，然后删除飞弹游戏对象。
>
> ```
> using UnityEngine;
> public class Projectile : MonoBehaviour
> {
>    public GameObject explosion;
>    void OnCollisionEnter()
>    {
>        Instantiate(explosion,transform.position,transform.rotation);
>        Destroy(gameObject);
>    }
> }
> ```

#### 3. 车辆、建筑物或角色（例如机器人）分解成许多部分。在这种情况中，示例脚本将删除完整正常的机器人预制件，并替换为残骸机器人预制件。这种残骸预制件由机器人的单独破碎部分组成，每个部分都具有自己的刚体和粒子系统。通过这种方法，只需一行代码即可将机器人炸成许多碎片，这种情况下会将原始游戏对象替换为一个残骸预制件。

---

## Order of execution for event functions

> [Unity - Manual: Order of execution for event functions](https://docs.unity.cn/2021.3/Documentation/Manual/ExecutionOrder.html)

在Unity脚本中，有许多事件函数会按照预定的顺序执行。这个顺序被称为“事件函数的执行顺序”（Order of execution for event functions）

下图概括了 Unity 如何在脚本的生命周期内对事件函数进行排序以及重复执行这些事件函数。

![img](https://docs.unity.cn/cn/current/uploads/Main/monobehaviour_flowchart.svg)

#### 加载第一个场景时

场景开始时将调用以下函数（为场景中的每个对象调用一次）。

- **Awake：**始终在**任何 Start 函数之前**并在实例化预制件之后调用此函数。（**如果游戏对象在启动期间处于非活动状态，则在激活之后才会调用 Awake。**）*在脚本实例的生命周期内，Unity仅调用一次`Awake`*   [Unity - Scripting API: MonoBehaviour.Awake()](https://docs.unity.cn/2021.3/Documentation/ScriptReference/MonoBehaviour.Awake.html)
- **OnEnable： **仅在对象处于激活状态时调用）在启用对象后立即调用此函数。在创建 MonoBehaviour 实例时（例如加载关卡或实例化具有脚本组件的游戏对象时）会执行此调用。（在Awake之后调用）

请注意，对于添加到场景中的对象，在为任何对象调用 Start 和 Update 等函数之前，会为_所有_ 脚本调用 Awake 和 OnEnable 函数。当然，在游戏运行过程中实例化对象时，不能强制执行此调用。  **[Unity - Scripting API: MonoBehaviour.OnEnable()](https://docs.unity.cn/2021.3/Documentation/ScriptReference/MonoBehaviour.OnEnable.html)**

#### Editor

- **Reset：**调用 Reset 可以在脚本首次附加到对象时以及使用 *Reset* 命令时初始化脚本的属性。
- **OnValidate：**每当设置脚本的属性时都会调用 OnValidate，包括反序列化对象时，这可能发生在不同的时间，例如在编辑器中打开场景时和域重新加载后。

> Reset函数用于在检查器中提供良好的默认值，而OnValidate函数用于验证发生变化的数据。

#### 第一次帧更新前

- **Start：**仅当启用脚本实例后，才会在第一次帧更新之前调用 Start。

对于场景资源中的对象，在为任何脚本调用 Update 等函数之前，将在所有脚本上调用 Start 函数。当然，在游戏运行过程中实例化对象时，不能强制执行此调用。

#### 帧与帧之间

- **OnApplicationPause：**在帧的结尾处调用此函数（在正常帧更新之间有效检测到暂停）。在调用 **OnApplicationPause** 之后，将发出一个额外帧，从而允许游戏显示图形来指示暂停状态。（如何使用请详见我的语法文档，很重要（Time.scaleTime=0无法触发））

#### 更新顺序

跟踪游戏逻辑和交互、动画、摄像机位置等的时候，可以使用一些不同事件。常见方案是在 **Update** 函数中执行大多数任务，但是也可以使用其他函数。

- **FixedUpdate：**调用 **FixedUpdate** 的频度常常超过 **Update**。如果帧率很低，可以每帧调用该函数多次；如果帧率很高，可能在帧之间完全不调用该函数。在 **FixedUpdate** 之后将立即进行所有物理计算和更新。在 **FixedUpdate** 内应用运动计算时，无需将值乘以 **Time.deltaTime**。这是因为 **FixedUpdate** 的调用**基于可靠的计时器（独立于帧率）。**（换言之，这玩意独立于帧率，并使用固定的时间间隔Time.fixedDeltaTime进行更新）
- **Update：**每帧调用一次 **Update**。这是用于帧更新的主要函数。（这玩意依靠帧率，并且帧率越高调用次数越多）
- **LateUpdate：**每帧调用一次 **LateUpdate__（在** Update__ 完成后）。**LateUpdate** 开始时，在 **Update** 中执行的所有计算便已完成。**LateUpdate** 的常见用途是跟随第三人称摄像机。如果在 **Update** 内让角色移动和转向，可以在 **LateUpdate** 中执行所有摄像机移动和旋转计算。这样可以确保角色在摄像机跟踪其位置之前已完全移动。（同Update，这玩意也是受帧率影响）

通常，不应依赖为不同的游戏对象调用相同事件函数的顺序 - 除非顺序明确记录或可设置。（如果需要对播放器循环进行更细粒度的控制，可以使用 [PlayerLoop API](https://docs.unity.cn/cn/current/ScriptReference/LowLevel.PlayerLoop.html)。）

> **不能指定为同一 MonoBehaviour 子类的不同实例调用事件函数的顺序。**（实例就是对象）例如，一个 MonoBehaviour 的 **Update** 函数可能会在另一个游戏对象（包括其父级或子级游戏对象）上的相同 MonoBehaviour 的 **Update** 函数之前或之后调用。
>
> **可以指定一个 MonoBehaviour 子类的事件函数应在不同子类的事件函数之前调用（使用 Project Settings 窗口的 Script Execution Order 面板）。**例如，如果有两个脚本，EngineBehaviour 和 SteeringBehaviour，可以设置 Script Execution Order，这样 EngineBehaviours 始终在 SteeringBehaviours 之前更新。
>
> 简单来说，就是对于同一子类的不同实例，无法决定这些实例之间的事件函数调用顺序，因为都是一家人；但是可以指定不同子类之间的事件函数的调用顺序

*下文需学过动画系统Animation再来观看*

---

#### 动画更新循环(我基本看不懂。。。)(此处建议在学完动画系统Animation之后再来看)

Unity 评估动画系统时，将调用以下函数和 [Profiler](https://docs.unity.cn/cn/current/Manual/Profiler.html) 标记。

- **OnStateMachineEnter：**在**状态机更新 (State Machine Update)** 步骤中，当控制器的状态机进行流经 Entry 状态的转换时，将在第一个更新帧上调用此回调。在转换到 **StateMachine** 子状态时不会调用此回调。

  仅当动画图中存在控制器组件（例如，[**AnimatorController**](https://docs.unity.cn/cn/current/ScriptReference/Animations.AnimatorController.html)、[**AnimatorOverrideController**](https://docs.unity.cn/cn/current/ScriptReference/AnimatorOverrideController.html) 或 [**AnimatorControllerPlayable**](https://docs.unity.cn/cn/current/ScriptReference/Animations.AnimatorControllerPlayable.html)）时才会发生此回调。

  > **状态机的状态指的是游戏对象在某一时刻所处的状态。**例如，在一个横版游戏中，女英雄可能有站立、跳跃、下蹲和下斩等不同的状态。每个状态都有一些预定义的行为，例如在跳跃状态下，女英雄会执行跳跃动画。状态机可以管理这些不同的状态，并根据游戏逻辑在不同的状态之间进行转换。
  >
  > 举个例子，当女英雄站在地面上时，她处于站立状态。当玩家按下向上方向键时，状态机会将女英雄的状态从站立转换为跳跃，并执行跳跃动画。当女英雄落回地面时，状态机会再次将她的状态从跳跃转换为站立。

  下面是**动画状态机和Update函数**的区别

  > **动画状态机和Update函数是两个不同的概念。**动画状态机是Unity中用来管理游戏对象动画的一种工具，它定义了一系列的状态和转换，每个状态都对应着一个特定的动画。开发者可以通过脚本来控制状态机中的参数，从而影响状态机的流程。
  >
  > 而Update函数是每帧调用一次的主要函数，用于帧更新。它通常用来执行游戏逻辑，例如移动游戏对象、检测碰撞等。
  >
  > 简单来说，动画状态机专注于管理游戏对象的动画，而Update函数则用来执行游戏逻辑。

  **注意：**将此回调添加到 [**StateMachineBehaviour**](https://docs.unity.cn/cn/current/ScriptReference/StateMachineBehaviour.html) 组件会禁用多线程的状态机评估。

- **OnStateMachineExit：**在**状态机更新 (State Machine Update)** 步骤中，当控制器的状态机进行流经 Exit 状态的转换时，将在最后一个更新帧上调用此回调。在转换到 **StateMachine** 子状态时不会调用此回调。

  仅当动画图中存在控制器组件（例如，[**AnimatorController**](https://docs.unity.cn/cn/current/ScriptReference/Animations.AnimatorController.html)、[**AnimatorOverrideController**](https://docs.unity.cn/cn/current/ScriptReference/AnimatorOverrideController.html) 或 [**AnimatorControllerPlayable**](https://docs.unity.cn/cn/current/ScriptReference/Animations.AnimatorControllerPlayable.html)）时才会发生此回调。

  **注意：**将此回调添加到 [**StateMachineBehaviour**](https://docs.unity.cn/cn/current/ScriptReference/StateMachineBehaviour.html) 组件会禁用多线程的状态机评估。

- **触发动画事件 (Fire Animation Events)：**调用在上次更新时间和当前更新时间之间采样的所有剪辑中的所有动画事件。

- **StateMachineBehaviour (OnStateEnter/OnStateUpdate/OnStateExit)：**一个层最多可以有 3 个活动状态：当前状态、中断状态和下一个状态。使用一个定义 **OnStateEnter**、**OnStateUpdate** 或 **OnStateExit** 回调的 StateMachineBehaviour 组件为每个活动状态调用此函数。

  依次针对当前状态、中断状态和下一个状态调用此函数。

  仅当动画图中存在控制器组件（例如，[**AnimatorController**](https://docs.unity.cn/cn/current/ScriptReference/Animations.AnimatorController.html)、[**AnimatorOverrideController**](https://docs.unity.cn/cn/current/ScriptReference/AnimatorOverrideController.html) 或 [**AnimatorControllerPlayable**](https://docs.unity.cn/cn/current/ScriptReference/Animations.AnimatorControllerPlayable.html)）时才会执行此步骤。

- **OnAnimatorMove：**在每个更新帧中为每个 Animator 组件调用一次此函数来修改根运动 (Root Motion)。

- **StateMachineBehaviour(OnStateMove)：**使用定义此回调的 [**StateMachineBehaviour**](https://docs.unity.cn/cn/current/ScriptReference/StateMachineBehaviour.html) 在每个活动状态中调用此函数。

- **OnAnimatorIK：**设置动画 IK。为每个启用 **IK pass** 的 Animator Controller 层进行一次此调用。

  仅当使用人形骨架时才会执行此事件。

- **StateMachineBehaviour(OnStateIK)：**使用在启用 **IK pass** 的层上定义此回调的 [**StateMachineBehaviour**](https://docs.unity.cn/cn/current/ScriptReference/StateMachineBehaviour.html) 组件在每个活动状态中调用此函数。

- **WriteProperties：**从主线程将所有其他动画属性写入场景。

### 有用的性能分析标记

[脚本生命周期流程图](https://docs.unity.cn/cn/2021.3/Manual/ExecutionOrder.html#ScriptLifecycleFlowchart)中显示的某些动画函数不是可以调用的事件函数；它们是 Unity 处理动画时调用的内部函数。

这些函数具有 Profiler 标记，因此您可以使用 [Profiler](https://docs.unity.cn/cn/2021.3/Manual/Profiler.html) 查看 Unity 在帧中调用这些函数的时间。知道 Unity 调用这些函数的时间有助于准确了解所调用的事件函数的具体执行时间。

例如，假设在 **FireAnimationEvents** 回调中调用 [Animator.Play](https://docs.unity.cn/cn/2021.3/ScriptReference/Animator.Play.html)。如果知道只有在执行**状态机更新 (State Machine Update)** 和**流程图 (Process Graph)** 函数后才会触发 **FireAnimationEvents** 回调，就可以预期动画剪辑会在下一帧播放，而不是马上播放。

- **状态机更新 (State Machine Update)：**在执行序列的此步骤中评估所有状态机。仅当动画图中存在控制器组件（例如，[**AnimatorController**](https://docs.unity.cn/cn/2021.3/ScriptReference/Animations.AnimatorController.html)、[**AnimatorOverrideController**](https://docs.unity.cn/cn/2021.3/ScriptReference/AnimatorOverrideController.html) 或 [**AnimatorControllerPlayable**](https://docs.unity.cn/cn/2021.3/ScriptReference/Animations.AnimatorControllerPlayable.html)）时才会发生此回调。

  **注意：**状态机评估通常是多线程的，但添加某些回调（例如，**OnStateMachineEnter** 和 **OnStateMachineExit**）会禁用多线程。请参数上文的[动画更新循环](https://docs.unity.cn/cn/2021.3/Manual/ExecutionOrder.html#AnimationUpdateLoop)以了解详细信息。

- **ProcessGraph：**评估所有动画图。此过程包括对需要评估的所有动画剪辑进行采样以及计算根运动 (Root Motion)。

- **ProcessAnimation：**混合动画图的结果。

- **WriteTransforms：**将所有动画变换从工作线程写入场景。

  如果一个人形骨架的多个层启用了 **IK pass**，则该人形骨架可以有多个 **WriteTransforms** 通道（请参阅[脚本生命周期流程图](https://docs.unity.cn/cn/2021.3/Manual/ExecutionOrder.html#ScriptLifecycleFlowchart)）

  ---

*上文需学过动画系统Animation再来观看*

#### Rendering（渲染）(目前还不怎么懂)

*渲染流程是指计算机图形学中将三维场景转换为二维图像的过程。*在Unity中，这个过程包括多个阶段，每个阶段都有相应的事件被调用。

1. 首先，在**摄像机开始剔除场景之前**，会调用`OnPreCull`事件。剔除操作将确定摄像机可以看到哪些对象。正好在进行剔除之前调用`OnPreCull`。

   > 举个例子，假设您正在玩一个3D游戏，游戏中有一个摄像机，它负责捕捉游戏世界中的画面并显示在您的屏幕上。但是，游戏世界中可能有很多物体，而摄像机并不需要显示所有的物体。例如，如果一个物体被其他物体挡住了，或者它离摄像机太远，那么这个物体就不需要被显示出来。
   >
   > 为了优化性能，Unity会在渲染画面之前执行一个剔除操作，来决定哪些物体需要被显示，哪些物体不需要被显示。OnPreCull事件就是在这个剔除操作执行之前被调用的。您可以在OnPreCull事件中执行您自己的代码，例如更改摄像机的设置，以影响剔除操作的结果。
   >
   > **渲染画面通常是在每一帧中执行一次。也就是说，每当游戏画面更新时，Unity都会执行一次渲染操作来绘制新的画面。在这个过程中，剔除操作会在渲染操作之前执行。**

2. 接下来，如果对象变为对**任何摄像机可见或不可见**，将调用`OnBecameVisible/OnBecameInvisible`事件。

   > 如果一个对象被渲染到屏幕上，那么它就是可见的。如果一个对象没有被渲染到屏幕上，那么它就不可见。

3. 然后，如果**对象可见**，则为**每个摄像机**调用一次`OnWillRenderObject`事件。

4. 在摄像机开始渲染场景之前，会调用`OnPreRender`事件。

5. 所有常规场景渲染完成之后，会调用`OnRenderObject`事件。此时，使用 [GL](https://docs.unity.cn/cn/2021.3/ScriptReference/GL.html) 类或 [Graphics.DrawMeshNow](https://docs.unity.cn/cn/2021.3/ScriptReference/Graphics.DrawMeshNow.html) 来绘制自定义几何形状。

6. 在摄像机完成场景渲染后，会调用`OnPostRender`事件。

   在场景渲染完成后，会调用`OnRenderImage`事件以允许对图像进行后处理,。请参阅[后期处理效果](https://docs.unity.cn/cn/2021.3/Manual/PostProcessingOverview.html)。

7. 最后，在每帧中，会多次调用`OnGUI`事件以响应GUI事件。首先处理布局和重新绘制事件，然后为每个输入事件处理布局和键盘/鼠标事件。

此外，还有一个`OnDrawGizmos`事件，它用于在场景视图中绘制辅助图标以实现可视化。

需要注意的是，这些回调仅适用于内置渲染管线。

---

### 协程

Update 函数返回后将运行正常协程更新。协程是一个可暂停执行 (yield) 直到给定的 YieldInstruction 达到完成状态的函数。 协程的不同用法：

- **yield** 在下一帧上调用所有 Update 函数后，协程将继续。
- **yield WaitForSeconds** 在为帧调用所有 Update 函数后，在指定的时间延迟后继续。
- **yield WaitForFixedUpdate** 在所有脚本上调用所有 FixedUpdate 后继续。如果协同程序在 FixedUpdate 之前生成，那么它会在当前帧的 FixedUpdate 之后继续运行。
- **yield WWW** 在 WWW 下载完成后继续。
- **yield StartCoroutine** 将协程链接起来，并会等待 MyFunc 协程先完成。

### 销毁对象时

- **OnDestroy：**对象存在的最后一帧完成所有帧更新之后，调用此函数（可能应 Object.Destroy 要求或在场景关闭时销毁该对象）。

### 退出时

在场景中的所有活动对象上调用以下函数：

- **OnApplicationQuit：**在退出应用程序之前在所有游戏对象上调用此函数。在编辑器中，用户停止播放模式时，调用函数。
- **OnDisable：**行为被禁用或处于非活动状态时，调用此函数。

*协程目前还看不懂*，还有上面那几个，先不管了，后面遇到再说再分析

---

---

## 协程

### 什么是协程

协程（Coroutine）是一种特殊的功能，它允许在函数中暂停执行一段时间，并在需要时恢复执行。

这与我们之前所学的IEnumerable与yield有关，试想一下，当我需要将一个状态描述为渐变时，该怎么写代码？

按传统方法进行函数的描写，会导致这样的后果：由于循环函数会一下子给出答案，例如下面，将无法将渐变的效果展现出来

```c#
void Fade()
{
    Color c = renderer.material.color;
    for (float alpha = 1f; alpha >= 0; alpha -= 0.1f)
    {
        c.a = alpha;
        renderer.material.color = c;
    }
}
```

使用 IEnumerator 返回类型进行声明，并在主体中包含一个 yield 返回语句。yield return nullline 是执行暂停并在下一帧恢复的点。它将在此处暂停，返回一个结果，并且在下一帧又解除暂停；然后又在演算完结果后再进行暂停

类似于是，在每帧计算一个结果后立即暂停，将该结果作用于当前帧，然后解除暂停。如此往复，直到所有结果被计算完成

```C#
IEnumerator Fade()
{
    Color c = renderer.material.color;
    for (float alpha = 1f; alpha >= 0; alpha -= 0.1f)
    {
        c.a = alpha;
        renderer.material.color = c;
        yield return null;
    }
}
```

要使一个协程运行，需要使用StartCoroutine开启一个协程

```C#
void Update()
{
    if (Input.GetKeyDown("f"))
    {
        StartCoroutine(Fade());
    }
```

由于协程是帧帧调用且帧帧进行更新迭代，多次执行会导致卡顿，所有往往我们使用WaitForSeconds控制迭代的次数

```C#
IEnumerator DoCheck()
{
    for(;;)
    {
        if (ProximityCheck())
        {
            // Perform some action here
        }
        yield return new WaitForSeconds(.1f);
    }
}
```

`yield return new WaitForSeconds(.1f)` 是一种特殊的语句，它用于在协程中暂停协程的执行一段时间。在这个例子中，协程会暂停 0.1 秒。在暂停期间，游戏会继续运行，直到暂停时间结束后，协程会从暂停的地方继续执行

假设以60帧作为一秒，那么使用yield return new WaitForSeconds(.1f)理论上是6帧进行一次更新迭代；不使用WaitForSeconds，则是每帧进行一个更新迭代

### 协程的停止

在 Unity 中，有几种方法可以停止协程的执行。您可以使用 `StopCoroutine` 和 `StopAllCoroutines` 函数来停止一个或所有协程。此外，如果您将协程所依附的游戏对象禁用（通过调用 `SetActive(false)`），那么协程也会停止。如果您销毁了一个 MonoBehaviour 实例（通过调用 `Destroy(example)`），那么 Unity 会立即触发 `OnDisable` 函数并处理协程，从而有效地停止它。最后，在帧结束时，Unity 会调用 `OnDestroy` 函数。

需要注意的是，如果您仅仅是禁用了一个 MonoBehaviour（通过设置 `enabled = false`），那么 Unity 并不会停止协程。

---

## 欧拉角、四元数以及旋转变换的理解

### 一、欧拉角

> 引用自知乎：[如何通俗地解释欧拉角？之后为何要引入四元数？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/47736315)

**欧拉角**的选取顺序有 (x,y,x),(x,z,x),(y,x,y),(y,z,y),(z,x,z),(z,y,z)(x, y, x), (x, z, x), (y, x, y), (y, z, y), (z, x, z), (z, y, z)(x, y, x), (x, z, x), (y, x, y), (y, z, y), (z, x, z), (z, y, z) 这6种，可见选取顺序是a,b,a这样的顺序，也就是**绕a轴旋转某角度后，绕新生成的b轴旋转一个角度，最后绕两次旋转以后的a轴再旋转一个角度**，以此表示最终的方向。

***欧拉角描述的绕着坐标轴的旋转，都是绕着物体的本地坐标系的坐标轴进行旋转***

以(z,x,z')作为例子，每一次转动都是以最新的坐标轴位置进行转动；先是绕**当前z轴**旋转，再是绕着当前**x轴旋转**，最后再绕着当前**z轴旋转** ；最终的红色坐标系就是结果

![动图](https://picx.zhimg.com/50/v2-35da80e428ca5750491ffd2770e187e1_720w.webp?source=1940ef5c)

<img src="https://picx.zhimg.com/80/v2-c339d9c88aa8988a0480365c74bbe1eb_1440w.webp?source=1940ef5c" alt="img" style="zoom:50%;" />

我们可以看出，无论哪种表示方式，记录这样一个变换，**至少需要三个角的sine和cosine值，也就是一共存储6个单位数据**。<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234501.png" alt="image-20230827182240728" style="zoom:60%;" />

> 三个基本旋转矩阵
>
> ![img](https://pic2.zhimg.com/80/v2-512338f54bbdbc46c3dfa3e829641b19_1440w.webp)
>
> 旋转矩阵的推导：[旋转矩阵（Rotation Matrix)的推导 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/433389563)

### 二、万向锁

[万向锁是指在使用欧拉角表示三维旋转时，当其中一个旋转角度达到90度时，另外两个旋转角度会变得等效，从而导致丢失一个自由度的现象。这种现象也称为“万向节死锁”](https://zhuanlan.zhihu.com/p/74040465)

这是一个长方体以(30,90,10)旋转得出的结果

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234502.png" alt="image-20230827190901962" style="zoom:50%;" />

然而，这与(20,90,0)的结果是相同的<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234503.png" alt="image-20230827191052652" style="zoom:50%;" />



在数学上的理解为：<img src="https://pic2.zhimg.com/80/v2-35df951674e6517119af35a8dee7d0fd_1440w.webp" alt="img" style="zoom:65%;" />即必定会丧失一个角的作用

如果用手机做演示，那么将是精彩绝伦简单易懂：

> 自己拿一个手机来做一下试验，有助于理解万象锁：
>
> 可以拿出手机放在桌面上，屏幕朝上，手机的最长边垂直与桌子的边缘设置为X轴，这个时候屏幕的短边平行于桌子的边缘设置为Y轴，因此垂直与屏幕的向量为Z轴。我们先绕手机的最长边X轴顺时针旋转30度，这个时候手机离开桌面，留下一个长边与桌子接触；然后再绕Y轴，也就是手机的短边旋转90度，让屏幕面与桌子的边缘平行；再绕Z轴旋转10度，也就是绕垂直于屏幕的轴旋转10度，这个时候你会发现，**绕Z轴旋转时，屏幕面还是平行桌子的边缘，而此时绕Z轴旋转的角度给手机姿态带来的影响和最开始旋转X轴给手机姿态带来的影响是一样的——都是使手机最终的姿态（已经绕Y轴旋转了90度使得手机屏幕与桌子边缘平行）为绕着垂直于屏幕的轴旋转一定的角度**。你完全可以不用绕Z轴旋转，通过调节绕X轴旋转的角度数，使得最终手机的姿态和上述旋转过程达到的姿态一样。此时就是造成了万向锁。

即如今的绕z轴转动相当于是绕着一开始x的轴转动，因此最后的绕z轴转动可以一开始就叠加给最初的绕x轴转动

### 三、四元数


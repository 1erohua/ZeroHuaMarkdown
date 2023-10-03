# Unity文档语法笔记

*前言：此笔记本不是给正常人看的，只是为了锻炼我自己的英语看的*

---

## 重要的类——GameObject

[重要的类 - GameObject - Unity 手册](https://docs.unity.cn/cn/2021.3/Manual/class-GameObject.html)

[在Unity中，GameObject类是所有可以存在于场景中的实体的基类

它可以充当用于确定GameObject外观以及GameObject作用的功能组件的容器

### 场景状态属性

对于场景中的GameObject，可以点击并获取其属性（将在Inspector中的显示控件）；这些控件可以由GameObject的API进行脚本控制	

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234846.png" alt="image-20230827100328579" style="zoom:45%;" />

### 活动状态

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234847.png" alt="image-20230827100353471" style="zoom:45%;" />

默认状态下处于活动状态（Active），但是可以停用，停用后将不可视并且不接收任何事件和回调，例如Update和FixedUpdate

> GameObject 的**活动**状态由 GameObject 名称左侧的复选框表示。可以使用 [`GameObject.SetActive`](https://docs.unity.cn/cn/2021.3/ScriptReference/GameObject.SetActive.html) 控制此状态。
>
> 还可以使用 [`GameObject.activeSelf`](https://docs.unity.cn/cn/2021.3/ScriptReference/GameObject-activeSelf.html) 读取当前活动状态，使用 [`GameObject.activeInHierarchy`](https://docs.unity.cn/cn/2021.3/ScriptReference/GameObject-activeInHierarchy.html) 读取 GameObject 是否在场景中实际处于活动状态。**这两者中的后者是必要的，因为 GameObject 是否实际处于活动状态取决于其自身的活动状态，以及其所有父项的活动状态。如果其所有父项都不处于活动状态，则尽管它自己设置为活动状态，它也不会处于活动状态。**

### 静态状态

![GameObject 的静态状态](https://docs.unity.cn/cn/2021.3/uploads/Main/GOInspectorStaticSetting.png)

处于静态状态下的物体无法移动，这意味着它将不需要跟着移动而进行变化，因此有利于提前的渲染

总之，游戏对象的静态状态可以帮助优化渲染和其他系统，但只有当该对象不会移动时才有效。

### 标签与层

#### 标签

[标签（Tag）是用来对游戏物体进行分类的，从而更加方便地在代码中对某一类物体进行统一操作。例如，可以使用GameObject.FindWithTag()函数查找场景中指定标签的物体。要添加新标签，可以在Tags and Layers设置（主菜单：Edit > Project Settings，然后选择Tags and Layers类别）中添加新标签

#### 层

Unity的层（Layers）是一种用于管理游戏对象和场景中元素的属性，例如渲染顺序、碰撞检测和光线投射等。层是Unity中的一种数据结构，可以用来分配游戏对象到不同的层级，以便在渲染、碰撞检测和光线投射等方面进行独立的控制。a

每个游戏对象都有一个对应的层（Layer），这些层被分配为从0到31的整数。默认情况下，游戏对象被分配到第0层，但你可以在Inspector窗口中修改层的分配。

> 在Unity中，层有以下几种用途：
>
> 1. 渲染顺序：层的值越小，游戏对象在渲染时的顺序就越靠前。因此，通过调整层的值，可以控制不同游戏对象之间的渲染顺序。
> 2. 碰撞检测：在物理引擎中，只有相同层或相邻层的游戏对象之间才会发生碰撞检测。因此，通过将相关的游戏对象分配到同一层，可以控制哪些对象之间会发生碰撞检测。
> 3. 光线投射：灯光和摄像机可以设置投射的层，只对指定层的游戏对象产生影响。例如，可以将灯光设置为仅对第1层的游戏对象产生光照效果。







---

## 重要的类——Vector3

> [Unity - Scripting API: Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html)

This **structure** is used throughout Unity to pass 3D **positions** and **directions** around. It also contains functions for doing common vector operations.（注意Vector不是一个类，它是一个结构，即它是值类型的数据）

> other classes can be used to manipulate vectors and points as well. For example the [Quaternion](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Quaternion.html) and the [Matrix4x4](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Matrix4x4.html) classes are useful for rotating or transforming vectors and points.

#### 静态属性

（都是缩写）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234848.png" alt="image-20230527103711042" style="zoom:75%;" />

注：float.PositiveInfinity是正无穷大，而float.NegativeInfinity是负无穷大

*[在这个标准中，正无穷大和负无穷大是通过一种特殊的浮点数编码方式来表示的。例如，在IEEE 754单精度浮点数（binary32）中，一个浮点数由三部分组成：符号位（1位）、指数位（8位）和尾数位（23位）。当指数位全为1且尾数位全为0时，这个浮点数表示正无穷大（如果符号位为0）或负无穷大（如果符号位为1）](https://en.wikipedia.org/wiki/IEEE_754)[1](https://en.wikipedia.org/wiki/IEEE_754)[2](https://en.wikipedia.org/wiki/Single-precision_floating-point_format)。*

#### 对象属性

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234849.png" alt="image-20230527104935579" style="zoom:75%;" />

第一个是返回长度，magnitude有大小的意思；第二个是单位化（normalized），返回一个长度为一的该向量；第三个是平方化，返回该向量的长度的平方值

#### 对象属性

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234850.png" alt="image-20230527105546678" style="zoom:75%;" />

#### 静态方法（太多了，建议需要的就去网站找）

| Method                                                       | Description                                                  | HowToUse                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Angle](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.Angle.html) | Calculates the angle between vectors from and.（计算向量夹角） | **public static float Angle(Vector3 from,Vector3 to)**       |
| [ClampMagnitude](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.ClampMagnitude.html) | Returns a copy of vector with its magnitude clamped to maxLength.(返回一个向量的副本，且长度为maxLength) | **public static Vector3 ClampMagnitude(Vector3 vector, float maxLength)** |
| [Cross](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.Cross.html) | Cross Product（交叉积） of two vectors. （返回两个向量的叉乘，当然也是一个向量） | public static Vector3 Cross(Vector3 lhs,Vector3 rhs)（lhs是左，rhs是右，Cross遵循左手原则，可以根据左手进行判断）（由于返回值是一个Vector3对象，可以使用normalized单位化长度） |
| [Distance](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.Distance.html) | Returns the distance between a and b.（a和b是两个position，不是direction） | **public static float Distance(Vector3 a,Vector3 b)**        |
| [Dot](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.Dot.html) | Dot Product of two vectors.(点乘)                            | **public static float Dot(Vector3 lhs,Vector3 rhs**     (我觉得的使用场景更多是这些话)For [normalized](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3-normalized.html) vectors Dot returns 1 if they point in exactly the same direction, -1 if they point in completely opposite directions and zero if the vectors are perpendicular. |
| [Lerp](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.Lerp.html) | Linearly interpolates between two points.                    | 两个点之间的线性插值的坐标，其中t是线性参数（t在0到1）                                                                        public static [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **Lerp**([Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **a**, [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **b**, float **t**); |
| [LerpUnclamped](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.LerpUnclamped.html) | Linearly interpolates between two vectors.                   | 不限制t的范围，即t可以超过1也可以小于0                                            [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **Lerp**([Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **a**, [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **b**, float **t**); |
| [Max](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.Max.html) | Returns a vector that is made from the largest components of two vectors. | public static Vector3 Max(Vector3 lhs,Vector3 rhs)(返回一个向量，该向量的x、y、z分别取每个向量x、y、z的最大值)（如（1，2，7）与（4，5，6），结果为（4，5，7）） |
| [Min](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.Min.html) | Returns a vector that is made from the smallest components of two vectors. | public static Vector3 Min(Vector3 lhs,Vector3 rhs)(同max，只不过是取最小的) |
| [MoveTowards](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.MoveTowards.html) | Calculate a **position** between the points specified by current and target, moving no farther than the distance specified by maxDistanceDelta. | public static Vector3 MoveTowards(Vector3 current,Vector3 target, float maxDistanceDelta)(在最大移动距离的限制下，从一个目标点向着另一个目标点移动时，返回移动结果的坐标)（比如由(0,0,1)移到(0,0,2)，但是限制移动0.5，那么返回的向量结果时(0,0,0.5)） |
| [Normalize](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.Normalize.html) | Makes this vector have a magnitude of 1.                     | public static [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **Normalize**([Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **value**);(返回一个单位化的向量) |
| [OrthoNormalize](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.OrthoNormalize.html) | Makes vectors normalized and orthogonal to each other.（normal法向量，tangent切向量） | public static void **OrthoNormalize**(ref [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **normal**, ref [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **tangent**);//这个是将两个向量单位化并以normal正交化                                                                           **另一个重载：**public static void **OrthoNormalize**(ref [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **normal**, ref [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **tangent**, ref [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **binormal**);                                                                                           将三个向量标准正交化，这对建立一个自己的坐标系很有用（建议看原文档） |
| [Project](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.Project.html) | Projects a vector onto another vector.                       | 将一个向量投影到另一个向量上                                       public static [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **Project**([Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **vector**, [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **onNormal**);<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234851.png" alt="image-20230527170244162" style="zoom:33%;" /> |
| [ProjectOnPlane](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.ProjectOnPlane.html) | Projects a vector onto a plane defined by a normal orthogonal to the plane. | 用于计算一个向量在一个平面上的投影。它接受两个参数：一个是要投影的向量，另一个是定义平面的法向量(planeNormal)。函数会返回一个新的向量，表示原始向量在平面上的投影。                                                                      public static Vector3 ProjectOnPlane(Vector3 vector, Vector3 planeNormal |
| [Reflect](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.Reflect.html) | Reflects a vector off the plane defined by a normal.         | <img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234852.png" alt="image-20230527195434929" style="zoom:50%;" />                                                  public static [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **Reflect**([Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **inDirection**, [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **inNormal**); |
| [RotateTowards](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.RotateTowards.html) | Rotates a vector current towards target.                     | 类似于MoveTowards,只不过是现在是对向量进行操作（所有向量放到原点）                                                              public static Vector3 RotateTowards(Vector3 current, Vector target, float maxRadianaDelta, float maxMagnitudeDelta) （ maxRadianaDelta是最大旋转角，maxMagnitudeDelta是最大变化长度）如果两个向量的长度不同，那么向量得出的结果的向量长度将采取线性插值的方法进行计算                                                                     线性插值的公式为：result = current + (target - current) * t  ，`t` 的值是由 `maxRadiansDelta` 参数决定的。这个参数指定了每次旋转的最大角度。随着旋转的进行，`t` 的值将根据旋转的角度增加，直到旋转完成 |
| [Scale](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.Scale.html) | Multiplies two vectors component-wise.                       | 将两个向量的x,y,z分别相乘，得到一个新向量。比如(1,2,3)乘以(4,5,6)，结果为(4,10,18)                                               public static Vector3 Scale(Vector3 a, Vector3 b) |
| [SignedAngle](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.SignedAngle.html) | Calculates the signed angle between vectors from and to in relation to axis. | 它返回一个浮点数，表示从“from”向量旋转到“to”向量所需的角度，单位为度。它以第三个参数为轴，也就是说，如果你用左手握住旋转轴，使拇指指向轴的正方向（也就是指向屏幕外），那么其他四个手指所指的方向就是旋转方向。顺时针方向是正值                                    public static float **SignedAngle**([Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **from**, [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **to**, [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **axis**); |
| [Slerp](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.Slerp.html) | Spherically interpolates between two vectors.                | 在两个向量之间进行球形插值，其中t为0到1的浮点数                                            public static [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **Slerp**([Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **a**, [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **b**, float **t**); |
| [SlerpUnclamped](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.SlerpUnclamped.html) | Spherically interpolates between two vectors.（不限制球形插值的参数t的范围，即可以超过1或者小于0） | 不限制 `t` 的范围可以让你在使用 `Vector3.SlerpUnclamped` 方法时获得更大的灵活性，但也需要注意结果可能超出预期的范围。                                           public static [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **SlerpUnclamped**([Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **a**, [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **b**, float **t**); |
| [SmoothDamp](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.SmoothDamp.html) | Gradually changes a vector towards a desired goal over time.（这是对position进行操作的） | 1. 它可以将当前位置 `current` 平滑地移动到目标位置 `target`。它通过类似于弹簧阻尼器的函数来平滑向量，永远不会超调。最常见的用途是平滑跟随摄像机。                                2. `currentVelocity` 参数是当前速度的引用。这个值在每次调用函数时都会被修改。                                                       3. `smoothTime` 参数指定了到达目标所需的大致时间。值越小，到达目标的速度就越快。可以选择使用 `maxSpeed` 参数来限制最大速度。（ `maxSpeed`默认正无穷）                                                                    4. `deltaTime` 参数指定了自上次调用此函数以来经过的时间，默认为 `Time.deltaTime` 变量。                                                       public static [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **SmoothDamp**([Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **current**, [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **target**, ref [Vector3](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Vector3.html) **currentVelocity**, float **smoothTime**, float **maxSpeed** = Mathf.Infinity, float **deltaTime** = Time.deltaTime); |

---

## 重要的类——Time

> [Unity - Scripting API: Time](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time.html)

Provides an interface to get time information from Unity.

Unity有两个追踪时间的系统，一个是每一步之间的时间不固定，另一个是每一步之间的时间固定。

可变时间步长的系统是在屏幕上重复绘制一帧，每帧运行一次你的应用程序或游戏代码。

固定时间步长系统每一步都以预先定义的数量向前迈进，并且与视觉帧的更新没有联系。它更多地与物理系统联系在一起，物理系统以固定时间步长指定的速率运行，但如果有必要，你也可以在每个固定时间步长执行自己的代码。

### 重要的属性以及解释、功能

| Static Properties                                            | Description                                                  | Function（功能）                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [timeScale](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-timeScale.html) | 用于控制时间的流逝速度。当 `timeScale` 为 1.0 时，时间以与现实时间相同的速度流逝。当 `timeScale` 为 0.5 时，时间流逝速度比现实时间慢 2 倍。                      当其值为负值时，则会被忽略（按照上一次的timeScale来计算） | 将 `timeScale` 设置为零，则应用程序将暂停（**如果您的所有函数都与帧速率无关**）。                                                                                                                                   *与帧速率相关的函数是指其行为取决于每秒渲染的帧数的函数。例如，如果您在 `Update` 函数中移动游戏对象，而移动距离与帧速率成正比，则该函数就与帧速率相关。这意味着，如果帧速率降低，则游戏对象的移动速度也会降低。为了避免这种情况，您可以使用 `Time.deltaTime` 来使函数与帧速率无关。 `Time.deltaTime` 表示上一帧所用的时间，因此您可以使用它来根据实际时间而非帧速率来调整游戏对象的移动速度。*                                                                                                                                      **请注意，更改 `timeScale` 只会在接下来的帧中生效。每帧执行`MonoBehaviour.FixedUpdate` 的次数取决于 `timeScale`。因此，要保持每帧 `FixedUpdate` 回调的数量不变，您还必须将 `Time.fixedDeltaTime` 乘以 `timeScale`。是否需要进行此调整取决于游戏。** |
| [deltaTime](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-deltaTime.html) | The interval in seconds from the last frame to the current one **(Read Only).** | 这个属于是老朋友了，上一帧到这一帧的时间。假如帧数是60帧，那么这个值为（1.0f/60,也就是0.0167）                                                                                                    **当它在 `MonoBehaviour.FixedUpdate` 中被调用时，它返回`Time.fixedDeltaTime`。**                                                                                                   在`MonoBehaviour.OnGUI` 中调用 `deltaTime` 是不可靠的，因为 Unity 可能会在每一帧中多次调用它。 |
| [time](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-time.html) | The time at the beginning of this frame (Read Only).         | 它表示当前帧开始时的时间（只读）。这是自应用程序启动以来经过的秒数，受 `Time.timeScale` 缩放和 `Time.maximumDeltaTime` 调整。当从 `MonoBehaviour.FixedUpdate` 内部调用时，它返回 `Time.fixedTime`。                                                     在 `Awake` 消息期间，此值未定义，并在所有这些消息完成后开始。如果编辑器暂停，则此值不会更新。有关不受暂停影响的时间值，请参阅 `Time.realtimeSinceStartup`。 |
| [fixedDeltaTime](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-fixedDeltaTime.html) | The interval in seconds at which physics and other fixed frame rate updates (like MonoBehaviour's FixedUpdate) are performed. | `Time.fixedDeltaTime`是一个静态浮点数，表示物理和其他固定帧率更新（如`MonoBehaviour`的`FixedUpdate`）执行的时间间隔（以秒为单位）。                      **`fixedDeltaTime` 的值本身不会随着 `Time.timeScale` 的改变而改变，它始终保持一个固定的数值。但是，当你改变 `Time.timeScale` 的值时，游戏内的时间流逝速度会改变，这会影响到 `fixedDeltaTime` 所表示的游戏内时间间隔。**                                            比如说，当timeScale表示为2时，表示游戏的速度将以2倍速进行，此时假设本属性为0.02，那么当现实过去0.02时，游戏世界已经过去了0.04秒了，FixedUpdate是根据游戏内时间调用的，所以此时，FixedUpdate会被调用两次。 |
| [fixedTime](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-fixedTime.html) | The time since the last FixedUpdate started (Read Only). This is the time in seconds since the start of the game. | 这段话描述了一个名为 `fixedTime` 的公共静态浮点变量，它表示自上一次 `FixedUpdate` 开始以来经过的时间（只读）。这是自游戏开始以来经过的秒数。                  **此值以等于 `Time.fixedDeltaTime` 的固定增量更新** |
| [maximumDeltaTime](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-maximumDeltaTime.html) | 表示任何给定帧中 `Time.deltaTime` 的最大值。这是一个以秒为单位的时间，用于限制两帧之间 `Time.time` 的增加。*重要提示：Unity 强制要求 `maximumDeltaTime` 始终至少与 `Time.fixedDeltaTime` 一样大。* | 当发生非常慢的帧时，`maximumDeltaTime` 会限制下一帧中 `Time.deltaTime` 的值，以避免由于 `deltaTime` 值过大而产生的不良副作用。                                                        推荐的值取决于您的应用程序在帧卡顿发生时所需的特征。 `maximumDeltaTime` 具有以下实际效果：                                                                                                                              1. 将 Unity 在一帧中执行 `MonoBehaviour.FixedUpdate` 的最大次数限制为                      `maximumDeltaTime / fixedDeltaTime`。                                                                             2. 设置 `Time.deltaTime` 的限制值，该值通常用于驱动应用程序的动态部分，例如玩家移动。这控制了应用程序在帧卡顿后是否以及多少程度地卡顿或加速。较低的 `maximumDeltaTime` 值可能会防止在具有长时间 `MonoBehaviour.FixedUpdate` 阶段的应用程序中出现长时间的帧卡顿序列。在这些情况下，长帧会导致下一帧中多次执行 `FixedUpdate` 阶段，从而导致另一个长帧，依此类推。 |
| [unscaledDeltaTime](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-unscaledDeltaTime.html) | 表示与 `timeScale` 无关的从上一帧到当前帧的时间间隔（以秒为单位）（只读）。 | 当从 `MonoBehaviour` 的 `FixedUpdate` 内部调用时，它返回**未缩放的固定帧速率增量时间。**与 `deltaTime` 不同，此值不受 `timeScale` 的影响。 |
| [unscaledTime](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-unscaledTime.html) | 它表示与 `timeScale` 无关的当前帧的时间（只读）。这是自游戏开始以来经过的秒数。 | 当从 `MonoBehaviour` 的 `FixedUpdate` 内部调用时，它返回 `Time.fixedUnscaledTime`                                                                                                     与 `Time.realtimeSinceStartup` 不同，如果在单个帧中多次调用或编辑器暂停时，它返回相同的值。与 `Time.time` 不同，此值不受 `Time.timeScale` 和 `Time.maximumDeltaTime` 的影响。 |
| [realtimeSinceStartup](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-realtimeSinceStartup.html) | `realtimeSinceStartup` 是一个静态浮点型变量，表示从游戏开始到现在的实际时间（以秒为单位），该值只读。这是从应用程序开始到现在的时间（以秒为单位），如果在一帧内多次调用，它不是恒定的。`Time.timeScale` 不会影响此属性。 | 在几乎所有情况下，你都应该使用 `Time.time` 或 `unscaledTime` 代替。如果你想将 `Time.timeScale` 设置为零以暂停应用程序，但**仍然希望能够以某种方式测量时间，那么使用 `realtimeSinceStartup` 是有用的**。在编辑器脚本中，你还可以在编辑器暂停时使用 `realtimeSinceStartup` 来测量时间。                                                               `realtimeSinceStartup` 返回系统计时器报告的时间。根据平台和硬件，它可能会在连续几帧中报告相同的时间。如果你要将某物除以时间差，请考虑这一点（例如，时间差可能变为零）。 |
| [fixedUnscaledDeltaTime](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-fixedUnscaledDeltaTime.html) | The timeScale-independent interval in seconds from the last MonoBehaviour.FixedUpdate phase to the current one (Read Only). | 表示与 `timeScale` 无关的从上一次 `MonoBehaviour.FixedUpdate` 阶段到当前阶段的时间间隔（以秒为单位）（只读）。*与 `Time.fixedDeltaTime` 不同，此值不受 `Time.timeScale` 的影响。* |
| [fixedUnscaledTime](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-fixedUnscaledTime.html) | 表示从游戏开始到上一次 `MonoBehaviour.FixedUpdate` 阶段开始时的时间（以秒为单位），该值只读。它以固定的增量更新，增量等于 `Time.fixedUnscaledDeltaTime` | 与 `Time.fixedTime` 不同，此值不受 `Time.timeScale` 的影响。（原理同Time.fixedUnscaledDeltaTime） |
| [smoothDeltaTime](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-smoothDeltaTime.html) | `smoothDeltaTime` 是一个静态浮点型变量，表示平滑后的 `Time.deltaTime`（只读）。 | 当 `deltaTime` 在帧之间保持恒定（即帧率平滑）时，该值等于 `Time.deltaTime`。当 `deltaTime` 在帧之间变化（例如在帧卡顿时），该值会在多个帧内逐渐增加或减少，直到接近 `deltaTime`。 |
| [frameCount](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-frameCount.html) | `frameCount` 是一个静态整型变量，表示从游戏开始到目前为止的总帧数（只读）。该值从 0 开始，在每次Update阶段增加 1。(每个 Update 阶段都会调用 Update 方法) | 在内部，Unity 使用一个 64 位整数，当调用此函数时，它将其转换为 32 位，并丢弃最高（即顶部）的 32 位。注意：`frameCount` 在所有 `Awake` 函数完成后才开始。在 `Awake` 函数期间，`frameCount` 的值是未定义的。 |
| [inFixedTimeStep](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-inFixedTimeStep.html) | 它的值表示当前是否在固定时间步长回调（例如 MonoBehaviour 的 FixedUpdate）中被调用。如果是，则返回 true，否则返回 false。 | `inFixedTimeStep` 变量可以用来判断当前代码是否在固定时间步长回调（例如 MonoBehaviour 的 FixedUpdate）中被调用。这可以帮助你确定当前的代码执行环境，从而决定是否执行某些操作。                                                                                              举个例子，如果你想在 Update 和 FixedUpdate 中都调用一个函数，但是希望这个函数在 FixedUpdate 中执行不同的逻辑，那么你可以在函数内部使用 `inFixedTimeStep` 变量来判断当前是否在 FixedUpdate 中被调用，从而执行不同的逻辑。 |
| [maximumParticleDeltaTime](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-maximumParticleDeltaTime.html) | 表示一帧中粒子更新所能花费的最大时间。如果一帧的时间超过这个值，那么更新将被分成多个较小的更新。 | **使用较小的值可以提高粒子模拟的质量，但需要更多的处理时间。(更吃性能)**如果帧时间超过所提供的阈值，粒子更新将以较小的时间增量运行多次。                                                                相反，**较高的值确保粒子模拟不会在每帧中分解为多个步骤，从而获得最佳性能**，但在使用一些更高级的粒子模拟功能时会**失去模拟精度**。 |
| [timeSinceLevelLoad](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-timeSinceLevelLoad.html) | 表示从当前场景开始到目前为止的时间（以秒为单位），只读。这意味着，当你加载一个新的场景时，这个变量会被重置为 0，然后随着时间的流逝而增加。 | `timeSinceLevelLoad` 变量是从当前场景加载完成后开始计时的。当你使用 `SceneManager.LoadScene` 函数加载一个新的场景时，Unity 会自动重置这个变量为 0，然后随着时间的流逝而增加。例如，如果你在游戏中加载了一个新的关卡，你可以使用这个变量来计算玩家完成这个关卡所需的时间。 |
| [captureFramerate](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-captureFramerate.html) | The reciprocal of Time.captureDeltaTime.                     | `captureFramerate` 是 `Time.captureDeltaTime` 的倒数。它等于 `(1.0 / Time.captureDeltaTime)` 四舍五入到最接近的整数。                                                            设置 `captureFramerate` 也会将 `Time.captureDeltaTime` 设置为等效的倒数。                                                  请注意，`captureFramerate` 的值为零相当于 `captureDeltaTime` 的值为零。（比如设置captureDeltaTime为1.0f/60，那么captureFramerate将是60s） |
| [captureDeltaTime](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-captureDeltaTime.html) | 减慢应用程序的播放时间，以便Unity在帧之间保存屏幕截图。      | 如果此属性的值不为零，则Time.time将以captureDeltaTime（受Time.timeScale缩放的影响）的间隔增加，**而不考虑实时和帧的持续时间（不考虑Time.deltaTime和Time.fixedDeltaTime。**）。这对于需要恒定帧速率并且希望在帧之间留出足够时间保存屏幕图像的电影非常有用                                                                                                      *注意：captureDeltaTime对Time.unscaledTime没有任何影响。因此，如果您的应用程序的某些部分依赖于它进行动画或其他效果，则captureDeltaTime可能不足以捕获电影。* |
| [timeAsDouble](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-timeAsDouble.html) | 一个双精度浮点数，表示从游戏开始到当前帧开始时的时间（以秒为单位）。`Time.timeAsDouble`属性是只读的。                                       **应避免每帧调用一次：`Time.timeAsDouble`旨在提供应用程序运行时间的长度，而不是每帧的时间。** | **双精度浮点数在表示经过了较长时间的实际时间时具有更高的精度。在几乎所有情况下，您都应该使用`timeAsDouble`而不是`time`。**                                                                         在每一帧的开始时，应用程序都会接收到当前的`Time.timeAsDouble`值，并且该值会在每一帧中递增。在每一帧中调用`timeAsDouble`都会返回相同的值。当从FixedUpdate中调用时，它会返回`Time.fixedTimeAsDouble`属性。                                      **在Awake消息期间，`Time.timeAsDouble`的值是未定义的，并且在所有消息完成后才开始。**如果编辑器暂停，则`Time.timeAsDouble`不会更新。有关不受暂停影响的时间值，请参阅`Time.realtimeSinceStartupAsDouble`。 |
| [fixedTimeAsDouble](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-fixedTimeAsDouble.html) | 它表示自上一次 FixedUpdate 开始以来的双精度时间（只读）。这是自游戏开始以来的秒数 | 固定时间按固定时间间隔（等于 fixedDeltaTime）更新，直到达到 timeAsDouble 属性。此属性是 fixedTime 的双精度版本。**与浮点数或单精度值相比，它在实际时间延长的情况下提供了更高的精度。**在几乎所有情况下，您都应该使用 fixedTimeAsDouble 而不是 fixedTime。 |
| [unscaledTimeAsDouble](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-unscaledTimeAsDouble.html) | unscaledTime 的双精度版本。与浮点数或单精度值相比，它在实际时间延长的情况下提供了更高的精度。在几乎所有情况下，您都应该使用 unscaledTimeAsDouble 而不是 unscaledTime。 | 当从 MonoBehaviour 的 FixedUpdate 内部调用时，它返回未缩放的固定时间。如果在单个帧中多次调用，则返回相同的值。与 timeAsDouble 不同，此值不受 timeScale 影响。 |
| [fixedUnscaledTimeAsDouble](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-fixedUnscaledTimeAsDouble.html) | 上一次 FixedUpdate 开始时的双精度 timeScale 独立时间（只读）。这是自游戏开始以来的秒数。是 fixedUnscaledTime 的双精度版本 | 与 fixedTimeAsDouble 不同，此值不受 timeScale 影响。与浮点数或单精度值相比，它在实际时间延长的情况下提供了更高的精度。在几乎所有情况下，您都应该使用 fixedUnscaledTimeAsDouble 而不是 fixedUnscaledTime。 |
| [realtimeSinceStartupAsDouble](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-realtimeSinceStartupAsDouble.html) | 它表示自游戏开始以来的实时秒数（只读）。这是 realtimeSinceStartup 的双精度版本。与 `realtimeSinceStartup` 相比，这个版本提供了更高的精度，特别是在长时间的现实世界时间内。在几乎所有情况下，都应该使用 `realtimeSinceStartupAsDouble` 而不是 `realtimeSinceStartup`。 | 当您想**将 `Time.timeScale` 设置为零以暂停应用程序，但仍然希望能够以某种方式测量时间时，使用 `realtimeSinceStartupAsDouble` 很有用。**在编辑器脚本中，您也可以使用 `realtimeSinceStartupAsDouble` 来测量编辑器暂停时的时间。                 **`realtimeSinceStartupAsDouble` 返回由系统计时器报告的时间。**根据平台和硬件，**它可能会在连续几帧中报告相同的时间**。如果您要按时间差分配某些东西，请考虑这一点（例如，时间差可能变为零）。 |
| [timeSinceLevelLoadAsDouble](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-timeSinceLevelLoadAsDouble.html) | 表示自当前帧开始以来的双精度时间（只读）。这是自上一个非附加场景加载完成以来的秒数。`timeSinceLevelLoad` 的双精度版本。 | 与单精度值相比，这个版本提供了更高的精度，特别是在长时间的现实世界时间内。在几乎所有情况下，都应该使用 `timeSinceLevelLoadAsDouble` 而不是 `timeSinceLevelLoad`。 |

1. 关于文档中Time.fixedUnscaledDeltaTime中说到“Unlike [Time.fixedDeltaTime](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-fixedDeltaTime.html) this value is not affected by [Time.timeScale](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Time-timeScale.html).”。这句话的实际意思是这样的 

   > That sentence is indeed confusing, and thanks for pointing it out! The documentation is correct but could be elaborated. Time.timeScale controls the overall time scale of the game. It can be used to slow down or speed up the passage of time in the game. Time.fixedUnscaleDeltaTime will occur in intervals based on the real-time passage (independent of Time.timeScale) When you change Time.timeScale, the perceived speed of time in the game is affected. However, while the value of fixedDeltaTime remains the same (unaffected), the number of times it occurs per second (interval) can change based on Time.timeScale.  (来自stack overflow的解释)
   >
   > Time.timeScale 控制游戏的整体时间比例。它可以用来减慢或加快游戏中时间的流逝。
   >
   > Time.fixedUnscaleDeltaTime 将根据实时流逝（独立于 Time.timeScale）以间隔发生。
   >
   > 当您更改 Time.timeScale 时，游戏中时间的感知速度会受到影响。然而，虽然 fixedDeltaTime 的值保持不变（不受影响），但每秒发生的次数（间隔）可以根据 Time.timeScale 改变。
   >
   > 我的理解就是，令Time.scaleTime为2，fixedDeltaTime为0.02，那么当现实世界经过0.02秒之时，游戏世界经过两个0.02秒，执行两次fixedDeltaTime，而只执行一次fixedUnscaledTime，因为现实就经过了0.02秒。换句话说，fixedDeltaTime以游戏世界为参照，fixedUnscaledTime以现实世界为参照

2. Time.unscaledTime与Time.realtimeSinceStartup的区别

   > `realtimeSinceStartup` 和 `unscaledTime` 都表示从游戏开始到现在的实际时间（以秒为单位），并且都不受 `Time.timeScale` 的影响。但是，它们之间有一些细微的差别。
   >
   > [`realtimeSinceStartup` **返回系统计时器报告的时间**，因此它的值会在一帧内多次调用时发生变化。而 `unscaledTime` 的**值在一帧内多次调用时保持恒定**，它返回的是帧开始时的时间](https://gamedevplanet.com/unscaled-time-vs-real-time-since-startup-in-unity/)[1](https://gamedevplanet.com/unscaled-time-vs-real-time-since-startup-in-unity/)。
   >
   > 因此，**如果你想要在一帧内多次获取时间，并且希望每次获取的值都相同，那么应该使用 `unscaledTime`。如果你希望每次获取的值都是最新的，那么应该使用 `realtimeSinceStartup`。**

---

## 重要的类——MonoBehaviour

[重要的类 - MonoBehaviour - Unity 手册](https://docs.unity.cn/cn/2021.3/Manual/class-MonoBehaviour.html)

所有的脚本都默认派生于此类，当你在Unity窗口创建一个脚本的时候，将自动派生自该类

MonoBehaviour类提供了框架，允许你将游戏脚本附加到编辑器中的游戏对象，并提供诸如Start和Update等常用事件的挂钩

MonoBehaviour 类允许您启动、停止和管理协程，这是一种编写异步代码的方法，其中包括等待一定时间或某些操作完成，同时允许其他代码继续执行。

MonoBehaviour 类提供对大量事件消息的访问，允许您根据项目中当前发生的情况执行代码

> 下面是一些比较常见的例子。有关完整的列表，请参阅 [MonoBehaviour 脚本参考页面](https://docs.unity.cn/cn/2021.3/ScriptReference/MonoBehaviour.html) 上的**消息**部分
>
> [`Start`](https://docs.unity.cn/cn/2021.3/ScriptReference/MonoBehaviour.Start.html) - 在游戏对象开始存在时（加载场景或实例化游戏对象时）调用。
>
> [`Update`](https://docs.unity.cn/cn/2021.3/ScriptReference/MonoBehaviour.Update.html) - 每帧都会被调用。
>
> [`FixedUpdate`](https://docs.unity.cn/cn/2021.3/ScriptReference/MonoBehaviour.FixedUpdate.html) - 每个物理时间步进调用。
>
> [`OnBecameVisible`](https://docs.unity.cn/cn/2021.3/ScriptReference/MonoBehaviour.OnBecameVisible.html) 和 [`OnBecameInvisible`](https://docs.unity.cn/cn/2021.3/ScriptReference/MonoBehaviour.OnBecameInvisible.html) - 当游戏对象的渲染器进入或离开摄像机的视图时调用。
>
> [`OnCollisionEnter`](https://docs.unity.cn/cn/2021.3/ScriptReference/MonoBehaviour.OnCollisionEnter.html) 和 [`OnTriggerEnter`](https://docs.unity.cn/cn/2021.3/ScriptReference/MonoBehaviour.OnTriggerEnter.html) - 在发生物理碰撞或触发时调用。
>
> [`OnDestroy`](https://docs.unity.cn/cn/2021.3/ScriptReference/MonoBehaviour.OnDestroy.html) - 在销毁游戏对象时调用。

---

## 重要的类——Transform

Transform 类提供多种方式来通过脚本处理游戏对象的位置、旋转和缩放，以及与父和子游戏对象的层级关系。

- 有关Transform的完整概述：

[变换组件 - Unity 手册](https://docs.unity.cn/cn/2021.3/Manual/class-Transform.html)

> 截取部分理解如下
>
> **Transform** 组件确定每个对象（GameObjcet）在场景中的 **Position**、**Rotation** 和 **Scale** 属性的值。每个游戏对象都有一个变换组件。而每个GameObject对象都有一个Transform属性，Transform属性下发还有三个其他属性
>
> **父子化**：当一个游戏对象是另一个游戏对象的__父__项时，__子__游戏对象完全跟随其父项移动、旋转和缩放。任何一个对象都可以有多个子项，但只有一个父项。这些多级父子关系形成了变换组件的_层级视图。**层级视图最顶层的对象（即层级视图中唯一没有父项的对象）称为__根。**
>
> 将 __Hierarchy 视图__中的任何游戏对象拖到另一个游戏对象上即可创建父项。这样就会在这两个游戏对象之间建立父子关系。

- 有关Transform的脚本细节

[Transform - Unity 脚本 API](https://docs.unity.cn/cn/2021.3/ScriptReference/Transform.html)

---

## 重要的类——Quaternion

[重要的类 - Quaternion - Unity 手册](https://docs.unity.cn/cn/2021.3/Manual/class-Quaternion.html)

Unity 使用 Quaternion 类来存储游戏对象的**三维方向**，也使用它们来描述**从一个方向到另一个方向**的相对旋转

> 有关本主题的基础知识，请阅读 [Unity 中的旋转和方向](https://docs.unity.cn/cn/2021.3/Manual/QuaternionAndEulerRotationsInUnity.html).

> 有关 Quaternion 类的每个成员的详尽参考，请参阅 [Quaternion 脚本参考](https://docs.unity.cn/cn/2021.3/ScriptReference/Quaternion.html)。







---

## Instantiate

> 链接：[Unity - Scripting API: Object.Instantiate](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Object.Instantiate.html)

这玩意应该是用来copy的，它返回一个**Object** The instantiated clone.

了解几个英文：Transform转换矩阵，包含Position, rotation and scale of an object.

就是这么个东西：<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234853.png" alt="image-20230519200531882" style="zoom:50%;" />

几个重载：<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234854.png" alt="image-20230519200846684" style="zoom:50%;" />

关于参数Parameters

original-你想复制的对象；position-新对象的位置；rotation-新物体的角度；parent-分配给新物体的父对象；instantiatedInWorldSpaces是一个布尔值，这是它的解释：

如果将instantiateInWorldSpace设置为true,则新创建的游戏对象将被放置在当前场景的世界空间中。这意味着它的位置、旋转和缩放将与场景中的其他对象相对应。

如果将instantiateInWorldSpace设置为false,则新创建的游戏对象将被放置在当前场景的本地坐标系中。这意味着它的位置、旋转和缩放将相对于场景中的其他对象进行计算，而不是相对于整个世界空间。（在这里是相对它的父对象）

When this method clones a child object, it also clones the child's own children. To prevent stack overflow, Unity limits this nested cloning. If you exceed more than half your stack size, Unity throws an .

Note:

- By default the *parent* of the new object is null; it is not a "sibling" of the original. However, you can still set the parent using the overloaded methods. If a parent is specified and no position and rotation are specified, the original object's position and rotation are used for the cloned object's local position and rotation, or its world position and rotation if the parameter is true. If the position and rotation are specified, they are used as the object's position and rotation in world space.
- The active status of a GameObject at the time of cloning is maintained, so if the original is inactive the clone is created in an inactive state too. Additionally for the object and all child objects in the hierarchy, each of their Monobehaviours and Components will have their Awake and OnEnable methods called only if they are active in the hierarchy at the time of this method call.
- These methods do not create a prefab connection to the new instantiated object. Creating objects with a prefab connection can be achieved using [PrefabUtility.InstantiatePrefab](https://docs.unity.cn/2021.3/Documentation/ScriptReference/PrefabUtility.InstantiatePrefab.html).

```c#
        for (int i = 0; i < 10; i++)
        {
            Instantiate(prefab, new Vector3(i, (3 * i - 2), (2 * i - 1)), Quaternion.identity);
        }//很神奇，比起直接拖拽创建多个prefab，它能通过代码瞬间创建多个prefab，并且在结束游戏后这些实例全都消失了
```

关于Quaternion.identity：当使用Quaternion.identity时，它会返回一个四元数，其w、x、y和z值均为0,这意味着它表示一个指向原点的旋转坐标系。因此，将这个四元数与任何向量相乘都会得到一个相对于坐标系原点的方向和大小的向量。通常情况下，我们会在游戏对象被创建时将其初始化为Quaternion.identity,以便后续对其进行旋转操作时能够方便地指定旋转中心。

当然，如果这个原始角色是一个浮在空中的球，那么将会很有趣，像这个鸟样

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234855.png" alt="image-20230519210954019" style="zoom:50%;" />

再来一段代码



```c#
using UnityEngine;

public class TestPrefab : MonoBehaviour
{
    public Rigidbody rb;
    Vector3 OnePosition;
    Quaternion Rotation;
    // Start is called before the first frame update
    void Start()
    {
        OnePosition = rb.transform.position;
        Rotation=rb.transform.rotation;
    }

    // Update is called once per frame
    void Update()
    {
        //如果你想要检测用户当前是否正在按下某个按钮，应该使用Input.GetButton;
        //如果你想要检测用户之前是否已经按下了某个按钮，应该使用Input.GetButtonDown。
        if (Input.GetButtonDown("Fire1"))//这里有个GetButton和GetButtonDown的区别
         //fire1是按下ctrl，这在 Edit > Project Settings > Input Manager
         //草点击鼠标也是可以的
        {
            //我这里设置的克隆位置和初始对象的一样，就是为了一个撒尿的效果
            Rigidbody clone = Instantiate(rb, OnePosition, Rotation);
            clone.velocity = transform.TransformDirection(Vector3.forward * 10);
        }
    }
}
```

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234856.png" alt="image-20230520083012819" style="zoom:50%;" />

这里就涉及到几个函数了，咱们展开讲一下

---

#### 1.Input.GetButton()与Input.GetButtoDown()

> GetButton链接：[Unity - Scripting API: Input.GetButton](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Input.GetButton.html)
>
> GetButtonDown链接：[Unity - Scripting API: Input.GetButtonDown](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Input.GetButtonDown.html)

如果你想要检测用户当前是否正在按下某个按钮，应该使用Input.GetButton()
如果你想要检测用户之前是否已经按下了某个按钮，应该使用Input.GetButtonDown()

简单来说，GetButton就是用的全自动机关枪，而GetButtonDown是半自动机关枪

这是关于GetButtonDown的英文描述：

> Returns true during the frame the user pressed down the virtual button identified by .`buttonName`
>
> Call this function from the [Update](https://docs.unity.cn/2021.3/Documentation/ScriptReference/MonoBehaviour.Update.html) function, since the state gets reset each frame(since这句是描述update的). **It will not return true until the user has released the key and pressed it again**.（在用户松开前是不会返回true值）

To edit, set up, or remove buttons and their names (such as "Fire1"): 

1. Go to **Edit** > **Project Settings** > **Input Manager** to bring up the Input Manager.
2. Expand **Axis** by clicking the arrow next to it. This shows the list of the current buttons you have. You can use one of these as the parameter "buttonName". 
3.  Expand one of the items in the list to access and change aspects such as the button's name and the key, joystick or mouse movement that triggers it. 
4.  For more information about buttons, see the [Input Manager](https://docs.unity.cn/2021.3/Documentation/Manual/class-InputManager.html) page.

这是全自动GetButton<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032234857.png" alt="image-20230520113727779" style="zoom:50%;" />

这是关于GetButton的描述

> ### Returns
>
> **bool** True when an axis has been pressed and not released.
>
> ### Description
>
> Returns true while the virtual button identified by `buttonName` is held down.
>
> Think auto fire - this will return true as long as the button is held down. Use this only when implementing events that trigger an action, eg, shooting a weapon. The `buttonName` argument will normally be one of the names in [InputManager](https://docs.unity.cn/2021.3/Documentation/Manual/class-InputManager.html) such as Jump or Fire1. [GetButton](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Input.GetButton.html) will return to `false` when it is released.
>
> **Note:** Use [GetAxis](https://docs.unity.cn/2021.3/Documentation/ScriptReference/Input.GetAxis.html) for input that controls continuous movement.

#### 2. RigidBody.velocity

刚体的速度矢量。它表示刚体位置的变化率。

In most cases you should not modify the velocity directly, as this can result in unrealistic behaviour - use AddForce instead (不要直接改速度，会失真)

Do not set the velocity of an object every physics step, this will lead to unrealistic physics simulation. A typical usage is where you would change the velocity is when jumping in a first person shooter（第一人称射击，按下起跳后会有一个向上的速度）, because you want an immediate change in velocity.

在这里我们实际上修改的是每次创造一个物体时赋予该物体的初速度

注意，它是一个世界空间属性

> 这里解释一下什么是世界空间和本地空间
>
> 我们知道脚本依附于物体存在，即如果物体坐标为(3,3,3)，那么在这个脚本里面我设定克隆体的坐标为(2,2,2)
>
> 如果(2,2,2)是世界空间属性，那么在游戏世界中它的坐标就是(2,2,2)
>
> 如果(2,2,2)是本地空间属性，那么在游戏世界中它的坐标就是（2-3，2-3，2-3），即（-1，-1，-1）
>
> 即本地空间属性是相对于当前脚本依附物的坐标而言的

​	

讲到这个世界空间与本地空间，请再允许我插入几个函数进行讲解（*注：它们都是Transform对象的成员函数*）

> 有参数为Vector3和(float x,float y, float z)的三个重载

- **当矢量Vector表示的是坐标**（注意，返回的位置受缩放影响）

  - **InverseTransformPoint**	将位置（坐标）从**世界空间**转换到**本地空间**

    在Unity中，每个对象都有一个本地坐标系和一个世界坐标系。本地坐标系是指对象相对于其父对象或根节点的坐标系，而世界坐标系是指对象相对于场景中心的坐标系。因此，当我们需要在不同的对象之间移动或旋转对象时，需要使用本地坐标系和世界坐标系之间的转换。

    此函数返回一个Vector变量。

    ```C#
    // Calculate the transform's position relative to the camera.
    using UnityEngine;
    using System.Collections;
    
    public class ExampleClass : MonoBehaviour
    {
        public Transform cam;
        public Vector3 cameraRelative;
    
        void Start()
        {
            cam = Camera.main.transform;
            Vector3 cameraRelative = cam.InverseTransformPoint(transform.position);
    
            if (cameraRelative.z > 0)
                print("The object is in front of the camera");
            else
                print("The object is behind the camera");
        }
    }//
    ```

    所有的 Transform 对象都有一个名为 InverseTransformPoint 的成员函数。这个函数用来将世界坐标系中的一个点转换为相对于该 Transform 对象所在坐标系的位置。

  - **TransformPoint** 将位置从**本地空间**转为**世界空间**

    `TransformPoint` 函数用来将一个点从物体的局部坐标系转换到世界坐标系。它的作用与 `InverseTransformPoint` 函数相反。例如，如果你有一个点在物体的局部坐标系中的位置为 `(1, 0, 0)`，你可以使用 `TransformPoint` 函数将这个点转换到世界坐标系中。

    如果一个点没有相对于另一个物体的局部坐标系的位置，那么它就只有一个世界坐标系中的位置。在这种情况下，使用 `TransformPoint` 函数将不会对这个点产生任何影响，因为这个点已经在世界坐标系中了

    ```c#
    using UnityEngine;
    using System.Collections;
    
    public class ExampleClass : MonoBehaviour
    {
        public GameObject someObject;
        public Vector3 thePosition;
    
        void Start()
        {
            // Instantiate an object to the right of the current object
            thePosition = transform.TransformPoint(Vector3.right * 2);
            Instantiate(someObject, thePosition, someObject.transform.rotation);
        }
    }
    ```

    *这两个函数的工作原理是通过矩阵运算来实现的。当你调用这两个函数时，Unity 会根据物体的位置、旋转和缩放计算出一个变换矩阵，然后使用这个矩阵来转换点的位置。这个过程涉及到一些线性代数的知识*

   

- 当矢量Vector表示的是方向的时候（该操作不受变换的缩放或位置的影响， 返回的矢量与 `direction` 的长度相同。）

  同样的，这些都是Transform对象的成员函数（不是Transform的静态函数）

  - TransformDirection 将方向Direction从**本地空间**转为**世界空间**

  - InverseTransformDirection 将方向Direction从**世界空间**转为**本地空间**

    ```c#
    using UnityEngine;
    
    public class Example : MonoBehaviour
    {
        void Start()
        {
            // transform the world forward into local space:
            Vector3 relative;
            relative = transform.InverseTransformDirection(Vector3.forward);
            Debug.Log(relative);
        }
    ```





----

## 事件函数

*此章请结合《Order of execution for event functions》阅读*

> [Unity - Manual: Order of execution for event functions](https://docs.unity.cn/2021.3/Documentation/Manual/ExecutionOrder.html)

### Awake

[Unity - Scripting API: MonoBehaviour.Awake()](https://docs.unity.cn/2021.3/Documentation/ScriptReference/MonoBehaviour.Awake.html)

`Awake`函数在脚本实例被加载时调用。这包括

- 当场景加载时初始化包含脚本的活动游戏对象，
- 当先前非活动的游戏对象被设置为活动
- 使用`Object.Instantiate`创建的游戏对象初始化后，都会调用`Awake`。在应用程序启动前使用`Awake`来初始化变量或状态。

**在脚本实例的生命周期内，Unity仅调用一次`Awake`。**脚本的生命周期持续到包含它的场景被卸载为止。如果重新加载场景，Unity会再次加载脚本实例，因此会再次调用`Awake`。如果以叠加方式加载场景多次，Unity会加载多个脚本实例，因此`Awake`会被调用多次（每个实例一次）。

对于放置在场景中的活动游戏对象，Unity在初始化场景中的所有活动游戏对象后调用`Awake`，因此您可以安全地使用诸如`GameObject.FindWithTag`之类的方法查询其他游戏对象。

**Unity调用每个游戏对象的`Awake`的顺序是不确定的**。因此，您不应依赖一个游戏对象的`Awake`会在另一个游戏对象的`Awake`之前或之后调用（例如，您不应假定由一个游戏对象的`Awake`设置的引用可在另一个游戏对象的`Awake`中使用）。相反，**您应该使用`Awake`在脚本之间设置引用，并使用在所有`Awake`调用完成后调用的 `Start`, 来回传递任何信息。**

**始终先调用 `Awake`, 然后才调用任何 `Start` 函数。**这让您可以对脚本的初始化进行排序。即使脚本是活动游戏对象的禁用组件，也将调用 `Awake`.

**注意：请使用 `Awake`, 而不是构造函数进行初始化，因为组件的序列化状态在构造时是未定义的。 `Awake`, 就像构造函数一样, 只被调用一次。**

> Awake函数确实是在脚本中编写的。当一个脚本对象被初始化时，无论该脚本是否被启用，都会调用该脚本中的Awake函数。**这意味着，即使你禁用了脚本，只要该脚本对象被初始化，Awake函数仍然会被调用。**



### OnEnable

描述只有一句话：This function is called when the object becomes enabled and active.

`OnEnable`函数可以在脚本实例的生命周期内被调用多次。

> 在Unity中，游戏对象可以处于活动状态或非活动状态。当游戏对象处于非活动状态时，它不会被更新或渲染。您可以通过在检查器中取消选中游戏对象的复选框或使用`SetActive`方法来更改游戏对象的活动状态



### Reset

重置为默认值。

当用户在检查器的上下文菜单中点击重置按钮或第一次添加组件时，将调用Reset函数。**这个函数只在编辑模式下被调用。**Reset函数通常用于在检查器中提供良好的默认值。



### OnValidate

**OnValidate是一个仅限编辑器的函数（可以在脚本中调用），当脚本被加载或检查器中的值发生变化时，Unity会调用它。**你可以使用它在检查器中的值发生变化后执行某些操作，例如确保数据保持在某个范围内。

**注意**：**你不应该使用这个回调来执行其他任务**，例如创建对象或调用其他非线程安全的Unity API。你应该只使用它来验证发生变化的数据。**这个限制是因为当用户在编辑器中与检查器交互时，OnValidate可能会被经常调用，而且OnValidate可能会从除Unity主线程之外的其他线程调用，例如加载线程。**

**注意**：你不能可靠地在这里执行相机渲染操作。相反，你应该添加一个监听器到EditorApplication.update，并在下一个编辑器更新调用期间执行渲染。



### Start

**Start函数在脚本启用时的帧上调用，就在任何Update方法第一次调用之前。**

**与Awake函数一样，Start函数在脚本的生命周期中仅被调用一次。**然而，Awake函数在脚本对象初始化时调用，无论脚本是否启用。如果脚本在初始化时未启用，则Start可能不会在与Awake相同的帧上调用。

**Awake函数在场景中所有对象上调用，然后才调用任何对象的Start函数。**这个事实**对于对象A的初始化代码需要依赖于对象B已经初始化的情况很有用**；B的初始化应该在Awake中完成，而A的初始化应该在Start中完成。

当对象在游戏过程中实例化时，它们的Awake函数会在场景对象的Start函数已经完成后调用。(注意是在游戏过程中实例化,这意味着这是游戏中,即其他已存在的场景角色早就已经Start完成了)

Start函数可以定义为协程，这允许Start暂停其执行（yield）。



### OnApplicationPause

当应用程序暂停时，`OnApplicationPause` 会发送给所有 GameObject。

*这里的游戏暂停不等同于Time.timeScale=0*

`OnApplicationPause` 设置为 true 或 false。通常情况下，`OnApplicationPause` 消息返回 false 值。这意味着游戏正在 Editor 中正常运行。**如果选择了某个 Editor 窗口（例如 Inspector），游戏将暂停并且 `OnApplicationPause` 返回 true。**当选中并激活游戏窗口时，`OnApplicationPause` 将再次返回 false。**true 表示游戏未激活。**(即游戏暂停)

**您可以在 Player Settings… 的 Resolution and Presentation 中关闭 Run in Background 和 Visible in Background。**（如果想要在编辑器触发，必须关闭这两个，开始游戏后，来回切换其他窗口，此时就可以护触发了）

> Run In Background 和 Visible In Background 是 Player Settings 中的两个选项，它们控制应用程序在后台运行时的行为。
>
> **Run In Background 选项控制应用程序在后台运行时是否继续执行。**如果此选项被启用，则当应用程序切换到后台时，它仍然会继续执行。如果此选项被禁用，则当应用程序切换到后台时，它会暂停执行。
>
> **Visible In Background 选项控制应用程序在后台运行时是否可见。**如果此选项被启用，则当应用程序切换到后台时，它仍然会在屏幕上显示。如果此选项被禁用，则当应用程序切换到后台时，它会从屏幕上消失。
>
> 这两个选项可以根据您的需要进行设置。如果您希望应用程序在后台运行时暂停执行并从屏幕上消失，则可以将这两个选项都禁用。

`OnApplicationPause` 可以在独立于 Editor 运行的游戏中使用。该游戏需要以小于全屏的窗口模式运行。如果游戏被其他应用程序（全部或部分）遮挡，`OnApplicationPause` 将返回 true。当游戏重新变为当前窗口时，它将取消暂停状态，并且 `OnApplicationPause` 返回 false。

`OnApplicationPause` 可以作为协同程序使用 - 在函数中使用 yield 语句即可。以这种方式实现时，将在初始帧期间对其计算两次：第一次是早期通知，第二次在正常的协同程序更新步骤期间进行。

在 Android 上，当启用屏幕键盘时，它会导致 OnApplicationFocus(false) 事件。此外，如果您在启用键盘时按 “Home” 键，则不会调用 OnApplicationFocus() 事件，而改为调用 OnApplicationPause()。

注意：MonoBehaviour.OnApplicationPause 接收 true 或 false。您无法调用该消息。此外，键盘/鼠标等也无法控制 MonoBehaviour.OnApplicationPause。暂停意味着游戏正常运行或已被暂停。

注意：MonoBehaviour.OnApplicationPause 在 GameObject 启动时调用。该调用在 Awake 之后进行。每个 GameObject 都会导致该调用。



### FixedUpdate

这段话描述了MonoBehaviour.FixedUpdate在物理计算中的作用。

**FixedUpdate的频率与物理系统相同，每个固定帧率帧都会调用它。**

在FixedUpdate之后计算物理系统。默认情况下，两次调用之间的时间为0.02秒（每秒50次调用），这也是Time.fixedDeltaTime的默认值。可以通过访问Time.fixedDeltaTime来获取这个值。可以通过在脚本中将其设置为您喜欢的值或导航到Edit>Settings>Time>Fixed Timestep并在那里设置来更改它。

FixedUpdate的频率可能多于或少于Update。如果应用程序以每秒25帧（fps）运行，则Unity大约每帧调用两次，或者，100 fps会导致大约两个渲染帧与一个FixedUpdate。可以从时间设置中控制所需的帧速率和固定时间步速率。**使用Application.targetFrameRate设置帧速率。**

**当使用Rigidbody时，请使用FixedUpdate。**为Rigidbody设置力，它将在每个固定帧应用。FixedUpdate发生在一个测量的时间步长上，通常不与MonoBehaviour.Update重合。



### Update

如果MonoBehaviour脚本处于启用状态，那么Update函数将在每一帧中被调用。Update函数是实现游戏脚本的最常用函数。但并不是每一个MonoBehaviour脚本都需要Update函数。



### LateUpdate

如果启用了 [Behaviour](https://docs.unity.cn/cn/current/ScriptReference/Behaviour.html)，则每帧调用 LateUpdate。

LateUpdate 在调用所有 Update 函数后调用。 这对于安排脚本的执行顺序很有用。例如，跟随摄像机应始终在 LateUpdate 中实现， 因为它跟踪的对象可能已在 Update 中发生移动。



### OnPreCull

**OnPreCull是一个事件函数，它会在摄像机剔除场景之前被Unity调用。**

在内置渲染管线中，Unity会在执行 ”决定摄像机能够看到什么的“ 剔除操作之前，**调用 附加到与启用的摄像机组件相同的游戏对象 上的MonoBehaviours上的OnPreCull**。您可以使用OnPreCull在渲染循环的这个点执行您自己的代码；例如，在**执行剔除操作之前更改摄像机的设置，以影响摄像机所看到的内容**。OnPreCull可以是一个协程。

> 当然可以。举个例子，假设您正在玩一个3D游戏，游戏中有一个摄像机，它负责捕捉游戏世界中的画面并显示在您的屏幕上。但是，游戏世界中可能有很多物体，而摄像机并不需要显示所有的物体。例如，如果一个物体被其他物体挡住了，或者它离摄像机太远，那么这个物体就不需要被显示出来。
>
> 为了优化性能，**Unity会在渲染画面之前执行一个剔除操作，来决定哪些物体需要被显示，哪些物体不需要被显示**。OnPreCull事件就是在这个剔除操作执行之前被调用的。您可以在OnPreCull事件中执行您自己的代码，例如更改摄像机的设置，以影响剔除操作的结果。
>
> **渲染画面通常是在每一帧中执行一次。也就是说，每当游戏画面更新时，Unity都会执行一次渲染操作来绘制新的画面。在这个过程中，剔除操作会在渲染操作之前执行。**

```csharp
using UnityEngine;

public class CameraController : MonoBehaviour
{
    public float zoomSpeed = 10f;
    private Camera cam;

    void Start()
    {
        cam = GetComponent<Camera>();
    }

    void OnPreCull()
    {
        // Zoom in/out when the user scrolls the mouse wheel
        float scroll = Input.GetAxis("Mouse ScrollWheel");
        cam.fieldOfView -= scroll * zoomSpeed;
    }
}
```

在这个示例中，我们首先获取了附加到游戏对象上的摄像机组件。然后，在OnPreCull事件中，我们检查用户是否滚动了鼠标滚轮，并根据滚动的方向和速度更改摄像机的视野角。这样，当用户滚动鼠标滚轮时，摄像机就会放大或缩小画面。

对于不需要脚本位于与摄像机组件相同的游戏对象上的类似功能，请参阅[Camera.onPreCull](https://docs.unity.cn/cn/2021.3/ScriptReference/Camera-onPreCull.html). 

对于可编程渲染管线中的类似功能，请参阅[RenderPipelineManager](https://docs.unity.cn/cn/2021.3/ScriptReference/Rendering.RenderPipelineManager.html).



### OnBecameVisible 和 OnBecameInvisible

[`OnBecameVisible` 和 `OnBecameInvisible` 是两个非常有用的函数，它们可以帮助你避免在对象不可见时进行不必要的计算。当渲染器不再被任何摄像机看到时，将调用 `OnBecameInvisible` 函数。这条消息将发送给附加到渲染器的所有脚本。当对象再次可见时，将调用 `OnBecameVisible` 函数

例如，下面是一个简单的脚本，它在对象可见时启用行为，而在对象不可见时禁用行为：

```csharp
using UnityEngine;
using System.Collections;

public class ExampleClass : MonoBehaviour {
    void OnBecameVisible() {
        enabled = true;
    }
    void OnBecameInvisible() {
        enabled = false;
    }
}
```



### OnWillRenderObject(不懂以后再看)

> 如果对象可见并且不是 UI 元素，则为每个摄像机调用 OnWillRenderObject。（这里的对象指的是附加了脚本的游戏对象，每个摄像机指的是场景中的每个摄像机。）
>
> 如果禁用了 MonoBehaviour，则不会调用该函数。
>
> The function is called during the culling process just before rendering each culled object.

它光这么说我还是可以理解的，当然就是有些听不懂就是了

[MonoBehaviour-OnWillRenderObject() - Unity 脚本 API](https://docs.unity.cn/cn/2021.3/ScriptReference/MonoBehaviour.OnWillRenderObject.html)

- OnPreRender

  在摄像机开始渲染场景之前调用

  文档解释了一大堆东西，自己看吧[MonoBehaviour-OnPreRender() - Unity 脚本 API](https://docs.unity.cn/cn/2021.3/ScriptReference/MonoBehaviour.OnPreRender.html)

- OnRenderObject

  所有常规场景渲染完成之后，会调用`OnRenderObject`事件。

  [MonoBehaviour-OnRenderObject() - Unity 脚本 API](https://docs.unity.cn/cn/2021.3/ScriptReference/MonoBehaviour.OnRenderObject.html)

- **OnPostRender**

  在摄像机完成场景渲染后调用

  [MonoBehaviour-OnPostRender() - Unity 脚本 API](https://docs.unity.cn/cn/current/ScriptReference/MonoBehaviour.OnPostRender.html)

- **OnRenderImage**

  在场景渲染完成后调用

  [MonoBehaviour-OnRenderImage(RenderTexture,RenderTexture) - Unity 脚本 API](https://docs.unity.cn/cn/current/ScriptReference/MonoBehaviour.OnRenderImage.html)

- OnGui

  最后，在每帧中，会多次调用`OnGUI`事件以响应GUI事件。首先处理布局和重新绘制事件，然后为每个输入事件处理布局和键盘/鼠标事件。

  [MonoBehaviour-OnGUI() - Unity 脚本 API](https://docs.unity.cn/cn/current/ScriptReference/MonoBehaviour.OnGUI.html)

- OnDestroy

  [MonoBehaviour-OnDestroy() - Unity 脚本 API](https://docs.unity.cn/cn/current/ScriptReference/MonoBehaviour.OnDestroy.html)

- OnApplicationQuit

  [MonoBehaviour-OnApplicationQuit() - Unity 脚本 API](https://docs.unity.cn/cn/current/ScriptReference/MonoBehaviour.OnApplicationQuit.html)

- OnDisable

  [MonoBehaviour-OnDisable() - Unity 脚本 API](https://docs.unity.cn/cn/current/ScriptReference/MonoBehaviour.OnDisable.html)

---


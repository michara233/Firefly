# unity

## 生命周期函数：

### Awake	类似构造函数，对象出现时只调用一次

### OnEnable	对象每次激活时调用

### Start	从创建到第一次帧更新前调用，仅调用一次

### FixedUpdate	物理帧更新

### Update	逻辑帧更新

### LateUpdate	于Update后执行（常用于摄像头更新）

### OnDisable	失活时调用

### OnDestroy	对象销毁时调用

## 检查器（inspector）

### 1.私有与保护变量无法在检查器修改

#### 如需修改，在变量上一行加上[SerializeFoeld]

### 2.公共的默认可显示在检查器

#### 如需修改，在变量上行加上[HideInInspector]

#### TIP：字典，自定义结构体，自定义类不可显示

### 3.让自定义的类型可显示

#### 变量上一行加上[System.Serializable]

### 4.辅助用特性

#### [header("分组说明")]	分组说明（为变量添加组名）

#### [Tooltip("说明内容")]	悬停注释（在鼠标停在对应变量上时显示）

#### [Space( )]	间隔特性（空行）

#### *[Range( , )]	滑条范围（让值通过滑条调整）

#### *[Multiline()]	多行显示（让字符串可分行显示）

#### [TextArea( , )]	滚动显示字符串（默认前变量行数，超过滚动至后变量行）

#### [ContextMenuItem("显示按钮名","方法名(无返回值，无参)")]	（给变量添加一个右键可用的方法）

#### [ContextMenu("测试函数")]	快捷测试函数	（在更多里测试函数）

## MonoBehaviour类

### 重要成员（获取成员）

#### 获取依附的gameobject

this.gameObject.name

#### 获取依附的GameObjec的位置信息

this.transform.position

this.transform.eulerAngles

this.transform.lossyScale

#### 获取脚本是否激活

this.enabled = false;	失活

this.enabled = true;	激活

### 重要方法（脚本获取）

#### 得到自己挂载的单个脚本

##### 1.根据脚本名获取

父类名 变量名=this.GetComponent("脚本名") as 父类名

##### 2.根据Type获取

父类名 变量名 = this.GetComponent(typeof(父类名)) as 父类名

##### 3.根据泛型获取

父类名 变量名 = this.GetComponent<父类名>();

#### 得到自己挂载的多个脚本

父类名[] 变量名=this.GetComponents<父类名>();

 List<父类名> 变量名 = new List<父类名>();

this.GetComponents<父类名>(List);

#### 得到子对象挂载的脚本

##### 单个

this.GetcomponentInChildren<子类名>(此处可填true或空);

##### 多个

this.GetcomponentsInChildren<子类名>(同上，List变量名);

解释：填true会查找失活与激活，否则只查激活

#### 得到父对象挂载的脚本

##### 单个

this.GetcomponentInParent<父类名>();

##### 多个

this.GetcomponentsInParent<父类名>(List变量名);

#### 尝试获取脚本

this.TryGeyComponent<父类名>(out 变量名)	(值为true or false)

## GameObject类

### GameObject中的成员变量

#### 名字

this.gameObject.name="  	"

#### 是否激活

this.gameobject.activeSelf

#### 是否是静态

this.gameobject.isStatic

#### 层级

this.gameobject.layer

#### 标签

this.gameobject.tag

#### transform

this.gameobject.tranform

### GameObject中的静态方法

#### 创建几何体

GameObject.CreatePrimitive(PrimitiveType.几何体)

也可

GameObject 变量名 =GameObject.CreatePrimitive(PrimitiveType.几何体)

#### 查找对象

##### 查找单个对象

GameObject.Find("对象名");

或者（以上更低效）

GameObject.FindWithTag("Tag名");

**（无法找到激活对象）**

（**无法准确确定找到的对象**）

##### 查找多个对象

GameObject[] objs = GameObject.FindGameObjectsWithTag("tagmin")

### 实例化对象

GameObject.Instantiate(对象名)

### 删除对象

#### 	删除对象

​	GameObeject.Destory(对象名,延迟几秒)

#### 	删除脚本

​	***GameObject.Destory(this)**

TIPS:一般会在下一帧才移除（常用）

​	GameObject.DestroyImmediate(对象名)

可在当前帧移除

**如继承了Moonbehaviour可直接省略GameObject**

### 切换场景不会移除

保留附着这个脚本的对象

GameObject.DontDestroyOnLoad(this.gameObject)

**如继承了Moonbehaviour可直接省略GameObject**

### GameObject中的成员方法

#### 创建空物体

GameObject 对象名 = new GameObject()

GameObject 对象名 = new GameObject(对象命名,typeof(脚本名),typeof(......))

**可创建空的，改名的空的，带脚本的空的**

#### *为对象添加脚本

对象名.AddComponent<脚本名>();

#### 判断对象

this.gameObject.CompareTag("标签名")

#### 设置激活失活

对象名.SetActive(false or true);



**以下一般不用**

#### 通知命令

this.gameObject..SendMessage("函数名")

#### 广播行为

this.gameObject.SendMessageUpwards("函数名")

## Time时间相关 

### *时间缩放比例

#### 时间暂停

Time.timeScale = 0;

#### 恢复正常

Time.timeScale = 1;

#### 2倍速

Time.timeScale =2;

### *帧间隔时间

用于控制位移

路程=速度乘时间

#### 受scale影响

Time.deltaTime

**受时间速率影响**

#### 不受scale影响

Time.unscaleDeltaTime

**不受时间速率影响**

### 游戏开始到现在的时间

**主要用于计时**

#### 受scale影响

Time.time

#### 不受scale影响

Time.unscaledTime

### 物理帧间隔时间

#### 受scale影响

Time.fixedDeltaTime

#### 不受scale影响

Time.fixedUnscaledDeltaTime

### *帧数

Time.frameCount

## 位置和位移

### Vector3

本身是一个结构体

Vector3 变量名 =new Vector3（x,y,z）;

### Vector的基本运算

将两  个vector3的x，y，z相运算

### 常用

#### 常量

Vector3.zero 	原点

Vector3.right	1 0 0

Vector3.left	-1 0 0

Vector3.forward	0 0 1

Vector3.back	0 0 -1

Vector3.up	0 1 0

Vector3.down	0 -1 0

#### 方法

**计算两个点的距离**

Vector3.Distance（丁1，点2）

### 位置

#### 相对世界坐标系

this.gameObject.transform

this.transform.position（得到的是世界坐标系位置，但面板上显示为相对父对象位置）

#### 相对父对象

this.transform.localPosition(得到面板位置)

.position无法改变单独x,y,z

#### 如何改变position

this.transform.position =new Vector3(19,this.transform.position.y(原坐标))

**或者单独用一个Vector3变量存储再修改再赋值**

#### 对象当前的各朝向

**对象的面朝向**

this.transform.forward

**对象的头顶向**

this.transform.up

**对象的右手边**

his.transform.right

### 位移

 **路程=方向* *速度* * 时间**

#### 计算方法

##### 一。自己算

this.transform.position=this.transform.po sition+this.transform.forward*1*Time.deltaTime;

##### 二。API

参数一：位移多少	路程=方向*速度*时间

参数二：相对坐标系	默认相对自身坐标

this.transform.Translate(Vecor3.forward*1*Time.deltaTime,Space.world(世界坐标系))

## 角度和旋转

#### 角度

##### 相对世界坐标角度

this.transform.eulerAngles

##### 相对父坐标

this.transform.localEulerAngles

#### 修改角度（同修改位置）

### 旋转相关

#### 自传

往x转（抬头低头）

往y转（左右摇头）

往z转（偏头）

this.transform.Rotate(new Vector3(0,10,0)*Time.deltaTime,相对坐标系)

##### 相对魔个轴旋转

this.transform.Rotate(Vector3.right,选张,相对坐标轴)

##### 相对某个点转

参数一：绕的点	参数二：绕那个轴	参数三：度数

this.tranform.RotateAround(Vector3.zreo,Vector3.right，10*Time.deltaTime)

## 缩放和看向

### 缩放

#### 相对世界坐标系

this.transform.lossyScale

（只能得到不能修改）

#### 相对本地坐标系

this.transform.localScale



TIPS:缩放在unity中无API,想要逐渐调整缩放需要自行计算

this.transform.localscale+=vector3(1,1,1)*deltatime

### 看向

//让物体一直面向坐标/物体

this.transform.LookAt(坐标)



public tranform obj;

this.transform.LookAt(obj)

## 父子关系

### 获取设置父对象

#### 获取

this.transform.parent

#### 移除

this.transform.parent=null

#### 修改

this.transform.parent=GameObject.Find("name").transform

#### 通过API修改

this.transform.SetParent(object.transform,**true**)

参数一：父对象

参数二：是否保留当前世界坐标，缩放，旋转

### 抛妻弃子

this.transform.DetachChildren()

### 获取子对象

#### 按名字查找儿子

this.transform.find("name")

TIPS:能够找到失活对象

#### 遍历儿子

this.transform.Getchild(索引)

### 子的操作

判断自己是否为另一个对象的子对象

if(son.IsChildof(this.transform))

得到自己作为子对象的编号

son.Getsiblingindex()

把自己设置为第一个子对象

son.Setasfirstsibling()

把自己设置为最后一个子对象

son.setaslastsibling()

把自己设置为指定个儿子

son.setsiblingindex(数字)

## 坐标转换

### 世界坐标转本地坐标

#### 世界坐标的点 转 本地坐标的点

transform.InverseTransformPoint(坐标点)

将该点转换为相对挂载对象坐标系上的本地坐标位置

transform.InverseTransformDirection(坐标点)

方向

### 本地坐标系 转 世界坐标系

点

this.tranform.TransformPoint(向量坐标)

方向

this.transform.TransformDirection(向量坐标)



## 鼠标键盘输入

### 鼠标在屏幕位置

Input.mousePosition

//屏幕的原点为左下角，右侧是x正方形，，上侧为y正方向

//返回时为vector3，，但是z=0

### 检测鼠标输入

//鼠标按下：0左键，1右键，2中键

Input.GetMouseButtonDown(0/1/2) 

//鼠标抬起：

Input.GetMouseButtonUp(0/1/2)

//鼠标长按按下抬起都会触发

Input.GetMouseButton(0/1/2)

//中键滚动-1下 0无  1上

Input.mouseScrollDelta(-1/0/1)

### 检测键盘输入

//键盘按下 

Input.GetKeyDown(KeyCode.w)	//w键按下

写法2：

Input.GetKeyDown(""w")	//填大写报错

//键盘抬起

Input.GetKeyUp(KeyCode.w)

//键盘长按

Input.GetKey(KeyCode.W)

### 检测默认轴输入

//键盘AD按下时 返回 -1到1的变换

Input.GetAxis("")	//字符串内容为edit->project setting ->input Manager->Assts下的所有热键名

### 其他输入

#### //是否有任意键或鼠标长按

Input.antKey

#### //是否有任意键或鼠标按下

Input.antKeyDown

#### //这一帧的键盘输入

Input.inputString

#### //得到链接的手柄的所有按钮名字

string[] strs = Input.GetJoystickNames()

**//检测字符串为“jump”......**

#### 移动设备触摸相关

举例

if(Input.touchCount > 0)	//触摸数大于0

{

​	Touch t1 = Input.touches[0];

}

#### 是否启用多点触控

Input.multiTouchEnabled = false;

#### 陀螺仪

Input.gyro.enabled = true;	//是否开启陀螺仪

重力加速度向量

Input.gyro.gravity

........

## Screen屏幕相关

### 静态属性

##### 当前分辨率

Resolution r = Screen.currentResolution;

print(r.width+r.height)	//得到当前屏幕分辨率的宽高

##### 屏幕窗口当前宽高

Screen.width	//游戏窗口的宽高

##### 屏幕休眠模式

Screen.dleepTimeout = SleepTimeout.NeverSleep	//永不熄屏

##### 运行时是否全屏

Screen.FullScreenMode = FullscreeMode.fullScreenWindow

##### 移动设备屏幕转向相关

(不常用,,需要再搜)......

### 静态方法

#### 设置分辨率

Screen.SetResolution(1920,1080,false//是否全屏)

## Camera摄像头参数说明

### 	clear Flags

skybox	天空盒	//常3D

solid Color	颜色填充	//3&2D

Depth only	只画该层，背景透明

Don,t Clear	不移除，覆盖渲染

### Culling Mask	选择性渲染部分层级

### Projection	

#### Perspective	透视模式

Fov Axis	视场角（决定是竖直方向还是水平方向计算视口）

Field of View	视口大小

Physical Camera 模拟物理（真实）相机的参数

#### orthgraphic 正交摄像机（常用于2D）

Size	摄制范围

Clipping Planes	裁剪平面距离（可渲染的范围）

### Depth	渲染顺序上的深度

值越大越后被渲染，遮盖掉之前渲染的画面

用法：两个摄像头，调整Clear Flags如图，渲染的层级不同，用来叠加画面

//常用于UI和游戏画面的分开渲染

![image-20260721182746055](C:\Users\michara\AppData\Roaming\Typora\typora-user-images\image-20260721182746055.png)

### Target Textrue	渲染纹理

作用：将摄像机画面渲染到一张图上

用法：右键创建Render Texture 把节点拖进去

### Occlusion Culling	剔除遮挡

被挡住的物体不会被渲染

### Viewport Rect	视口范围

用于多视角，修改xywd改变显示在窗口的位置与宽高

#### 其他

**Allow HDR**

开启后相机支持高动态光照渲染，可以呈现更大明暗反差、光晕、泛光等特效；关闭则使用普通 LDR 渲染，性能开销更低。

**Allow MSAA**

开启多重采样抗锯齿，消除物体边缘锯齿；移动端 / 性能紧张场景可关闭节省性能。

**Allow Dynamic Resolution**

允许引擎动态调整渲染分辨率，在设备负载过高时自动降低分辨率维持帧率。

**Target Display**

指定相机画面输出到几号显示器，多用于多屏输出、街机、多屏幕主机游戏开发。

## Camera代码相关

### 重要静态成员

#### 1.获取摄像机

Camera.main(需要摄像机的Tag为Main Camera)

#### 2.获取摄像机的数量

Camera.allCamerasCpunt

#### 3.得到所有摄像机

camera[] allCamera = Camerea.allCameras;

#### 4.渲染相关委托

//摄像机剔除前处理的委托

Camera.onPreCull +=(c)=>

{

},

//摄像机渲染前的委托

Camera.onPreRender+=(c)=>

{

};

//摄像机渲染后的委托

Camera.onPostRender+=(c)=>

{

};

### 重要成员

#### 1.Camera.界面上的参数 都可以在Camera中获取到

Camera.main.depth = 10;

#### 2.世界坐标转屏幕坐标

Vector3 v = Camera.main,WorldToScreenPoint(this.transform.position)

//可用于制作血条，3d世界人物头顶那种

#### 3.屏幕坐标转世界坐标

Vector3 v = Input.mousePosition;

v.z = 5;

Camera.maiin.ScreenToWorldPoint(v)

//在横截面v距离的位置移动

## 光源组件

### Type

#### spot	聚光灯

#### Directional	方向灯（环境光）

#### Point	点光源

#### Area	面光源

### color	颜色

### Mode	模式

#### RealTime	实时光源	//效果好，性能消耗大

#### Baked	烘焙光源	//实现计算好，无法动态变化

#### Mixed	混合光源	以上相加

### Intensity	光源亮度

### Shadow Type

NoShadows	关闭阴影

HardShadows	生硬阴影

SoftShadows	柔和阴影

### Cookie	投影遮罩

Draw Halo	球形光环开关（蜡烛的光晕）

Flare	耀斑（添加Flare 组件，实现光中的小光圈效果）

### Culling Mask	剔除遮罩层

勾选的层级才会被光影响

### 其他......

## 光源的代码控制

light.参数

## 光相关面板

（Window->Rendeing->Lighting Settings）

### Environment

#### **Skybox** **Material**	可以改变天空盒材质

#### Sun Source	太阳来源

#### Environment Lighting	环境光设置：......

### 其他

......

## 碰撞检测

**碰撞产生的必要条件：两个物体都有碰撞体，至少一个物体有刚体**

### 刚体

（Rigid body	让物体受到力的效果）

##### RigidBody 组件信息

Mass

质量（默认为千克）

质量越大惯性越大

Drag

空气阻力

根据力移动对象时影响对象的空气阻力大小

0 表示没有空气阻力

Angular Drag

根据扭矩旋转对象时影响对象的空气阻力大小。0 表示没有空气阻力。

Use Gravity

是否受重力影响

Is Kinematic

如果启用此选项，则对象将不会被物理引擎驱动，只能通过 (Transform) 对其进行操作。对于移动平台，或者如果要动画化附加了 HingeJoint 的刚体，此属性将非常有用。

Interpolate

插值运算

让刚体物体移动更平滑

Collision Detection（碰撞检测模式）

用于防止快速移动的对象穿过其它对象而不检测碰撞

Constraints

约束

对刚体运动的限制

### 碰撞器

#### 1.3D碰撞器种类

盒状

球状

胶囊

网格

轮胎

地形

#### 2.共同参数

##### Is trigger：

是否是触发器，如果启用，则该碰撞体用于检测碰撞，无物理效果

##### Material：

物理材质，用于确定碰撞体和其他对象碰撞的交互表现

##### Center：

碰撞体在对象局部空间的中心点位置

#### 3.常用碰撞体

##### 盒状：

SIze：在xyz上的大小

##### 球状：

Radius：球形的半径大小

##### 胶囊：

Radius：胶囊的半径

Height：胶囊高度

Direction：在对象局部空间中的轴向

#### 4.**异形物体使用多种碰撞器组合**

子对象会继承父对象的刚体，由此组合一个复杂物体的碰撞体

#### 5.不常用碰撞器

网格，地形（性能开销大）

轮胎（环形）不常用

##### 网格：

根据物体的面设置碰撞判断

Convex：勾选才能使用刚体（最多255面计算）



### 材质

##### 参数

###### Dyanamic Friction

已在移动时使用的摩擦力

###### Static Friction

静止时的摩擦力（越大越难移动）

###### Bounciness

表面弹性（越小弹力越小）

###### Friction Combine

（两个对象摩擦力的组合方式）

Average（取平均）

Minimum（取最小）

Maximum（最大值）

Multiply（相乘）

###### unce Combine

弹性组合方式，类似于碰撞摩擦力组合方式

### 刚体加力

#### 1.刚体自带添加力的方法

##### A.获取刚体组件

Rigidbody nody = this.GetComponent<Rigibody>();

##### B.添加力

###### 相对世界坐标系

body.AddForce(Vector3.forward*10)

###### 相对本地坐标

body.AddRelativeForce(Vector3.forward*10)

##### C.添加扭矩力，使其旋转

###### 相对世界坐标

body.AddTorque(Vector3.up * 10)

###### 相对本地坐标

body.AddRelativeTorque(Vector.up *10)

##### D.直接改变速度

body.velocity = vector3.forward *5;

##### E.模拟爆炸效果

body.AddExplosionForce(10,vector3.zero,10)	//力，中心点，半径

（只有挂载写了的此句的才能受到爆炸影响）

#### 2.力的几种模式

body.Addforce(vector3.forward * 10,ForceMode.(mode))

(mode)如下：

Acceleration

Force

Impulse

VelocityChange

#### 3.立场脚本

Constant Force

#### 补充：

**刚体休眠：**

......

## 音效系统

### 音频文件

#### AudioClip 音频导入参数

Force To Mono

多声道转单声道      Normalize

强制为单声道时，混合过程中被标准化

Load In Background

在后台加载，不阻塞主线程

Ambisonic

立体混响声

非常适合 360 度视频和 XR 应用程序

如果音频文件包含立体混响声编码的音频，请启用此选项

LoadType 加载类型

Decompress On Load

不压缩形式存在内存，加载快，但是内存占用高    适用于小音效

Compress in memory

压缩形式存在内存，加载慢，内存小    仅适用于较大音效文件

Streaming

以流形式存在，使用时解码。内存占用最小，cpu 消耗高    性能换内存

Preload Audio Data

预加载音频，勾选后进入场景就加载，不勾选，第一次使用时才加载

Compression Format 压缩方式

PCM

音频以最高质量存储

Vorbis

相对 PCM 压缩的更小，根据质量决定

ADPCM

包含噪音，会被多次播放的声音，如碰撞声

------

### 音频源与监听脚本

#### AudioSource 音频源基础参数

AudioClip

声音剪辑文件（音频文件）

AudioSource 音频源

Output

默认将直接输出到场景中的音频监听器

可以更改为输出到混音器

Mute

静音开关

Bypass Effect

开关滤波器效果

Bypass Listener Effects

快速开关所有监听器效果

Bypass Reverb Zones

快速开关所有混响区

Play On Awake

对象创建时就播放音乐

也就是游戏启动物体激活后自动播放

Loop

循环

Priority

优先级

Volume

音量大小

Pitch

音高

------

#### AudioSource 3D Sound Settings 3D 音效设置

Stereo Pan

2D 声音立体声位置

相当于左右声道

Spatial Blend

音频受 3D 空间的影响程度

Reverb Zone Mix

到混响区的输出信号量

3D Sound Settings

和 Spatial Blend 参数成正比应用

Doppler Level

多普勒效果等级

Spread

扩散角度设置为 3D 立体声还是多声道

Volume Rolloff 声音衰减速度

Logarithmic Rolloff

靠近音频源时，声音很大，但离开对象时，声音降低得非常快。

Linear Rolloff

与音频源的距离越远，听到的声音越小。

Custom Rolloff

音频源的音频效果是根据曲线图的设置变化的。

Min/Max Distance

最小距离内，声音保持最大响度

最大距离外，声音开始减弱

#### Audio Listerner	监听脚本

**常挂载于主摄像头**



### 代码控制音频



#### 代码控制播放停止

AudioSource audio;//定义

audio.play()	//播放

audio.stop()	//停止

audio.Pause()	//暂停

audio.UnPause()	//停止暂停（播放）

audio.playDelayed(秒数)	//延迟播放



#### 如何检测音效播放完毕

audio.isPlaying	//是否正在播放（true是，反之）



#### 如何动态控制音效播放

##### 1.直接在需播放的对象上挂载脚本

##### 2.实例化挂载了音效原脚本的对象

##### 3.用一个AudioSource来控制不同的音效

public AudioClip clip;	//定义切片

AudioSource aus =this.gameObject.addComponent<AudioSource>();	//新增音效

aus.clip = clip	//获取当前对象上的切片

aus.play()'	//播放切片



### 麦克风输入相关

#### 获取设备麦克风信息

#### 开始录制

#### 结束录制

#### 获取音频数据用于存储或者传输

Microphone.devices	//麦克风信息（字符型）

#### 开始录制

Microphone.Start(null(设备名),false(超过录制长度后是否重头录制),10(录制时长),44100(采样率));

#### 结束录制

Microphone.End(null(设备名))

#### 获取音频数据用于存储或者传输

float[] f=new float[clip.channels * clip.samples];	//声道数*剪辑长度


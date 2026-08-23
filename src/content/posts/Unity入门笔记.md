---
title: Unity 入门阶段笔记 Stage 1
published: 2026-07-20
updated: 2026-08-01
description: Unity 常用 API 速查 — 生命周期、Transform、Input、Camera、物理、光照、音频等
tags: [unity, 游戏开发, 基础知识, 速查]
category: 基础知识
slug: unity-core-scripting-cheatsheet
pinned: true
---

# Unity 入门阶段笔记 Stage 1

## 一、MonoBehaviour 生命周期函数

| 函数 | 执行时机 | 说明 |
|------|----------|------|
| `Awake` | 对象出现时（实例化） | 类似构造函数，只调用一次 |
| `OnEnable` | 对象每次激活时 | 每次激活都调用 |
| `Start` | 从创建到第一次帧更新前 | 仅调用一次 |
| `FixedUpdate` | 固定物理帧率 | 物理帧更新 |
| `Update` | 每渲染帧 | 逻辑帧更新 |
| `LateUpdate` | 于 Update 之后执行 | 常用于摄像头更新 |
| `OnDisable` | 失活时 | 失活时调用 |
| `OnDestroy` | 对象销毁时 | 对象销毁时调用 |

> **补充规则：**
> 1. 场景加载时所有物体先统一执行完 `Awake`，再统一执行 `Start`
> 2. `SetActive(false)` 仅触发 `OnDisable`，不会调用 `OnDestroy`
> 3. `Destroy()` 默认延迟到当前帧末尾才移除对象（常被说成“下一帧”）
> 4. 脚本 `enabled = false` 不会触发 `OnDisable`，仅阻止 `Update` 系列调用

---

## 二、Inspector 检查器序列化特性

### 2.1 基础显示规则
- `private / protected` 变量无法在检查器修改，如需修改，在变量上一行加 `[SerializeField]`
- `public` 变量默认可显示在检查器，如需隐藏，在变量上一行加 `[HideInInspector]`
- 属性（`{ get; set; }`）：默认不序列化，加 `[field: SerializeField]` 可显示自动属性
- TIP：字典、自定义结构体、自定义类不可显示

### 2.2 让自定义的类型可显示
变量上一行加 `[System.Serializable]`

### 2.3 辅助用特性
```csharp
[Header("分组说明")]                       // 分组说明（为变量添加组名）
[Tooltip("说明内容")]                      // 悬停注释（在鼠标停在对应变量上时显示）
[Space(10)]                               // 间隔特性（空行）
[Range(0, 10)]                            // 滑条范围（让值通过滑条调整）
[Multiline(3)]                            // 多行显示（让字符串可分行显示）
[TextArea(2, 10)]                         // 滚动显示字符串（默认前变量行数，超过滚动至后变量行）
[ContextMenuItem("显示按钮名", "方法名(无返回值，无参)")]  // 给变量添加一个右键可用的方法
[ContextMenu("测试函数")]                   // 快捷测试函数（在更多里测试函数）
[RequireComponent(typeof(Rigidbody))]       // 自动挂载依赖组件
[DisallowMultipleComponent]                 // 禁止重复挂载到同一物体
[ExecuteAlways]                             // 编辑器模式也执行生命周期
```

---

## 三、MonoBehaviour 核心成员与方法

### 3.1 重要成员（获取成员）
```csharp
this.gameObject.name        // 获取依附的 GameObject 名称
this.transform.position     // 获取位置信息
this.transform.eulerAngles  // 获取欧拉角信息
this.transform.lossyScale   // 获取世界缩放信息
this.enabled = false;       // 失活脚本
this.enabled = true;        // 激活脚本
```

### 3.2 重要方法（脚本获取）
#### 得到自己挂载的单个脚本（优先泛型写法）
```csharp
// 1. 根据泛型获取（推荐）
父类名 变量名 = this.GetComponent<父类名>();

// 2. 根据 Type 获取
父类名 变量名 = this.GetComponent(typeof(父类名)) as 父类名;

// 3. 根据脚本名获取
父类名 变量名 = this.GetComponent("脚本名") as 父类名;
```

#### 尝试获取脚本（值为 true or false，避免空引用）
```csharp
this.TryGetComponent<父类名>(out 变量名);
```

#### 得到自己挂载的多个脚本
```csharp
父类名[] 变量名 = this.GetComponents<父类名>();

List<父类名> 变量名 = new List<父类名>();
this.GetComponents<父类名>(变量名);
```

#### 得到子对象挂载的脚本
```csharp
// 单个（此处可填 true 或空：填 true 会查找失活与激活，否则只查激活）
this.GetComponentInChildren<子类名>(true);

// 多个
this.GetComponentsInChildren<子类名>(true, List变量名);
```

#### 得到父对象挂载的脚本
```csharp
// 单个
this.GetComponentInParent<父类名>();

// 多个
this.GetComponentsInParent<父类名>(List变量名);
```

---

## 四、GameObject 游戏物体

### 4.1 成员变量
```csharp
this.gameObject.name        // 名字
this.gameObject.activeSelf  // 是否激活
this.gameObject.isStatic    // 是否是静态
this.gameObject.layer       // 层级
this.gameObject.tag         // 标签
this.gameObject.transform   // transform
```

### 4.2 静态方法
#### 创建几何体
```csharp
GameObject.CreatePrimitive(PrimitiveType.几何体);
// 也可
GameObject 变量名 = GameObject.CreatePrimitive(PrimitiveType.几何体);
```

#### 查找对象
```csharp
// 查找单个对象
GameObject.Find("对象名");

// 或者（比 Find 更低效）
GameObject.FindWithTag("Tag名");
// 无法找到失活对象，无法准确确定找到的对象

// 查找多个对象
GameObject[] objs = GameObject.FindGameObjectsWithTag("tag名");
```

#### 实例化与删除对象
```csharp
// 实例化对象
GameObject.Instantiate(对象名);

// 删除对象（第二参数为延迟秒数）
GameObject.Destroy(对象名, 延迟几秒);

// 删除脚本
GameObject.Destroy(this);
// TIPS：一般会在下一帧才移除（常用）

// 立即删除（可在当前帧移除）
GameObject.DestroyImmediate(对象名);

// 切换场景不会移除（保留附着这个脚本的对象）
GameObject.DontDestroyOnLoad(this.gameObject);
```
> 如继承了 MonoBehaviour 可直接省略 `GameObject.` 前缀。

### 4.3 成员方法
```csharp
// 创建空物体（可创建空的、改名的空的、带脚本的空的）
GameObject 对象名 = new GameObject();
GameObject 对象名 = new GameObject(对象命名, typeof(脚本名), typeof(......));

// 为对象添加脚本
对象名.AddComponent<脚本名>();

// 判断对象标签
this.gameObject.CompareTag("标签名");

// 设置激活/失活
对象名.SetActive(false or true);
```

> **以下一般不用：**
> ```csharp
> // 通知命令
> this.gameObject.SendMessage("函数名");
>
> // 广播行为
> this.gameObject.SendMessageUpwards("函数名");
> ```

---

## 五、Time 时间相关

### 5.1 时间缩放比例
```csharp
Time.timeScale = 0;   // 时间暂停
Time.timeScale = 1;   // 恢复正常
Time.timeScale = 2;   // 2 倍速
```

### 5.2 帧间隔 / 时间相关
| 属性 | 说明 | 受 timeScale 影响 |
|------|------|-------------------|
| `Time.deltaTime` | 帧间隔时间（用于控制位移） | ✅ |
| `Time.unscaledDeltaTime` | 帧间隔时间 | ❌ |
| `Time.time` | 游戏开始到现在的时间（主要用于计时） | ✅ |
| `Time.unscaledTime` | 游戏开始到现在的时间 | ❌ |
| `Time.fixedDeltaTime` | 物理帧间隔时间 | ✅ |
| `Time.fixedUnscaledDeltaTime` | 物理帧间隔时间 | ❌ |
| `Time.frameCount` | 帧数 | — |
| `Time.smoothDeltaTime` | 平滑后的帧间隔（用于 UI 帧率显示） | ✅ |

> **核心公式：路程 = 速度 × 时间**
> ```csharp
> this.transform.position += this.transform.forward * speed * Time.deltaTime;
> ```

---

## 六、位置与位移

### 6.1 Vector3
```csharp
// Vector3 本身是一个结构体
Vector3 变量名 = new Vector3(x, y, z);
```

#### 常用常量
| 常量 | 值 | 含义 |
|------|-----|------|
| `Vector3.zero` | (0, 0, 0) | 原点 |
| `Vector3.one` | (1, 1, 1) | 全 1 |
| `Vector3.right` | (1, 0, 0) | 右 |
| `Vector3.left` | (-1, 0, 0) | 左 |
| `Vector3.forward` | (0, 0, 1) | 前 |
| `Vector3.back` | (0, 0, -1) | 后 |
| `Vector3.up` | (0, 1, 0) | 上 |
| `Vector3.down` | (0, -1, 0) | 下 |

#### 方法
```csharp
// 计算两个点的距离
Vector3.Distance(点1, 点2);

// 归一化方向向量
Vector3 dir = (target - origin).normalized;

// 线性插值（t 为 0~1 的混合比例）
Vector3 lerp = Vector3.Lerp(a, b, t);

// 匀速逼近（每步最多移动 maxDelta）
Vector3 move = Vector3.MoveTowards(cur, target, maxDelta);

// 点积（判断前后 / 夹角）
float dot = Vector3.Dot(a, b);

// 叉积（求法线 / 垂直方向）
Vector3 cross = Vector3.Cross(a, b);
```

### 6.2 位置
```csharp
this.transform.position;      // 相对世界坐标系（面板上显示为相对父对象位置）
this.transform.localPosition; // 相对父对象（得到面板位置）
```
> 注意：`.position` 无法直接改变单独的 x / y / z。

#### 如何改变 position
```csharp
// 方式一：整体重新赋值
this.transform.position = new Vector3(19, this.transform.position.y, this.transform.position.z);

// 方式二：单独用一个 Vector3 变量存储，修改后再赋值
```

#### 对象当前的各朝向
```csharp
this.transform.forward;  // 对象的面朝向
this.transform.up;       // 对象的头顶朝向
this.transform.right;    // 对象的右手边
```

### 6.3 位移（路程 = 方向 × 速度 × 时间）
```csharp
// 方法一：自己算
this.transform.position += this.transform.forward * speed * Time.deltaTime;

// 方法二：API
// 参数一：位移多少（路程 = 方向 × 速度 × 时间）
// 参数二：相对坐标系（默认相对自身坐标）
this.transform.Translate(Vector3.forward * speed * Time.deltaTime, Space.World);
```

---

## 七、角度与旋转

### 7.1 欧拉角与四元数
```csharp
// 欧拉角（直观，但可能有万向锁）
this.transform.eulerAngles;       // 相对世界坐标角度
this.transform.localEulerAngles;  // 相对父坐标角度

// 四元数（推荐，无万向锁）
this.transform.rotation;          // 世界坐标系旋转（Quaternion）
this.transform.localRotation;     // 相对父物体的旋转（Quaternion）
Quaternion.identity;              // 无旋转
```
> 修改角度同修改位置（整体重新赋值）。

### 7.2 旋转方法
```csharp
// 自转：往 x 转（抬头低头）、往 y 转（左右摇头）、往 z 转（偏头）
this.transform.Rotate(new Vector3(0, 10, 0) * Time.deltaTime, 相对坐标系);

// 相对某个轴旋转
this.transform.Rotate(Vector3.right, 旋转角度, 相对坐标轴);

// 相对某个点转（参数一：绕的点；参数二：绕的轴；参数三：度数）
this.transform.RotateAround(Vector3.zero, Vector3.right, 10 * Time.deltaTime);

// 常用四元数旋转
this.transform.rotation = Quaternion.Euler(0, 45, 0);               // 欧拉角 → 四元数
this.transform.rotation = Quaternion.LookRotation(forward, up);      // 朝向方向
Quaternion target = Quaternion.FromToRotation(from, to);            // 从方向 A 转到方向 B
Quaternion lerp = Quaternion.Lerp(a, b, t);                        // 线性插值
Quaternion slerp = Quaternion.Slerp(a, b, t);                      // 球面插值（旋转更平滑）
```

---

## 八、缩放与看向

### 8.1 缩放
```csharp
this.transform.lossyScale;  // 相对世界坐标系（只能得到不能修改）
this.transform.localScale;  // 相对本地坐标系（可读写）
```
> TIPS：缩放在 Unity 中无 API，想要逐渐调整缩放需要自行计算：
> ```csharp
> this.transform.localScale += new Vector3(1, 1, 1) * Time.deltaTime;
> ```

### 8.2 看向（让物体一直面向坐标 / 物体）
```csharp
this.transform.LookAt(坐标);

public Transform obj;
this.transform.LookAt(obj);
```

---

## 九、父子关系

### 9.1 获取 / 设置父对象
```csharp
// 获取
this.transform.parent;

// 移除
this.transform.parent = null;

// 修改
this.transform.parent = GameObject.Find("name").transform;

// 通过 API 修改
// 参数一：父对象；参数二：是否保留当前世界坐标、缩放、旋转
this.transform.SetParent(object.transform, true);
```

### 9.2 解除所有子对象（抛妻弃子）
```csharp
this.transform.DetachChildren();
```

### 9.3 获取子对象
```csharp
// 按名字查找儿子（能够找到失活对象）
this.transform.Find("name");

// 按索引遍历儿子
this.transform.GetChild(索引);
```

### 9.4 子的操作
```csharp
son.IsChildOf(this.transform);   // 判断自己是否为另一个对象的子对象
son.GetSiblingIndex();           // 得到自己作为子对象的编号
son.SetAsFirstSibling();         // 把自己设置为第一个子对象
son.SetAsLastSibling();          // 把自己设置为最后一个子对象
son.SetSiblingIndex(数字);        // 把自己设置为指定个儿子
```

---

## 十、坐标转换

### 10.1 世界坐标转本地坐标
```csharp
// 点（将该点转换为相对挂载对象坐标系上的本地坐标位置）
transform.InverseTransformPoint(坐标点);

// 方向
transform.InverseTransformDirection(坐标点);
```

### 10.2 本地坐标转世界坐标
```csharp
// 点
this.transform.TransformPoint(向量坐标);

// 方向
this.transform.TransformDirection(向量坐标);
```

---

## 十一、鼠标键盘输入

> **注意：** 以下为旧版 Input Manager API，适合快速原型。新项目推荐使用 **Input System** 包（`UnityEngine.InputSystem`），支持事件驱动、多设备映射、Input Action 资产管理。

### 11.1 鼠标在屏幕位置
```csharp
Input.mousePosition;
// 屏幕的原点为左下角，右侧是 x 正方向，上侧为 y 正方向
// 返回时为 Vector3，但是 z = 0
```

### 11.2 检测鼠标输入
```csharp
// 鼠标按下：0 左键，1 右键，2 中键
Input.GetMouseButtonDown(0 / 1 / 2);

// 鼠标抬起
Input.GetMouseButtonUp(0 / 1 / 2);

// 鼠标长按（按下抬起都会触发）
Input.GetMouseButton(0 / 1 / 2);

// 中键滚动：-1 下，0 无，1 上
Input.mouseScrollDelta;
```

### 11.3 检测键盘输入
```csharp
// 键盘按下
Input.GetKeyDown(KeyCode.W);   // W 键按下
Input.GetKeyDown("w");         // 写法 2：填大写会报错

// 键盘抬起
Input.GetKeyUp(KeyCode.W);

// 键盘长按
Input.GetKey(KeyCode.W);
```

### 11.4 检测默认轴输入
```csharp
// 键盘 AD 按下时返回 -1 到 1 的变换（平滑过渡）
Input.GetAxis("轴名");
// 轴名内容为 Edit → Project Settings → Input Manager → Axes 下的所有热键名

// 无过渡，仅返回 -1 / 0 / 1 三档
Input.GetAxisRaw("轴名");
```

### 11.5 其他输入
```csharp
Input.anyKey;               // 是否有任意键或鼠标长按
Input.anyKeyDown;           // 是否有任意键或鼠标按下
Input.inputString;          // 这一帧的键盘输入
string[] strs = Input.GetJoystickNames();  // 得到连接手柄的所有按钮名字
```

### 11.6 移动设备触摸相关
```csharp
if (Input.touchCount > 0)   // 触摸数大于 0
{
    Touch t1 = Input.touches[0];
}

Input.multiTouchEnabled = false;   // 是否启用多点触控

// 陀螺仪
Input.gyro.enabled = true;         // 是否开启陀螺仪
Input.gyro.gravity;                // 重力加速度向量
```

---

## 十二、Screen 屏幕相关

### 12.1 静态属性
```csharp
// 当前分辨率
Resolution r = Screen.currentResolution;
print(r.width + r.height);   // 得到当前屏幕分辨率的宽高

// 屏幕窗口当前宽高
Screen.width;    // 游戏窗口的宽高
Screen.height;

// 屏幕休眠模式
Screen.sleepTimeout = SleepTimeout.NeverSleep;   // 永不熄屏

// 运行时是否全屏
Screen.fullScreenMode = FullScreenMode.FullScreenWindow;

// 移动设备屏幕转向相关（不常用，需要再搜）......
```

### 12.2 静态方法
```csharp
// 设置分辨率（宽，高，是否全屏）
Screen.SetResolution(1920, 1080, false);
```

---

## 十三、Camera 摄像头参数说明

### 13.1 核心参数
| 参数 | 说明 |
|------|------|
| Clear Flags | 背景清除方式：Skybox 天空盒（常 3D）/ Solid Color 颜色填充（3D 与 2D）/ Depth Only 只画该层背景透明 / Don't Clear 不移除覆盖渲染 |
| Culling Mask | 选择性渲染部分层级 |
| Projection | Perspective 透视模式（FOV Axis 视场角方向 / Field of View 视口大小 / Physical Camera 模拟真实相机参数）；Orthographic 正交摄像机（常用于 2D，Size 摄制范围） |
| Clipping Planes | 裁剪平面距离（可渲染的范围） |
| Depth | 渲染顺序上的深度，值越大越后被渲染，遮盖掉之前渲染的画面 |
| Target Texture | 渲染纹理，将摄像机画面渲染到一张图上 |
| Occlusion Culling | 剔除遮挡，被挡住的物体不会被渲染 |
| Viewport Rect | 视口范围，修改 x/y/w/h 改变显示在窗口的位置与宽高 |
| Allow HDR | 开启后相机支持高动态光照渲染，呈现更大明暗反差、光晕、泛光等特效；关闭则性能开销更低 |
| Allow MSAA | 开启多重采样抗锯齿，消除物体边缘锯齿；性能紧张场景可关闭节省性能 |
| Allow Dynamic Resolution | 允许引擎动态调整渲染分辨率，负载过高时自动降低分辨率维持帧率 |
| Target Display | 指定相机画面输出到几号显示器，多用于多屏输出、街机、多屏幕主机游戏开发 |

---

## 十四、Camera 代码相关

### 14.1 重要静态成员
```csharp
// 1. 获取摄像机（需要摄像机的 Tag 为 Main Camera）
Camera.main;

// 2. 获取摄像机的数量
Camera.allCamerasCount;

// 3. 得到所有摄像机
Camera[] allCamera = Camera.allCameras;

// 4. 渲染相关委托
Camera.onPreCull += (c) => { };    // 摄像机剔除前处理
Camera.onPreRender += (c) => { };  // 摄像机渲染前
Camera.onPostRender += (c) => { }; // 摄像机渲染后
```

### 14.2 重要成员
```csharp
// 1. 界面上的参数都可以在 Camera 中获取到
Camera.main.depth = 10;

// 2. 世界坐标转屏幕坐标（可用于制作血条，3D 世界人物头顶那种）
Vector3 v = Camera.main.WorldToScreenPoint(this.transform.position);

// 3. 屏幕坐标转世界坐标（在横截面 v 距离的位置移动）
Vector3 v = Input.mousePosition;
v.z = 5;
Camera.main.ScreenToWorldPoint(v);
```

---

## 十五、光源组件

### 15.1 Type（光源类型）
| 类型 | 说明 |
|------|------|
| Spot | 聚光灯 |
| Directional | 方向灯（环境光） |
| Point | 点光源 |
| Area | 面光源 |

### 15.2 核心参数
- **Color**：颜色
- **Mode**：RealTime 实时光源（效果好，性能消耗大）/ Baked 烘焙光源（预先计算好，无法动态变化）/ Mixed 混合光源（以上相加）
- **Intensity**：光源亮度
- **Shadow Type**：No Shadows 关闭阴影 / Hard Shadows 生硬阴影 / Soft Shadows 柔和阴影
- **Cookie**：投影遮罩
- **Draw Halo**：球形光环开关（蜡烛的光晕）
- **Flare**：耀斑（添加 Flare 组件，实现光中的小光圈效果）
- **Culling Mask**：剔除遮罩层，勾选的层级才会被光影响
- **其他**：……

### 15.3 光源的代码控制
获取 Light 组件后通过 `light.参数` 访问以上面板参数。

---

## 十六、光相关面板

面板路径：`Window → Rendering → Lighting Settings`

### 16.1 Environment
- **Skybox Material**：可以改变天空盒材质
- **Sun Source**：太阳来源
- **Environment Lighting**：环境光设置：……

### 16.2 其他
……

---

## 十七、碰撞检测

> 碰撞产生的必要条件：两个物体都有碰撞体，至少一个物体有刚体。

### 17.1 刚体（Rigidbody：让物体受到力的效果）
| 参数 | 说明 |
|------|------|
| Mass | 质量（默认为千克），质量越大惯性越大 |
| Drag | 空气阻力，根据力移动对象时影响对象的空气阻力大小，0 表示没有空气阻力 |
| Angular Drag | 根据扭矩旋转对象时影响对象的空气阻力大小，0 表示没有空气阻力 |
| Use Gravity | 是否受重力影响 |
| Is Kinematic | 启用后对象不被物理引擎驱动，只能通过 Transform 操作，适用于移动平台或动画化刚体 |
| Interpolate | 插值运算，让刚体物体移动更平滑 |
| Collision Detection | 碰撞检测模式，用于防止快速移动的对象穿过其它对象而不检测碰撞 |
| Constraints | 约束，对刚体运动的限制 |

### 17.2 碰撞器
#### 3D 碰撞器种类
盒状、球状、胶囊、网格、轮胎、地形

#### 共同参数
- **Is Trigger**：是否是触发器，启用后该碰撞体用于检测碰撞，无物理效果
- **Material**：物理材质，用于确定碰撞体和其他对象碰撞的交互表现
- **Center**：碰撞体在对象局部空间的中心点位置

#### 常用碰撞体参数
- 盒状：Size 在 xyz 上的大小
- 球状：Radius 球形的半径大小
- 胶囊：Radius 半径、Height 高度、Direction 在对象局部空间中的轴向

#### 异形物体
使用多种碰撞器组合，子对象会继承父对象的刚体，由此组合一个复杂物体的碰撞体。

#### 不常用碰撞器
- 网格：根据物体的面设置碰撞判断，Convex 勾选才能使用刚体（最多 255 面计算）
- 地形：性能开销大
- 轮胎：环形，不常用

### 17.3 材质（物理材质）
| 参数 | 说明 |
|------|------|
| Dynamic Friction | 已在移动时使用的摩擦力 |
| Static Friction | 静止时的摩擦力（越大越难移动） |
| Bounciness | 表面弹性（越小弹力越小） |
| Friction Combine | 两个对象摩擦力的组合方式：Average 取平均 / Minimum 取最小 / Maximum 取最大 / Multiply 相乘 |
| Bounce Combine | 弹性组合方式，类似于碰撞摩擦力组合方式 |

### 17.4 刚体加力
```csharp
// 1. 获取刚体组件
Rigidbody body = this.GetComponent<Rigidbody>();

// 2. 添加力（相对世界坐标系 / 相对本地坐标）
body.AddForce(Vector3.forward * 10);
body.AddRelativeForce(Vector3.forward * 10);

// 3. 添加扭矩力，使其旋转（相对世界 / 相对本地）
body.AddTorque(Vector3.up * 10);
body.AddRelativeTorque(Vector3.up * 10);

// 4. 直接改变速度
body.velocity = Vector3.forward * 5;

// 5. 模拟爆炸效果（参数：力、中心点、半径）
body.AddExplosionForce(10, Vector3.zero, 10);
// 只有挂载并写了此句的物体才能受到爆炸影响
```

#### 力的几种模式
```csharp
body.AddForce(Vector3.forward * 10, ForceMode.模式);
```

| ForceMode | 说明 | 计算公式 |
|-----------|------|----------|
| `Force` | 持续力（考虑质量） | `v += F/m × Δt` |
| `Acceleration` | 持续加速度（忽略质量） | `v += F × Δt` |
| `Impulse` | 瞬间冲量（考虑质量） | `v += F/m` |
| `VelocityChange` | 瞬间速度变化（忽略质量） | `v += F` |

> **区分：** `Force` / `Acceleration` 需要在 `FixedUpdate` 中每帧施加；`Impulse` / `VelocityChange` 为瞬时作用，一次调用即生效。

#### Constant Force 立场脚本
挂载到物体上即可持续施加恒定的力 / 扭矩，无需代码控制。

> **补充：刚体休眠** —— 静止不动的刚体会自动进入休眠状态以节省性能，受到力 / 碰撞时会自动唤醒。
> ```csharp
> // 手动休眠 / 唤醒
> body.Sleep();             // 立即进入休眠
> body.WakeUp();            // 立即唤醒
> body.isSleeping;          // 返回是否处于休眠状态
>
> // 注意：物体激活状态改变时会自动唤醒刚体
> // 休眠阈值可在 Edit → Project Settings → Physics 中调整：
> //   Sleep Threshold：线速度低于此值进入休眠
> //   Sleep Angular Threshold：角速度低于此值进入休眠
> ```

---

## 十八、音效系统

### 18.1 音频文件 AudioClip 导入参数
| 参数 | 说明 |
|------|------|
| Force To Mono | 多声道转单声道 |
| Normalize | 强制为单声道时，混合过程中被标准化 |
| Load In Background | 在后台加载，不阻塞主线程 |
| Ambisonic | 立体混响声，非常适合 360 度视频和 XR 应用程序（音频文件包含立体混响声编码时启用） |
| Preload Audio Data | 预加载音频，勾选后进入场景就加载，不勾选第一次使用时才加载 |

#### LoadType 加载类型
| 类型 | 说明 |
|------|------|
| Decompress On Load | 不压缩形式存在内存，加载快，内存占用高，适用于小音效 |
| Compress In Memory | 压缩形式存在内存，加载慢，内存小，仅适用于较大音效文件 |
| Streaming | 以流形式存在，使用时解码，内存占用最小，CPU 消耗高（性能换内存） |

#### Compression Format 压缩方式
| 格式 | 说明 |
|------|------|
| PCM | 音频以最高质量存储 |
| Vorbis | 相对 PCM 压缩得更小，根据质量决定 |
| ADPCM | 包含噪音，适用于会被多次播放的声音，如碰撞声 |

### 18.2 音频源 AudioSource 基础参数
| 参数 | 说明 |
|------|------|
| AudioClip | 声音剪辑文件（音频文件） |
| Output | 默认直接输出到场景中的音频监听器，可以更改为输出到混音器 |
| Mute | 静音开关 |
| Bypass Effect | 开关滤波器效果 |
| Bypass Listener Effects | 快速开关所有监听器效果 |
| Bypass Reverb Zones | 快速开关所有混响区 |
| Play On Awake | 对象创建时就播放音乐（游戏启动物体激活后自动播放） |
| Loop | 循环 |
| Priority | 优先级 |
| Volume | 音量大小 |
| Pitch | 音高 |

### 18.3 AudioSource 3D 音效设置（3D Sound Settings）
| 参数 | 说明 |
|------|------|
| Stereo Pan | 2D 声音立体声位置（相当于左右声道） |
| Spatial Blend | 音频受 3D 空间的影响程度 |
| Reverb Zone Mix | 到混响区的输出信号量 |
| 3D Sound Settings | 和 Spatial Blend 参数成正比应用 |
| Doppler Level | 多普勒效果等级 |
| Spread | 扩散角度设置为 3D 立体声还是多声道 |
| Min/Max Distance | 最小距离内声音保持最大响度，最大距离外声音开始减弱 |

#### Volume Rolloff 声音衰减速度
| 模式 | 说明 |
|------|------|
| Logarithmic Rolloff | 靠近音频源时声音很大，但离开对象时声音降低得非常快 |
| Linear Rolloff | 与音频源的距离越远，听到的声音越小 |
| Custom Rolloff | 音频源的音频效果根据曲线图的设置变化 |

### 18.4 监听脚本 Audio Listener
常挂载于主摄像头。

### 18.5 代码控制音频
```csharp
AudioSource audio;        // 定义音频源

audio.Play();            // 播放
audio.Stop();            // 停止
audio.Pause();           // 暂停
audio.UnPause();         // 停止暂停（播放）
audio.PlayDelayed(秒数);  // 延迟播放

audio.isPlaying;         // 是否正在播放（true 是 / false 否）
```

#### 如何动态控制音效播放
1. 直接在需播放的对象上挂载脚本
2. 实例化挂载了音效原脚本的对象
3. 用一个 AudioSource 来控制不同的音效：
```csharp
public AudioClip clip;                                            // 定义切片
AudioSource aus = this.gameObject.AddComponent<AudioSource>();    // 新增音频源
aus.clip = clip;                                                  // 获取当前对象上的切片
aus.Play();                                                       // 播放切片
```

### 18.6 麦克风输入相关
```csharp
// 获取设备麦克风信息（字符型）
Microphone.devices;

// 开始录制（设备名, 超过录制长度后是否重头录制, 录制时长, 采样率）
Microphone.Start(null, false, 10, 44100);

// 结束录制（设备名）
Microphone.End(null);

// 获取音频数据用于存储或者传输（声道数 × 剪辑长度）
float[] f = new float[clip.channels * clip.samples];
```

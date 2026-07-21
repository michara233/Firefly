---
title: unity学习速查
published: 2026-07-20
description: 更新中ing...
tags: [unity, 游戏开发,基础知识]
category: 基础知识
slug: unity学习速查
pinned: true
---

# Unity C# 核心脚本速查笔记

## 一、MonoBehaviour 生命周期函数

| 函数 | 执行时机 | 核心用途 | 调用次数 |
|------|----------|----------|----------|
| `Awake()` | 对象实例化/激活瞬间 | 初始化组件、获取引用（类似构造函数） | 生命周期仅1次 |
| `OnEnable()` | 对象每次从失活→激活 | 注册事件、开启显示、刷新状态 | 每次激活都执行 |
| `Start()` | 首次 Update 前一帧 | 依赖其他对象的初始化（Awake 全部执行完才会走） | 生命周期仅1次 |
| `FixedUpdate()` | 固定物理帧率（不受帧率波动影响） | 物理运动、刚体、碰撞逻辑 | 默认每 0.02s 执行 |
| `Update()` | 渲染帧更新（随设备帧率变化） | 输入检测、常规移动、游戏逻辑 | 每渲染帧执行 |
| `LateUpdate()` | 所有 Update 执行完毕后 | 相机跟随、后处理、跟随物体位置修正 | 每渲染帧执行 |
| `OnDisable()` | 对象从激活→失活 | 注销事件、关闭特效、暂停逻辑 | 每次失活都执行 |
| `OnDestroy()` | 对象被销毁时 | 释放资源、移除事件、清理缓存 | 销毁前执行1次 |

> **补充规则：**
> 1. 场景加载时，所有物体先执行全部 `Awake`，再统一执行 `Start`
> 2. 物体 `SetActive(false)` 只会触发 `OnDisable`，不会调用 `OnDestroy`
> 3. `Destroy()` 销毁物体**下一帧**才会执行 `OnDestroy`

---

## 二、Inspector 面板序列化特性

### 1. 基础显示规则
- `public` 变量：默认显示在 Inspector
- `private/protected` 变量：默认隐藏，需加 `[SerializeField]` 序列化显示
- 自定义类/结构体、Dictionary 字典：默认无法序列化显示，需加 `[System.Serializable]`
- 不想公开变量被面板修改：加 `[HideInInspector]`

### 2. 常用面板装饰特性
```csharp
[Header("玩家基础属性")]     // 分组标题
[Tooltip("移动速度，推荐2~8")] // 鼠标悬停提示
[Space(10)]                  // 下方空10像素间距
[Range(0, 10)]               // 滑动条限定数值范围
public float moveSpeed;

[Multiline(3)]               // 多行单行文本框
public string shortText;

[TextArea(2, 10)]            // 滚动文本域（最小2行，超10行出现滚动条）
public string longDesc;

[ContextMenuItem("重置数值", "ResetSpeed")] // 变量右键菜单按钮
public float jumpPower;

// 脚本右上角更多菜单
[ContextMenu("打印速度")]
void PrintSpeed()
{
    Debug.Log(moveSpeed);
}
```

### 3. 自定义类型显示要求
自定义类/结构体头部必须加 `[System.Serializable]`，否则面板无法渲染：
```csharp
[System.Serializable]
public class PlayerData
{
    public int hp;
    public string name;
}
```

---

## 三、MonoBehaviour 核心成员与 API

### 1. 自带快捷成员
```csharp
gameObject   // 当前挂载脚本的物体
transform    // 物体变换组件（transform = gameObject.transform）
enabled      // 脚本启用状态：this.enabled = false; 仅关闭脚本，物体不会失活
```

### 2. 获取自身组件（优先泛型写法）
```csharp
// 1. 泛型（推荐，编译安全，无类型转换）
Player player = GetComponent<Player>();

// 2. Type 类型
Player player = GetComponent(typeof(Player)) as Player;

// 3. 字符串名字（不推荐，写错无编译报错）
Player player = GetComponent("Player") as Player;

// 4. 安全尝试获取（避免空引用报错）
if (TryGetComponent(out Player player))
{
    // 存在组件才执行逻辑
}

// 获取自身多个同类型组件
Player[] allPlayers = GetComponents<Player>();
List<Player> playerList = new List<Player>();
GetComponents<Player>(playerList);
```

### 3. 获取子物体组件
```csharp
// 参数 true：包含失活子物体；不传/填 false：仅查找激活子物体
Player childPlayer = GetComponentInChildren<Player>(true);
Player[] allChildPlayer = GetComponentsInChildren<Player>(true);
```

### 4. 获取父物体组件
```csharp
Player parentPlayer = GetComponentInParent<Player>();
Player[] allParentPlayer = GetComponentsInParent<Player>();
```

---

## 四、GameObject 物体操作

### 1. 物体基础属性
```csharp
gameObject.name = "Player";    // 修改物体名称
gameObject.tag = "Enemy";      // 修改标签
gameObject.layer = 8;          // 修改层级
gameObject.activeSelf;         // 是否激活（只读）
gameObject.isStatic;           // 是否静态
gameObject.transform;          // 变换组件（等价于 transform）
```

### 2. 创建与查找物体
```csharp
// 创建内置几何体
GameObject cube = GameObject.CreatePrimitive(PrimitiveType.Cube);

// 创建空物体
GameObject emptyObj = new GameObject();
GameObject objWithScript = new GameObject("MyObj", typeof(Rigidbody), typeof(BoxCollider));

// 查找单个物体
GameObject player = GameObject.Find("Player");       // 按名称（效率低，无法找失活）
GameObject enemy = GameObject.FindWithTag("Enemy");  // 按标签

// 查找多个物体
GameObject[] enemies = GameObject.FindGameObjectsWithTag("Enemy");
```

### 3. 实例化与销毁
```csharp
// 克隆物体（预制体实例化）
GameObject clone = Instantiate(prefabObj);

// 销毁物体（第二参数为延迟秒数，默认0）
Destroy(gameObject, 2f);
Destroy(this); // 销毁脚本组件本身

// 立即销毁（当前帧移除，少用）
DestroyImmediate(gameObject);

// 切换场景不销毁（DontDestroyOnLoad）
DontDestroyOnLoad(gameObject);
```

> **注意：** 继承 MonoBehaviour 后，`Instantiate`、`Destroy`、`DontDestroyOnLoad` 可省略 `GameObject.` 直接调用

### 4. 常用成员方法
```csharp
// 添加组件
Rigidbody rb = gameObject.AddComponent<Rigidbody>();

// 标签对比
if (gameObject.CompareTag("Player")) { }

// 设置激活/失活
gameObject.SetActive(true);
gameObject.SetActive(false);
```

### 5. 不常用方法
```csharp
// 向自身及子物体发送消息调用方法
gameObject.SendMessage("方法名");
gameObject.SendMessageUpwards("方法名"); // 同时向父物体传播
```

---

## 五、Time 时间系统

| 属性 | 说明 | 受 timeScale 影响 |
|------|------|-------------------|
| `Time.deltaTime` | 上一帧到当前帧的间隔时间（位移必备） | ✅ |
| `Time.unscaledDeltaTime` | 不受时间缩放影响的帧间隔 | ❌ |
| `Time.time` | 游戏开始到现在的总时间（计时用） | ✅ |
| `Time.unscaledTime` | 不受时间缩放影响的总时间 | ❌ |
| `Time.fixedDeltaTime` | 物理帧间隔时间 | ✅ |
| `Time.fixedUnscaledDeltaTime` | 不受时间缩放影响的物理帧间隔 | ❌ |
| `Time.frameCount` | 游戏运行总帧数 | - |
| `Time.timeScale` | 时间缩放比例（0=暂停，1=正常，2=二倍速） | - |

> **核心公式：路程 = 速度 × 时间** → 移动必须乘 `Time.deltaTime` 才能保证不同帧率下速度一致

---

## 六、位置与位移

### 1. Vector3 三维向量
```csharp
// 构造
Vector3 pos = new Vector3(x, y, z);

// 常用常量
Vector3.zero     // (0, 0, 0) 原点
Vector3.one      // (1, 1, 1)
Vector3.right    // (1, 0, 0)
Vector3.left     // (-1, 0, 0)
Vector3.up       // (0, 1, 0)
Vector3.down     // (0, -1, 0)
Vector3.forward  // (0, 0, 1)
Vector3.back     // (0, 0, -1)

// 计算两点距离
float dist = Vector3.Distance(pointA, pointB);
```

### 2. 位置坐标
```csharp
transform.position       // 世界坐标
transform.localPosition  // 本地相对父物体坐标（面板显示的值）

// 物体自身朝向向量
transform.forward        // 物体正前方（Z轴）
transform.up             // 物体正上方（Y轴）
transform.right          // 物体正右方（X轴）
```

> **注意：** 不能直接修改 position 的单个分量（如 `position.x = 5`），必须整体赋值：
> ```csharp
> transform.position = new Vector3(5, transform.position.y, transform.position.z);
> ```

### 3. 位移移动
```csharp
// 方法一：直接计算
transform.position += transform.forward * speed * Time.deltaTime;

// 方法二：Translate API（第二参数可选 Space.World / Space.Self）
transform.Translate(Vector3.forward * speed * Time.deltaTime, Space.Self);
```

---

## 七、角度与旋转

### 1. 欧拉角
```csharp
transform.eulerAngles       // 世界欧拉角
transform.localEulerAngles  // 本地欧拉角（面板显示的值）
```

> 修改方式同 position，需整体赋值，不能单独改某一分量

### 2. 旋转方法
```csharp
// 自转（绕自身轴旋转）
// X轴：抬头低头  Y轴：左右摇头  Z轴：偏头
transform.Rotate(new Vector3(0, 90, 0) * Time.deltaTime);

// 绕指定轴旋转
transform.Rotate(Vector3.up, 30f * Time.deltaTime);

// 绕某点某轴旋转（公转）
transform.RotateAround(Vector3.zero, Vector3.up, 20f * Time.deltaTime);
```

---

## 八、缩放与看向

### 1. 缩放
```csharp
transform.localScale    // 本地缩放（可读写）
transform.lossyScale    // 世界总缩放（只读，不可赋值）

// 缩放无专用 API，自行累加
transform.localScale += Vector3.one * Time.deltaTime;
```

### 2. 看向（LookAt）
```csharp
// 让物体 Z 轴始终朝向目标点/物体
transform.LookAt(targetPosition);
transform.LookAt(targetTransform);
```

---

## 九、父子关系操作

### 1. 设置父物体
```csharp
// 获取父物体
Transform parent = transform.parent;

// 设置/移除父物体
transform.parent = null;                    // 解除父子关系
transform.SetParent(newParent, true);       // 第二参数：是否保留世界坐标/缩放/旋转
```

### 2. 子物体操作
```csharp
// 解除所有子物体
transform.DetachChildren();

// 按名字查找子物体（可找到失活物体）
Transform child = transform.Find("ChildName");

// 按索引获取子物体
Transform firstChild = transform.GetChild(0);

// 子物体层级操作
child.IsChildOf(parent);        // 判断是否为子物体
child.GetSiblingIndex();        // 获取自身索引
child.SetAsFirstSibling();      // 设为第一个
child.SetAsLastSibling();       // 设为最后一个
child.SetSiblingIndex(2);       // 设为指定索引
```

---

## 十、坐标转换

### 世界坐标 ↔ 本地坐标
```csharp
// 世界坐标 → 本地坐标
Vector3 localPos = transform.InverseTransformPoint(worldPos);
Vector3 localDir = transform.InverseTransformDirection(worldDir);

// 本地坐标 → 世界坐标
Vector3 worldPos = transform.TransformPoint(localPos);
Vector3 worldDir = transform.TransformDirection(localDir);
```

---

## 十一、输入系统（Input）

### 1. 鼠标输入
```csharp
// 鼠标位置（屏幕像素坐标，左下角为原点，X向右Y向上，返回Vector3但z=0）
Vector3 mousePos = Input.mousePosition;

// 鼠标按键：0=左键，1=右键，2=中键
Input.GetMouseButton(0);      // 按住持续触发
Input.GetMouseButtonDown(0);  // 按下瞬间触发
Input.GetMouseButtonUp(0);    // 抬起瞬间触发

// 鼠标滚轮（-1向下，0无，1向上）
float scroll = Input.mouseScrollDelta.y;
```

### 2. 键盘输入
```csharp
Input.GetKey(KeyCode.W);          // 按住持续触发
Input.GetKeyDown(KeyCode.Space);  // 按下瞬间触发
Input.GetKeyUp(KeyCode.Space);    // 抬起瞬间触发

// 字符串写法（不推荐，写错无报错，注意小写）
Input.GetKeyDown("w");
```

### 3. 轴输入（Horizontal / Vertical）
```csharp
float h = Input.GetAxis("Horizontal");   // A/D 或 ←/→ 控制，-1 ~ 1，平滑过渡
float v = Input.GetAxis("Vertical");     // W/S 或 ↑/↓ 控制

float hRaw = Input.GetAxisRaw("Horizontal"); // 只有 -1 / 0 / 1 三档，无过渡
```

> 轴名称可在 Edit → Project Settings → Input Manager 中查看和自定义

### 4. 其他输入
```csharp
// 任意键/鼠标
Input.anyKey;       // 是否有任意键或鼠标长按
Input.anyKeyDown;   // 是否有任意键或鼠标按下

// 这一帧的键盘输入字符串
string input = Input.inputString;

// 获取已连接的手柄名称
string[] joysticks = Input.GetJoystickNames();
```

### 5. 移动设备触摸
```csharp
if (Input.touchCount > 0)
{
    Touch touch = Input.touches[0];
    // touch.position 触摸位置
    // touch.phase 触摸阶段（Began/Moved/Ended等）
}

// 是否启用多点触控
Input.multiTouchEnabled = false;
```

### 6. 陀螺仪
```csharp
Input.gyro.enabled = true;    // 开启陀螺仪
Vector3 gravity = Input.gyro.gravity; // 重力加速度向量
```

---

## 十二、Screen 屏幕系统

### 1. 静态属性
```csharp
// 当前显示器分辨率
Resolution res = Screen.currentResolution;
int width = res.width;
int height = res.height;

// 游戏窗口宽高
int windowWidth = Screen.width;
int windowHeight = Screen.height;

// 屏幕休眠模式（永不熄屏）
Screen.sleepTimeout = SleepTimeout.NeverSleep;

// 全屏模式
Screen.fullScreenMode = FullScreenMode.FullScreenWindow;
```

### 2. 静态方法
```csharp
// 设置分辨率（宽，高，是否全屏）
Screen.SetResolution(1920, 1080, false);
```

---

## 十三、Camera 摄像机参数

### 1. Clear Flags（清除标记）
| 模式 | 说明 | 适用场景 |
|------|------|----------|
| Skybox | 天空盒填充背景 | 3D 游戏常用 |
| Solid Color | 纯色填充背景 | 2D / UI 场景 |
| Depth Only | 只绘制深度，背景透明 | 分层渲染叠加 |
| Don't Clear | 不清除，上一帧画面残留 | 特殊效果 |

### 2. Culling Mask（剔除遮罩）
选择性渲染指定 Layer 的物体，可用于分层渲染

### 3. Projection（投影模式）
#### Perspective（透视模式 - 3D 常用）
- **Field of View**：视场角大小
- **FOV Axis**：视场角计算轴向（竖直/水平）
- **Physical Camera**：模拟真实相机参数

#### Orthographic（正交模式 - 2D 常用）
- **Size**：正交摄像机摄制范围

#### 通用
- **Clipping Planes**：裁剪平面（Near / Far），可渲染的距离范围

### 4. Depth（深度）
- 值越大越后渲染，会遮盖之前渲染的画面
- **用法**：多摄像机分层渲染（如 UI 摄像机 + 游戏摄像机），配合 Clear Flags 和 Culling Mask 使用

### 5. Target Texture（渲染纹理）
将摄像机画面渲染到一张 Render Texture 上，可用于小地图、监控画面等

### 6. Occlusion Culling（遮挡剔除）
被其他物体完全挡住的物体不会被渲染，提升性能

### 7. Viewport Rect（视口矩形）
调整摄像机画面在屏幕上的位置和大小（XY 为位置，WH 为宽高，取值 0~1），可用于分屏、画中画等效果

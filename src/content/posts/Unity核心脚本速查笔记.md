---
title: Unity 核心脚本速查笔记
published: 2026-07-20
updated: 2026-07-28
description: Unity 常用 API 速查 — 生命周期、Transform、Input、Camera、物理等，持续更新中
tags: [unity, 游戏开发, 基础知识, 速查]
category: 基础知识
slug: unity-core-scripting-cheatsheet
pinned: true
---

# Unity 核心脚本速查笔记

> 基于 Unity 学习笔记整理，重新分类结构、统一术语、补充代码示例，方便复习查询。

## 目录

1. [生命周期](#1-生命周期)
2. [Inspector 检查器与特性](#2-inspector-检查器与特性)
3. [MonoBehaviour 常用 API](#3-monobehaviour-常用-api)
4. [GameObject 常用 API](#4-gameobject-常用-api)
5. [Time 时间系统](#5-time-时间系统)
6. [Vector3 与 Transform](#6-vector3-与-transform)
7. [父子关系](#7-父子关系)
8. [坐标转换](#8-坐标转换)
9. [输入系统 Input](#9-输入系统-input)
10. [Screen 屏幕相关](#10-screen-屏幕相关)
11. [Camera 摄像机](#11-camera-摄像机)
12. [Light 光源与光照设置](#12-light-光源与光照设置)
13. [碰撞检测与物理](#13-碰撞检测与物理)

---

## 1. 生命周期

| 函数 | 调用时机 | 次数 |
|---|---|---|
| `Awake()` | 对象创建时（类似构造函数） | 一次 |
| `OnEnable()` | 对象每次激活时 | 多次 |
| `Start()` | 第一次帧更新前 | 一次 |
| `FixedUpdate()` | 固定物理帧（默认 0.02s） | 多次 |
| `Update()` | 每帧逻辑更新 | 多次 |
| `LateUpdate()` | Update 之后（常用于摄像机跟随） | 多次 |
| `OnDisable()` | 对象失活时 | 多次 |
| `OnDestroy()` | 对象销毁时 | 一次 |

**调用顺序：**

```
Awake → OnEnable → Start → FixedUpdate → Update → LateUpdate → OnDisable → OnDestroy
```

---

## 2. Inspector 检查器与特性

### 2.1 变量显示规则

- **私有/保护变量**：默认不在 Inspector 显示
  ```csharp
  [SerializeField]
  private int value;
  ```

- **公共变量**：默认显示
  ```csharp
  public int value;
  ```

- **隐藏公共变量**：
  ```csharp
  [HideInInspector]
  public int value;
  ```

- **自定义类/结构体显示**：
  ```csharp
  [System.Serializable]
  public class MyData { }
  ```

> **注意：** 字典、自定义结构体、自定义类默认不可显示，需加 `[System.Serializable]`。

### 2.2 常用辅助特性

| 特性 | 功能 | 示例 |
|---|---|---|
| `[Header("标题")]` | 为变量添加分组标题 | `[Header("玩家属性")]` |
| `[Tooltip("说明")]` | 鼠标悬停时显示提示 | `[Tooltip("移动速度")]` |
| `[Space(n)]` | 添加 n 像素空白间距 | `[Space(10)]` |
| `[Range(min, max)]` | 滑动条调整值 | `[Range(0, 100)]` |
| `[Multiline(n)]` | 字符串多行显示（n 行） | `[Multiline(3)]` |
| `[TextArea(min, max)]` | 可滚动文本区域，超过 min 行滚动至 max 行 | `[TextArea(2, 5)]` |
| `[ContextMenuItem("按钮名", "方法名")]` | 右键变量显示可点击的菜单项 | `[ContextMenuItem("重置", "ResetValue")]` |
| `[ContextMenu("测试函数")]` | 组件菜单「⋮」中添加测试入口 | `[ContextMenu("Do Test")]` |

---

## 3. MonoBehaviour 常用 API

### 3.1 获取自身信息

```csharp
this.gameObject.name;           // 对象名
this.transform.position;        // 世界坐标
this.transform.eulerAngles;     // 世界旋转角度
this.transform.lossyScale;      // 世界缩放（只读）
this.enabled = false;           // 失活当前脚本
this.enabled = true;            // 激活当前脚本
```

### 3.2 获取组件

```csharp
// 单个 — 泛型（推荐）
Rigidbody rb = GetComponent<Rigidbody>();

// 单个 — 字符串（不推荐）
Rigidbody rb = GetComponent("Rigidbody") as Rigidbody;

// 单个 — Type
Rigidbody rb = GetComponent(typeof(Rigidbody)) as Rigidbody;

// 多个
Rigidbody[] rbs = GetComponents<Rigidbody>();
List<Rigidbody> list = new List<Rigidbody>();
GetComponents<Rigidbody>(list);

// 子对象（单个 / 多个），参数 true 则包含失活对象
GetComponentInChildren<T>(true);
GetComponentsInChildren<T>(true, list);

// 父对象（单个 / 多个）
GetComponentInParent<T>();
GetComponentsInParent<T>(list);

// 尝试获取（返回 bool）
if (TryGetComponent<Rigidbody>(out Rigidbody rb)) { }
```

---

## 4. GameObject 常用 API

### 4.1 成员属性

```csharp
gameObject.name = "新名字";
gameObject.activeSelf;          // 是否激活
gameObject.isStatic;            // 是否静态
gameObject.layer;               // 层级
gameObject.tag;                 // 标签
gameObject.transform;           // Transform 引用
```

### 4.2 创建对象

```csharp
// 创建几何体
GameObject cube = GameObject.CreatePrimitive(PrimitiveType.Cube);

// 创建空物体
GameObject go = new GameObject();
GameObject go = new GameObject("名字");
GameObject go = new GameObject("名字", typeof(Rigidbody), typeof(BoxCollider));
```

### 4.3 查找对象

```csharp
// 单个 — 按名称（无法找到失活对象）
GameObject.Find("Player");

// 单个 — 按 Tag（相比 Find 更高效）
GameObject.FindWithTag("Player");

// 多个 — 按 Tag
GameObject[] enemies = GameObject.FindGameObjectsWithTag("Enemy");
```

### 4.4 实例化与删除

```csharp
// 实例化
Instantiate(prefab);
Instantiate(prefab, position, rotation);

// 延迟删除（下一帧执行，推荐）
Destroy(gameObject);
Destroy(gameObject, 2.0f);      // 延迟 2 秒

// 立即删除（当前帧）
DestroyImmediate(gameObject);

// 切换场景不销毁
DontDestroyOnLoad(gameObject);
```

> **提示：** 继承 `MonoBehaviour` 的类中可直接调用，省略 `GameObject.` 前缀。

### 4.5 其他成员方法

```csharp
// 添加组件
gameObject.AddComponent<Rigidbody>();

// 判断 Tag
if (gameObject.CompareTag("Enemy")) { }

// 激活/失活
gameObject.SetActive(false);
gameObject.SetActive(true);
```

---

## 5. Time 时间系统

### 5.1 时间缩放

```csharp
Time.timeScale = 0;     // 暂停
Time.timeScale = 1;     // 正常速度
Time.timeScale = 2;     // 两倍速
```

### 5.2 帧间隔时间

| 变量 | 说明 | 受 timeScale 影响 |
|---|---|---|
| `Time.deltaTime` | 当前帧逻辑时间间隔 | ✅ |
| `Time.unscaledDeltaTime` | 不受缩放影响的时间间隔 | ❌ |
| `Time.fixedDeltaTime` | 固定物理帧间隔 | ✅ |
| `Time.fixedUnscaledDeltaTime` | 不受缩放影响的物理帧间隔 | ❌ |

### 5.3 游戏运行时间

| 变量 | 说明 | 受 timeScale 影响 |
|---|---|---|
| `Time.time` | 游戏开始后总时间 | ✅ |
| `Time.unscaledTime` | 真实运行时间 | ❌ |
| `Time.frameCount` | 已渲染总帧数 | — |

### 5.4 位移公式

```
距离 = 方向 × 速度 × 时间
```

```csharp
transform.position += transform.forward * speed * Time.deltaTime;
```

---

## 6. Vector3 与 Transform

### 6.1 Vector3

结构体，表示三维向量或位置。

```csharp
Vector3 pos = new Vector3(x, y, z);
```

**常量：**

```csharp
Vector3.zero      // (0, 0, 0)
Vector3.forward   // (0, 0, 1)
Vector3.back      // (0, 0, -1)
Vector3.up        // (0, 1, 0)
Vector3.down      // (0, -1, 0)
Vector3.right     // (1, 0, 0)
Vector3.left      // (-1, 0, 0)
```

**距离计算：**

```csharp
float dist = Vector3.Distance(pointA, pointB);
```

**运算：** 两个 Vector3 的 x, y, z 分别进行加减乘除运算。

### 6.2 Transform — 位置

```csharp
// 世界坐标（Inspector 面板显示为相对父对象位置）
transform.position;

// 本地坐标（相对父对象）
transform.localPosition;

// 修改位置 — position 不能直接修改单个分量
transform.position = new Vector3(19, transform.position.y, transform.position.z);

// 或先存变量再改
Vector3 pos = transform.position;
pos.x = 10;
transform.position = pos;
```

**方向向量：**

```csharp
transform.forward;  // 物体自身 Z 轴朝向
transform.up;       // 物体自身 Y 轴朝向
transform.right;    // 物体自身 X 轴朝向
```

### 6.3 Transform — 位移

```csharp
// 手动计算
transform.position += transform.forward * speed * Time.deltaTime;

// API — 参数一：位移量，参数二：坐标系（默认 Space.Self）
transform.Translate(Vector3.forward * speed * Time.deltaTime);
transform.Translate(Vector3.forward * speed * Time.deltaTime, Space.World);
```

### 6.4 Transform — 旋转

```csharp
// 世界旋转角度
transform.eulerAngles;

// 本地旋转角度（相对父对象）
transform.localEulerAngles;

// 自转 — 绕自身轴
transform.Rotate(new Vector3(0, 10, 0) * Time.deltaTime);
transform.Rotate(Vector3.right, 10 * Time.deltaTime);

// 绕点旋转 — 参数：中心点、轴、角度
transform.RotateAround(Vector3.zero, Vector3.right, 10 * Time.deltaTime);
```

### 6.5 Transform — 缩放

```csharp
// 世界缩放（只读）
transform.lossyScale;

// 本地缩放（可读写）
transform.localScale;

// 逐渐缩放 — 无内置 API，手动计算
transform.localScale += Vector3.one * Time.deltaTime;
```

### 6.6 Transform — 看向

```csharp
// 看向世界坐标点
transform.LookAt(new Vector3(0, 0, 0));

// 看向另一个物体
public Transform target;
transform.LookAt(target);
```

---

## 7. 父子关系

### 7.1 设置父对象

```csharp
// 获取父对象
Transform parent = transform.parent;

// 取消父对象
transform.parent = null;

// 设置父对象
transform.parent = GameObject.Find("Parent").transform;

// API 设置 — 参数二：是否保留世界坐标/旋转/缩放
transform.SetParent(parentObj.transform, true);
```

### 7.2 断开子对象

```csharp
// 解除所有子对象的父子关系
transform.DetachChildren();
```

### 7.3 查找与操作子对象

```csharp
// 按名字查找 — 可找到失活对象
Transform child = transform.Find("ChildName");

// 遍历子对象
Transform child = transform.GetChild(index);

// 判断是否为某对象的子对象
bool isChild = son.IsChildOf(transform);

// 获取子对象索引
int index = son.GetSiblingIndex();

// 设置子对象顺序
son.SetAsFirstSibling();
son.SetAsLastSibling();
son.SetSiblingIndex(2);
```

---

## 8. 坐标转换

### 8.1 世界坐标 → 本地坐标

```csharp
// 点
Vector3 localPoint = transform.InverseTransformPoint(worldPoint);

// 方向
Vector3 localDir = transform.InverseTransformDirection(worldDir);
```

### 8.2 本地坐标 → 世界坐标

```csharp
// 点
Vector3 worldPoint = transform.TransformPoint(localPoint);

// 方向
Vector3 worldDir = transform.TransformDirection(localDir);
```

---

## 9. 输入系统 Input

### 9.1 鼠标

```csharp
// 屏幕位置 — 左下角原点，x 右、y 上，z 恒为 0
Vector3 mousePos = Input.mousePosition;

// 按下：0 = 左键、1 = 右键、2 = 中键
Input.GetMouseButtonDown(0);
Input.GetMouseButtonUp(0);
Input.GetMouseButton(0);        // 按住时持续触发

// 滚轮：-1 向下，0 无滚动，1 向上
float scroll = Input.mouseScrollDelta.y;
```

### 9.2 键盘

```csharp
// KeyCode 方式（推荐）
Input.GetKeyDown(KeyCode.W);
Input.GetKeyUp(KeyCode.W);
Input.GetKey(KeyCode.W);        // 按住时每帧触发

// 字符串方式 — 字母必须小写
Input.GetKeyDown("w");
```

### 9.3 轴输入

配置路径：Edit → Project Settings → Input Manager → Axes

```csharp
// 返回 -1 到 1 之间的渐变值，带平滑过渡
float horizontal = Input.GetAxis("Horizontal");
float vertical = Input.GetAxis("Vertical");

// 无平滑过渡，直接返回 -1/0/1
float raw = Input.GetAxisRaw("Horizontal");
```

### 9.4 其他输入

```csharp
// 是否有任意键或鼠标按住
Input.anyKey;

// 是否有任意键或鼠标按下（当前帧）
Input.anyKeyDown;

// 当前帧键盘输入字符串
string input = Input.inputString;

// 连接的手柄名称
string[] joystickNames = Input.GetJoystickNames();
```

### 9.5 触摸与陀螺仪

```csharp
// 多点触控开关
Input.multiTouchEnabled = false;

// 触摸检测
if (Input.touchCount > 0)
{
    Touch touch = Input.touches[0];
}

// 陀螺仪
Input.gyro.enabled = true;
Vector3 gravity = Input.gyro.gravity;
```

---

## 10. Screen 屏幕相关

### 10.1 静态属性

```csharp
// 当前分辨率
Resolution res = Screen.currentResolution;
Debug.Log($"{res.width} × {res.height}");

// 游戏窗口宽高
int w = Screen.width;
int h = Screen.height;

// 禁止屏幕休眠
Screen.sleepTimeout = SleepTimeout.NeverSleep;

// 运行时全屏
Screen.fullScreen = true;
Screen.fullScreenMode = FullScreenMode.FullScreenWindow;
```

### 10.2 设置分辨率

```csharp
// 参数：宽、高、是否全屏
Screen.SetResolution(1920, 1080, false);
Screen.SetResolution(1920, 1080, FullScreenMode.FullScreenWindow);
```

---

## 11. Camera 摄像机

### 11.1 常用参数

| 参数 | 说明 |
|---|---|
| `Clear Flags` | Skybox（3D 常用）/ Solid Color（2D）/ Depth Only（只画该层）/ Don't Clear（覆盖渲染） |
| `Culling Mask` | 选择性渲染部分层级 |
| `Projection` | Perspective（透视）/ Orthographic（正交，常用于 2D） |
| `Field of View` | 透视模式视口大小 |
| `Size` | 正交模式渲染范围 |
| `Clipping Planes` | 近/远裁剪平面距离 |
| `Depth` | 渲染顺序，值越大越靠后渲染，会覆盖之前画面 |
| `Target Texture` | 渲染到 Render Texture |
| `Viewport Rect` | 视口范围（x, y, w, h），用于分屏/多视角 |
| `Occlusion Culling` | 遮挡剔除，被挡物体不渲染 |
| `Allow HDR` | 是否允许高动态范围渲染（HDR） |
| `Allow MSAA` | 是否允许多重采样抗锯齿 |
| `Allow Dynamic Resolution` | 是否允许动态分辨率渲染 |
| `Target Display` | 指定输出到哪个显示器，用于多屏平台游戏开发 |

### 11.2 代码获取摄像机

```csharp
// 主摄像机 — 需要摄像机的 Tag 为 "MainCamera"
Camera main = Camera.main;

// 摄像机数量
int count = Camera.allCamerasCount;

// 所有摄像机
Camera[] cameras = Camera.allCameras;
```

### 11.3 渲染委托

```csharp
// 剔除前
Camera.onPreCull += (cam) => { };

// 渲染前
Camera.onPreRender += (cam) => { };

// 渲染后
Camera.onPostRender += (cam) => { };
```

### 11.4 坐标转换

```csharp
// 世界坐标 → 屏幕坐标（常用于血条、头顶 UI）
Vector3 screenPos = Camera.main.WorldToScreenPoint(worldPos);

// 屏幕坐标 → 世界坐标
Vector3 mousePos = Input.mousePosition;
mousePos.z = 5;                             // 设置距摄像机的深度距离
Vector3 worldPos = Camera.main.ScreenToWorldPoint(mousePos);
```

---

## 12. Light 光源与光照设置

### 12.1 光源类型

| 类型 | 说明 |
|---|---|
| `Directional` | 方向光（环境光），模拟太阳 |
| `Point` | 点光源，向所有方向发光 |
| `Spot` | 聚光灯，锥形范围发光 |
| `Area` | 区域光（仅烘焙模式可用） |

### 12.2 光照模式

| 模式 | 说明 |
|---|---|
| `Realtime` | 实时光照，效果好，性能消耗大 |
| `Baked` | 预先烘焙到光照贴图，无法动态变化 |
| `Mixed` | 混合模式，静态物体烘焙，动态物体实时 |

### 12.3 常用参数

```csharp
light.color = Color.red;          // 颜色
light.intensity = 1.5f;           // 亮度
light.shadowType = LightShadows.Soft;   // 阴影：None / Hard / Soft
light.cookie = shadowCookie;      // 投影遮罩
light.cullingMask = LayerMask.GetMask("Default");  // 光照层级

// 光晕/耀斑
light.drawHalo = true;            // 球形光环
light.flare = flareAsset;         // 光斑耀斑
```

### 12.4 光照面板

路径：**Window → Rendering → Lighting Settings**

- **Skybox Material** — 更换天空盒
- **Sun Source** — 指定方向光源
- **Environment Lighting** — 环境光相关设置

---

## 13. 碰撞检测与物理

### 13.1 碰撞条件

两个物体产生碰撞响应的必要条件：
1. 两个物体都有 **Collider**（碰撞器）
2. 至少一个物体有 **Rigidbody**（刚体）

### 13.2 Rigidbody 刚体

使物体受物理系统影响：重力、力、碰撞响应。

```csharp
// 获取
Rigidbody rb = GetComponent<Rigidbody>();

// 质量
rb.mass = 1.0f;

// 使用重力
rb.useGravity = true;

// 运动学（不受物理力但仍参与碰撞）
rb.isKinematic = false;

// 冻结位置/旋转
rb.constraints = RigidbodyConstraints.FreezeRotation;
```

**参数详解：**

| 参数 | 说明 |
|---|---|
| `Mass` | 质量（默认千克）。质量越大，惯性越大 |
| `Drag` | 空气阻力。用力移动对象时的阻力大小，0 表示无阻力 |
| `Angular Drag` | 角阻力。扭矩旋转对象时的空气阻力大小，0 表示无阻力 |
| `Use Gravity` | 是否受重力影响 |
| `Is Kinematic` | 启用后物体不受物理引擎驱动，仅可通过 Transform 操作。适用于移动平台、动画化铰链关节等 |
| `Interpolate` | 插值运算，让刚体移动更平滑 |
| `Collision Detection` | 碰撞检测模式：Discrete / Continuous / Continuous Dynamic / Continuous Speculative。防止快速移动对象穿过其他物体（穿模） |
| `Constraints` | 约束，限制刚体位置/旋转自由度 |

> **碰撞检测模式说明：** `Discrete`（默认，每帧检测，快速物体可能穿透）、`Continuous`（与静态几何体持续检测）、`Continuous Dynamic`（与其他动态刚体持续检测，开销最大）、`Continuous Speculative`（推测性连续检测，性能较好）

### 13.3 Collider 碰撞器

常用 3D 碰撞器：

| 类型 | 说明 |
|---|---|
| `BoxCollider` | 盒形碰撞器 |
| `SphereCollider` | 球形碰撞器 |
| `CapsuleCollider` | 胶囊形（常用于角色） |
| `MeshCollider` | 网格碰撞器（精确但开销大） |

```csharp
// 获取碰撞器
Collider col = GetComponent<Collider>();

// 是否触发（不产生物理碰撞，仅触发事件）
col.isTrigger = true;

// 碰撞体尺寸
BoxCollider box = GetComponent<BoxCollider>();
box.size = new Vector3(1, 2, 1);
```

### 13.4 碰撞回调

```csharp
// 碰撞事件（非 Trigger）
void OnCollisionEnter(Collision collision) { }
void OnCollisionStay(Collision collision) { }
void OnCollisionExit(Collision collision) { }

// 触发事件（勾选 isTrigger）
void OnTriggerEnter(Collider other) { }
void OnTriggerStay(Collider other) { }
void OnTriggerExit(Collider other) { }
```

---

## 学习建议路线

1. 生命周期 — 理解脚本执行顺序
2. GameObject 与 Component — 掌握对象和组件操作
3. Transform — 移动、旋转、缩放
4. Input 输入 — 鼠标键盘响应
5. Time — 帧率无关的运动控制
6. Camera — 摄像机控制与坐标转换
7. Rigidbody 与 Collider — 物理与碰撞
8. 综合实践 — 制作小游戏巩固

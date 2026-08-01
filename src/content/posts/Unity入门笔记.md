---
title: Unity 入门阶段笔记 Stage 1
published: 2026-07-20
updated: 2026-08-01
description: Unity 常用 API 速查 — 生命周期、Transform、Input、Camera、物理、协程、Debug 等
tags: [unity, 游戏开发, 基础知识, 速查]
category: 基础知识
slug: unity-core-scripting-cheatsheet
pinned: true
---

# Unity 入门阶段笔记 Stage 1

## 一、MonoBehaviour 生命周期函数
| 函数 | 执行时机 | 调用次数 | 核心用途 |
|------|----------|----------|----------|
| `Awake` | 对象实例化瞬间 | 生命周期仅1次 | 初始化组件、获取引用，类似构造函数 |
| `OnEnable` | 对象每次从失活→激活 | 每次激活都执行 | 注册事件、开启显示、刷新状态 |
| `Start` | 首次 Update 帧更新前 | 生命周期仅1次 | 依赖其他对象的初始化逻辑 |
| `FixedUpdate` | 固定物理帧率执行 | 固定间隔调用 | 物理运动、刚体、碰撞逻辑 |
| `Update` | 每渲染帧执行 | 随设备帧率变化 | 输入检测、常规游戏逻辑 |
| `LateUpdate` | 所有 Update 执行完毕后 | 每渲染帧执行 | 相机跟随、位置后处理修正 |
| `OnDisable` | 对象从激活→失活 | 每次失活都执行 | 注销事件、暂停逻辑 |
| `OnDestroy` | 对象被销毁时 | 销毁前执行1次 | 释放资源、清理缓存 |

> **补充规则：**
> 1. 场景加载时所有物体先统一执行完 `Awake`，再统一执行 `Start`
> 2. `SetActive(false)` 仅触发 `OnDisable`，不会调用 `OnDestroy`
> 3. `Destroy()` 默认延迟到当前帧末尾才移除对象（非下一帧）
> 4. 脚本 `enabled = false` 不会触发 `OnDisable`，仅阻止 `Update` 系列调用

---

## 二、Inspector 检查器序列化特性
### 2.1 基础显示规则
- `public` 字段：默认显示在 Inspector 面板
- `private / protected` 字段：默认隐藏，添加 `[SerializeField]` 可强制序列化显示
- 隐藏 `public` 字段：添加 `[HideInInspector]`
- 属性（`{ get; set; }`）：默认不序列化，加 `[field: SerializeField]` 可显示自动属性
- `Dictionary`、自定义 `struct`、自定义 `class` 默认不显示，自定义类型需标记 `[System.Serializable]`

### 2.2 常用辅助特性
```csharp
[Header("分组标题")]                       // 变量分组说明
[Tooltip("悬停提示内容")]                   // 鼠标悬停显示注释
[Space(10)]                                // 字段间空行间隔
[Range(0, 10)]                             // 数值滑条（仅 float/int）
[Multiline(3)]                             // 字符串多行输入框
[TextArea(2, 10)]                          // 带滚动条的文本域（minLines, maxLines）
[ContextMenuItem("按钮名", "方法名")]        // 字段右键菜单
[ContextMenu("测试函数")]                   // 组件右上角快捷菜单
[RequireComponent(typeof(Rigidbody))]       // 自动挂载依赖组件
[DisallowMultipleComponent]                 // 禁止重复挂载到同一物体
[ExecuteAlways]                             // 编辑器模式也执行生命周期
```

---

## 三、MonoBehaviour 核心成员与方法
### 3.1 核心成员变量
```csharp
gameObject   // 当前脚本挂载的游戏物体
transform    // 物体的 Transform 变换组件
enabled      // 脚本自身的启用/禁用状态
// 示例：this.enabled = false; 禁用脚本
```

### 3.2 组件获取方法
#### 获取单个组件（优先泛型写法）
```csharp
// 1. 泛型获取（推荐，编译安全）
脚本类型 变量名 = GetComponent<脚本类型>();

// 2. Type 类型获取
脚本类型 变量名 = GetComponent(typeof(脚本类型)) as 脚本类型;

// 3. 字符串名称获取（不推荐）
脚本类型 变量名 = GetComponent("脚本名") as 脚本类型;

// 4. 安全尝试获取（避免空引用报错）
if (TryGetComponent<脚本类型>(out 变量名))
{
    // 获取成功后执行逻辑
}
```

#### 获取多个组件
```csharp
// 数组形式
脚本类型[] 数组名 = GetComponents<脚本类型>();

// 列表形式（复用List减少GC）
List<脚本类型> 列表名 = new List<脚本类型>();
GetComponents<脚本类型>(列表名);
```

#### 父子物体组件获取
```csharp
// 子物体获取（参数true：包含失活物体；默认false：仅激活物体）
GetComponentInChildren<脚本类型>(true);
GetComponentsInChildren<脚本类型>(true);

// 父物体获取
GetComponentInParent<脚本类型>();
GetComponentsInParent<脚本类型>();
```

---

## 四、GameObject 游戏物体
### 4.1 成员变量
```csharp
gameObject.name        // 物体名称
gameObject.activeSelf  // 物体是否激活（只读）
gameObject.isStatic    // 是否为静态物体
gameObject.layer       // 物体层级
gameObject.tag         // 物体标签
gameObject.transform   // 变换组件引用
```

### 4.2 静态方法
#### 创建与查找
```csharp
// 创建内置几何体
GameObject cube = GameObject.CreatePrimitive(PrimitiveType.Cube);

// 查找物体（仅能找到激活物体）
GameObject obj = GameObject.Find("物体名");
GameObject obj = GameObject.FindWithTag("标签名");
GameObject[] objs = GameObject.FindGameObjectsWithTag("标签名");
```

#### 实例化与销毁
```csharp
// 克隆/实例化物体
GameObject clone = Instantiate(预制体对象);

// 销毁物体（第二参数为延迟秒数）
Destroy(游戏物体, 2f);
Destroy(this); // 销毁脚本自身

// 立即销毁（当前帧移除，少用）
DestroyImmediate(游戏物体);

// 切换场景不销毁该物体
DontDestroyOnLoad(gameObject);
```

### 4.3 成员方法
```csharp
// 创建空物体
GameObject empty = new GameObject();
GameObject obj = new GameObject("物体名", typeof(Rigidbody), typeof(BoxCollider));

// 添加组件
Rigidbody rb = gameObject.AddComponent<Rigidbody>();

// 标签对比
if (gameObject.CompareTag("Player")) { }

// 设置激活/失活
gameObject.SetActive(true);
gameObject.SetActive(false);
```

> 不常用方法：`SendMessage`、`SendMessageUpwards`，性能差且不易维护，不推荐使用。

---

## 五、Time 时间系统
| 属性 | 说明 | 受 timeScale 影响 |
|------|------|-------------------|
| `Time.deltaTime` | 上一帧到当前帧的间隔（位移必备） | ✅ |
| `Time.unscaledDeltaTime` | 不受时间缩放的帧间隔 | ❌ |
| `Time.time` | 游戏开始到当前的总时间 | ✅ |
| `Time.unscaledTime` | 不受时间缩放的总时间 | ❌ |
| `Time.fixedDeltaTime` | 物理帧间隔（默认 0.02s = 50FPS） | 只读 |
| `Time.fixedUnscaledDeltaTime` | 不受时间缩放的物理帧间隔 | ❌ |
| `Time.frameCount` | 游戏运行总帧数 | — |
| `Time.timeScale` | 时间缩放（0=暂停，1=正常，2=2倍速） | — |
| `Time.smoothDeltaTime` | 平滑后的 deltaTime（用于 UI 帧率显示） | ✅ |

> **核心公式：位移 = 速度 × 时间**
> ```csharp
> transform.position += direction * speed * Time.deltaTime;
> ```

---

## 六、位置与位移
### 6.1 Vector3 常用操作
```csharp
// 构造
Vector3 pos = new Vector3(x, y, z);

// 常用方向常量
Vector3.zero     // (0, 0, 0)
Vector3.one      // (1, 1, 1)
Vector3.right    // (1, 0, 0)  世界 X 轴正方向
Vector3.left     // (-1, 0, 0)
Vector3.up       // (0, 1, 0)  世界 Y 轴正方向
Vector3.down     // (0, -1, 0)
Vector3.forward  // (0, 0, 1)  世界 Z 轴正方向
Vector3.back     // (0, 0, -1)

// 常用计算
float distance = Vector3.Distance(a, b);       // 两点距离
Vector3 dir = (target - origin).normalized;     // 归一化方向
Vector3 lerp = Vector3.Lerp(a, b, t);          // 线性插值
Vector3 move = Vector3.MoveTowards(cur, target, maxDelta); // 匀速逼近
float dot = Vector3.Dot(a, b);                 // 点积（判断前后/夹角）
Vector3 cross = Vector3.Cross(a, b);           // 叉积（求法线/垂直方向）
```

### 6.2 位置坐标
```csharp
transform.position       // 世界坐标系位置
transform.localPosition  // 相对父物体的本地坐标（面板显示值）

// 物体自身朝向向量
transform.forward  // 物体正前方
transform.up       // 物体正上方
transform.right    // 物体正右方
```

> **注意：** `transform.position` 是 struct 属性，不能直接修改单个分量：
> ```csharp
> // ❌ 错误
> transform.position.x = 5;
> // ✅ 正确
> transform.position = new Vector3(5, transform.position.y, transform.position.z);
> ```

### 6.3 位移移动
```csharp
// 方式1：直接计算
transform.position += transform.forward * speed * Time.deltaTime;

// 方式2：Translate API（第二参数可选 Space.Self / Space.World）
transform.Translate(Vector3.forward * speed * Time.deltaTime, Space.Self);
```

---

## 七、角度与旋转
### 7.1 欧拉角与四元数
```csharp
// 欧拉角（直观但可能有万向锁）
transform.eulerAngles       // 世界坐标系欧拉角
transform.localEulerAngles  // 相对父物体的本地欧拉角

// 四元数（推荐，无万向锁）
transform.rotation          // 世界坐标系旋转（Quaternion）
transform.localRotation     // 相对父物体的旋转（Quaternion）
Quaternion.identity         // 无旋转
```

### 7.2 旋转方法
```csharp
// 自转（绕自身轴旋转）
transform.Rotate(new Vector3(0, 90, 0) * Time.deltaTime);
transform.Rotate(Vector3.up, 30f * Time.deltaTime);  // 绕指定世界轴

// 绕指定点公转
transform.RotateAround(Vector3.zero, Vector3.up, 20f * Time.deltaTime);

// 常用四元数旋转
transform.rotation = Quaternion.Euler(0, 45, 0);               // 欧拉角 → 四元数
transform.rotation = Quaternion.LookRotation(forward, up);      // 朝向方向
Quaternion target = Quaternion.FromToRotation(from, to);        // 从方向A转到B
Quaternion lerp = Quaternion.Lerp(a, b, t);                    // 线性插值
Quaternion slerp = Quaternion.Slerp(a, b, t);                  // 球面插值（旋转更平滑）
```

---

## 八、缩放与看向
### 8.1 缩放
```csharp
transform.localScale    // 本地缩放（可读写）
transform.lossyScale    // 世界总缩放（只读，不可赋值）

// 缩放无专用渐变API，自行计算
transform.localScale += Vector3.one * Time.deltaTime;
```

### 8.2 LookAt 看向
```csharp
// 让物体Z轴朝向目标点/目标物体
transform.LookAt(targetPosition);
transform.LookAt(targetTransform);
```

---

## 九、父子关系
### 9.1 父对象操作
```csharp
// 获取父物体
Transform parent = transform.parent;

// 解除父子关系
transform.parent = null;

// 设置父物体（第二参数：是否保留世界坐标/旋转/缩放）
transform.SetParent(newParent, true);
```

### 9.2 子对象操作
```csharp
// 解除所有子物体
transform.DetachChildren();

// 按名字查找子物体（可找到失活物体）
Transform child = transform.Find("子物体名");

// 按索引获取子物体
Transform firstChild = transform.GetChild(0);
```

### 9.3 子物体层级排序
```csharp
child.IsChildOf(parent);        // 判断是否为子物体
child.GetSiblingIndex();        // 获取自身索引
child.SetAsFirstSibling();      // 设为第一个子物体
child.SetAsLastSibling();       // 设为最后一个子物体
child.SetSiblingIndex(2);       // 设为指定索引位置
```

---

## 十、坐标转换
```csharp
// 世界坐标 → 本地坐标
Vector3 localPos = transform.InverseTransformPoint(worldPos);
Vector3 localDir = transform.InverseTransformDirection(worldDir);

// 本地坐标 → 世界坐标
Vector3 worldPos = transform.TransformPoint(localPos);
Vector3 worldDir = transform.TransformDirection(localDir);
```

---

## 十一、输入系统 Input（旧版）

> **注意：** 以下为旧版 Input Manager API，适合快速原型。新项目推荐使用 **Input System** 包（`UnityEngine.InputSystem`），支持事件驱动、多设备映射、Input Action 资产管理。

### 11.1 鼠标输入
```csharp
// 鼠标屏幕位置（左下角为原点，z恒为0）
Vector3 mousePos = Input.mousePosition;

// 鼠标按键：0=左键，1=右键，2=中键
Input.GetMouseButton(0);      // 按住持续触发
Input.GetMouseButtonDown(0);  // 按下瞬间触发
Input.GetMouseButtonUp(0);    // 抬起瞬间触发

// 鼠标滚轮
Input.mouseScrollDelta.y; // -1向下，0无操作，1向上
```

### 11.2 键盘输入
```csharp
Input.GetKey(KeyCode.W);          // 按住持续触发
Input.GetKeyDown(KeyCode.Space);  // 按下瞬间触发
Input.GetKeyUp(KeyCode.Space);    // 抬起瞬间触发
```

### 11.3 轴输入
```csharp
// 平滑过渡，返回 -1 ~ 1
float h = Input.GetAxis("Horizontal");
float v = Input.GetAxis("Vertical");

// 无过渡，仅 -1 / 0 / 1 三档
float hRaw = Input.GetAxisRaw("Horizontal");
```

### 11.4 其他输入
```csharp
Input.anyKey;        // 任意键/鼠标按住
Input.anyKeyDown;    // 任意键/鼠标按下瞬间
Input.inputString;   // 本帧键盘输入字符
Input.GetJoystickNames(); // 获取已连接手柄名称
```

### 11.5 移动端输入
```csharp
// 触摸检测
if (Input.touchCount > 0)
{
    Touch touch = Input.touches[0];
}

Input.multiTouchEnabled = false; // 禁用多点触控

// 陀螺仪
Input.gyro.enabled = true;
Input.gyro.gravity; // 重力加速度向量
```

---

## 十二、Screen 屏幕系统
### 12.1 静态属性
```csharp
// 当前显示器分辨率
Resolution res = Screen.currentResolution;
int width = res.width;
int height = res.height;

// 游戏窗口宽高
Screen.width;
Screen.height;

// 屏幕休眠模式
Screen.sleepTimeout = SleepTimeout.NeverSleep; // 永不熄屏

// 全屏模式
Screen.fullScreenMode = FullScreenMode.FullScreenWindow;
```

### 12.2 静态方法
```csharp
// 设置分辨率（宽，高，是否全屏）
Screen.SetResolution(1920, 1080, false);
```

---

## 十三、Camera 摄像机面板参数
### 13.1 核心参数
| 参数 | 说明 |
|------|------|
| Clear Flags | 背景清除方式：Skybox天空盒 / Solid Color纯色 / Depth Only仅深度 / Don't Clear不清除 |
| Culling Mask | 选择性渲染指定层级的物体 |
| Projection | 投影模式：Perspective透视（3D） / Orthographic正交（2D） |
| Clipping Planes | 裁剪平面：Near近裁剪面 / Far远裁剪面 |
| Depth | 渲染深度，值越大越后渲染，画面在上层 |
| Target Texture | 渲染纹理，将画面输出到RenderTexture |
| Occlusion Culling | 遮挡剔除，被挡住的物体不渲染 |
| Viewport Rect | 视口范围，控制摄像机画面在窗口的位置和大小 |
| Allow HDR | 高动态光照渲染，支持更大明暗反差、光晕特效，性能开销更高 |
| Allow MSAA | 多重采样抗锯齿，消除物体边缘锯齿，性能紧张场景可关闭 |
| Allow Dynamic Resolution | 动态调整渲染分辨率，高负载时自动降分辨率保帧率 |
| Target Display | 指定输出到几号显示器，多用于多屏游戏开发 |

---

## 十四、Camera 代码 API
### 14.1 静态成员
```csharp
Camera.main;          // 获取标签为 MainCamera 的主摄像机
Camera.allCamerasCount; // 场景中摄像机总数
Camera[] allCams = Camera.allCameras; // 所有摄像机数组

// 渲染生命周期委托
Camera.onPreCull += cam => { /* 剔除前 */ };
Camera.onPreRender += cam => { /* 渲染前 */ };
Camera.onPostRender += cam => { /* 渲染后 */ };
```

### 14.2 坐标转换
```csharp
// 世界坐标 → 屏幕坐标（常用于血条跟随）
Vector3 screenPos = Camera.main.WorldToScreenPoint(transform.position);

// 屏幕坐标 → 世界坐标（需设置z轴距离）
Vector3 mouseWorld = Input.mousePosition;
mouseWorld.z = 5f;
Vector3 worldPos = Camera.main.ScreenToWorldPoint(mouseWorld);
```

---

## 十五、光源 Light 组件
### 15.1 光源类型
| 类型 | 说明 |
|------|------|
| Spot | 聚光灯，锥形照射范围 |
| Directional | 方向光，模拟太阳光/环境光 |
| Point | 点光源，向四周发散 |
| Area | 面光源，仅烘焙生效 |

### 15.2 核心参数
- **Mode**：RealTime实时（性能高、可动态） / Baked烘焙（预计算、不可变） / Mixed混合
- **Color**：光源颜色
- **Intensity**：光源亮度
- **Shadow Type**：No Shadows无阴影 / Hard Shadows硬阴影 / Soft Shadows软阴影
- **Cookie**：投影遮罩纹理
- **Draw Halo**：球形光晕开关
- **Flare**：镜头耀斑效果
- **Culling Mask**：仅影响指定层级的物体

---

## 十六、光照全局设置
面板路径：`Window → Rendering → Lighting Settings`
- **Skybox Material**：设置天空盒材质
- **Sun Source**：指定方向光作为太阳
- **Environment Lighting**：环境光参数设置

---

## 十七、物理碰撞检测
### 17.1 碰撞产生必要条件
1. 两个物体都必须挂载**碰撞体（Collider）**组件
2. 至少其中一个物体挂载**刚体（Rigidbody）**组件

### 17.2 刚体 Rigidbody 参数
| 参数 | 说明 |
|------|------|
| Mass | 质量（单位：千克），质量越大惯性越大 |
| Drag | 直线运动空气阻力，0为无阻力 |
| Angular Drag | 旋转运动空气阻力，0为无阻力 |
| Use Gravity | 是否受重力影响 |
| Is Kinematic | 启用后不受物理引擎驱动，仅通过Transform操作，适用于移动平台、动画驱动物体 |
| Interpolate | 插值运算，让刚体物体移动更平滑 |
| Collision Detection | 碰撞检测模式，防止快速移动物体穿墙 |
| Constraints | 运动约束，限制刚体在某轴的位移或旋转 |

### 17.3 碰撞器 Collider
#### 3D碰撞器种类
- 盒状碰撞体（Box Collider）
- 球状碰撞体（Sphere Collider）
- 胶囊碰撞体（Capsule Collider）
- 网格碰撞体（Mesh Collider）
- 轮胎碰撞体（Wheel Collider）
- 地形碰撞体（Terrain Collider）

#### 共同参数
- **Is Trigger**：是否为触发器，启用后仅检测碰撞，无物理碰撞效果
- **Material**：物理材质，决定碰撞交互表现（摩擦力、弹性等）
- **Center**：碰撞体在局部空间的中心点位置

#### 常用碰撞体参数
- **盒状碰撞体**：Size 控制XYZ三轴大小
- **球状碰撞体**：Radius 控制球形半径
- **胶囊碰撞体**：Radius 半径、Height 高度、Direction 轴向

#### 复杂碰撞体组合
异形物体可通过多个子对象挂载基础碰撞体组合实现，子对象会继承父对象的刚体，以此拼接复杂碰撞外形。

#### 不常用碰撞体说明
- 网格碰撞体：按模型网格面精确碰撞，性能开销大；勾选 Convex 才可搭配刚体使用（最多255面）
- 地形碰撞体：适配地形组件，性能开销较高
- 轮胎碰撞体：专门用于载具轮胎物理

### 17.4 物理材质 Physic Material
| 参数 | 说明 |
|------|------|
| Dynamic Friction | 运动时的摩擦力 |
| Static Friction | 静止时的摩擦力，数值越大越难被推动 |
| Bounciness | 表面弹性，数值越小弹力越小 |
| Friction Combine | 两个物体摩擦力的组合方式：Average平均 / Minimum最小 / Maximum最大 / Multiply相乘 |
| Bounce Combine | 两个物体弹性的组合方式，规则同摩擦力组合 |

---


## 十八、Debug 调试
```csharp
Debug.Log("普通日志");
Debug.LogWarning("警告");
Debug.LogError("错误");

---

## 十九、实用代码片段
```csharp
// 平滑过渡（Lerp）
transform.position = Vector3.Lerp(transform.position, target, Time.deltaTime * speed);

// 物体朝向目标（2D，Z 轴旋转）
Vector3 dir = target.position - transform.position;
float angle = Mathf.Atan2(dir.y, dir.x) * Mathf.Rad2Deg;
transform.rotation = Quaternion.Euler(0, 0, angle);

// 随机数
int rand = Random.Range(0, 10);           // int [0, 10)
float randF = Random.Range(0f, 1f);       // float [0, 1]
```

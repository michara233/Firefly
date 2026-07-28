\---

title: unity学习

published: 2026-07-20

description: 更新中ing...

tags: [unity, 游戏开发,基础知识]

category: 基础知识

slug: unity学习速查

pinned: true

\---

# Unity 学习笔记

> 基于原始 Unity
> 学习笔记整理，重新分类结构、统一术语、补充目录，方便复习查询。

## 目录

1.  Unity 生命周期
2.  Inspector 检查器与特性
3.  MonoBehaviour 常用 API
4.  GameObject 常用 API
5.  Time 时间系统
6.  Vector3 与 Transform
7.  输入系统 Input
8.  Screen 屏幕相关
9.  Camera 摄像机
10.  Light 光源
11.  碰撞检测基础

------------------------------------------------------------------------

# 1. Unity 生命周期

  函数          作用

------------- ----------------------------------

  Awake         类似构造函数，对象创建时调用一次
  OnEnable      对象激活时调用
  Start         第一次帧更新前调用一次
  FixedUpdate   固定物理帧更新，用于物理计算
  Update        每帧逻辑更新
  LateUpdate    Update 后执行，常用于摄像机跟随
  OnDisable     对象失活时调用
  OnDestroy     对象销毁时调用

------------------------------------------------------------------------

# 2. Inspector 检查器

## 变量显示规则

-   私有变量默认无法显示：

``` csharp
[SerializeField]
private int value;
```

-   公共变量默认显示：

``` csharp
public int value;
```

-   隐藏公共变量：

``` csharp
[HideInInspector]
public int value;
```

-   自定义类、结构体显示：

``` csharp
[System.Serializable]
public class Data{}
```

## 常用特性

  特性        功能

----------- ----------------

  Header      添加分组标题
  Tooltip     鼠标悬停提示
  Space       添加间距
  Range       滑动范围
  Multiline   多行文本
  TextArea    可滚动文本区域

------------------------------------------------------------------------

# 3. MonoBehaviour 常用 API

## 获取自身对象信息

``` csharp
this.gameObject.name;
this.transform.position;
this.transform.eulerAngles;
this.transform.lossyScale;
```

## 获取组件

### 获取单个组件

``` csharp
GetComponent<T>();
```

### 获取多个组件

``` csharp
GetComponents<T>();
```

### 获取子对象组件

``` csharp
GetComponentInChildren<T>();
GetComponentsInChildren<T>();
```

### 获取父对象组件

``` csharp
GetComponentInParent<T>();
GetComponentsInParent<T>();
```

### 尝试获取

``` csharp
TryGetComponent<T>(out T value);
```

------------------------------------------------------------------------

# 4. GameObject

## 常用成员

``` csharp
name
activeSelf
isStatic
layer
tag
transform
```

## 创建对象

``` csharp
GameObject.CreatePrimitive(PrimitiveType.Cube);
```

## 查找对象

``` csharp
GameObject.Find("名字");

GameObject.FindWithTag("Tag");
```

## 实例化与删除

``` csharp
Instantiate(obj);

Destroy(obj);
```

立即删除：

``` csharp
DestroyImmediate(obj);
```

场景切换保留：

``` csharp
DontDestroyOnLoad(gameObject);
```

## 添加脚本

``` csharp
AddComponent<T>();
```

------------------------------------------------------------------------

# 5. Time 时间系统

## 时间缩放

``` csharp
Time.timeScale = 0; //暂停
Time.timeScale = 1; //正常
Time.timeScale = 2; //两倍速
```

## 常用时间变量

  变量             说明

---------------- ----------------

  deltaTime        当前帧时间间隔
  fixedDeltaTime   物理帧间隔
  time             游戏运行时间
  frameCount       当前帧数

移动计算：

    距离 = 速度 × 时间

------------------------------------------------------------------------

# 6. Vector3 与 Transform

## Vector3

创建：

``` csharp
Vector3 pos = new Vector3(x,y,z);
```

常用：

``` csharp
Vector3.zero
Vector3.forward
Vector3.back
Vector3.up
Vector3.down
Vector3.left
Vector3.right
```

距离：

``` csharp
Vector3.Distance(a,b);
```

------------------------------------------------------------------------

# Transform

## 坐标

世界坐标：

``` csharp
transform.position;
```

本地坐标：

``` csharp
transform.localPosition;
```

## 方向

``` csharp
transform.forward;
transform.up;
transform.right;
```

## 移动

公式：

    方向 × 速度 × 时间

代码：

``` csharp
transform.position += transform.forward * speed * Time.deltaTime;
```

API：

``` csharp
transform.Translate();
```

## 旋转

``` csharp
transform.Rotate();
```

绕点旋转：

``` csharp
transform.RotateAround();
```

## 缩放

``` csharp
transform.localScale;
```

## 看向目标

``` csharp
transform.LookAt(target);
```

------------------------------------------------------------------------

# 7. 输入系统 Input

## 鼠标

``` csharp
Input.mousePosition;

Input.GetMouseButtonDown(0);
Input.GetMouseButtonUp(0);
Input.GetMouseButton(0);
```

## 键盘

``` csharp
Input.GetKeyDown(KeyCode.W);

Input.GetKey(KeyCode.W);

Input.GetKeyUp(KeyCode.W);
```

## 轴输入

``` csharp
Input.GetAxis("Horizontal");
Input.GetAxis("Vertical");
```

------------------------------------------------------------------------

# 8. Screen

获取分辨率：

``` csharp
Screen.width;
Screen.height;
```

设置分辨率：

``` csharp
Screen.SetResolution(1920,1080,false);
```

禁止休眠：

``` csharp
Screen.sleepTimeout = SleepTimeout.NeverSleep;
```

------------------------------------------------------------------------

# 9. Camera 摄像机

## 常用参数

-   Clear Flags：背景清除方式
-   Culling Mask：渲染层级
-   Projection：
    -   Perspective 透视
    -   Orthographic 正交
-   Depth：摄像机渲染顺序

## 获取摄像机

``` csharp
Camera.main;
```

## 坐标转换

世界转屏幕：

``` csharp
Camera.main.WorldToScreenPoint(pos);
```

屏幕转世界：

``` csharp
Camera.main.ScreenToWorldPoint(pos);
```

------------------------------------------------------------------------

# 10. Light 光源

## 类型

  类型          说明

------------- --------

  Directional   方向光
  Point         点光源
  Spot          聚光灯
  Area          区域光

## 光照模式

-   Realtime：实时计算
-   Baked：烘焙
-   Mixed：混合

常用参数：

-   Color
-   Intensity
-   Shadow Type
-   Cookie
-   Culling Mask

------------------------------------------------------------------------

# 11. 碰撞检测基础

碰撞产生条件：

1.  两个物体都有 Collider
2.  至少一个物体拥有 Rigidbody

## Rigidbody

作用：

-   受到物理力影响
-   支持重力
-   支持碰撞响应

## Collider

用于定义物体碰撞范围。

------------------------------------------------------------------------

# 学习建议

建议学习顺序：

1.  生命周期
2.  GameObject 与 Component
3.  Transform 移动旋转
4.  Input 输入
5.  Camera
6.  Rigidbody 与 Collider
7.  综合制作小游戏
---
title: C# 高级
published:
description: C# 进阶笔记 — 集合、泛型、委托与事件、协变逆变、多线程、反射特性、迭代器与常用语法糖
tags: [csharp, 基础知识, 速查]
category: 基础知识
---

# C#高级

入门课讲了语法和面向对象，这门课讲的是"框架级别"的东西：集合怎么选、泛型为什么能省掉拆箱、委托怎么变成事件、运行时怎么用反射回头看自己。下面对每一块都补了定义，光记 API 名字容易记混。

## 一、数据集合

`System.Collections` 命名空间下的非泛型集合，元素类型统一是 `object`。

### 1.1 ArrayList　//数组基类

**定义**：长度可变的数组。底层还是数组，装满了就按倍数扩容并把旧元素复制过去，所以它的容量（`Capacity`）通常大于元素个数（`Count`）。

```csharp
using System.Collections;

ArrayList array = new ArrayList();
```

| 操作 | 方法 | 说明 |
|------|------|------|
| 增 | `Add()` | 追加到末尾 |
| | `AddRange()` | 追加一整个集合 |
| | `Insert(位置, 数值)` | 插入到指定下标 |
| 删 | `Remove()` | 从头遍历，删掉第一个匹配的元素 |
| | `RemoveAt()` | 删指定下标 |
| | `Clear()` | 清空 |
| 查 | `array[0]` | 按下标取 |
| | `Contains()` | 是否存在该元素 |
| | `IndexOf()` | 返回第一个匹配的下标 |
| | `LastIndexOf()` | 从尾部往前找 |
| 改 | `array[0] = ...` | 按下标赋值 |

遍历：

```csharp
foreach (var item in array)
{
}
```

**装箱拆箱**：非泛型集合的元素类型是 `object`，而 `object` 是引用类型。

```csharp
int i = 1;
array[0] = i;        // 装箱：把值类型搬到堆上，外面套一层 object
i = (int)array[0];   // 拆箱：从 object 里把值取回来
```

每次存取都伴随一次内存分配或类型检查，元素一多就有开销——这也是泛型集合出现的原因。

### 1.2 排序

**1. 用自带的排序方法**

```csharp
list.Sort();
```

**2. 自定义类实现 `IComparable<T>`**

`CompareTo` 的返回值不是随便定的：小于返回负数，等于返回 0，大于返回正数。框架只认这三档符号。

```csharp
class Item : IComparable<Item>
{
    public int money;

    public Item(int money)
    {
        this.money = money;
    }

    public int CompareTo(Item other)
    {
        if (this.money > other.money)
            return 1;
        else if (this.money < other.money)
            return -1;
        else
            return 0;
    }
}

public static void Main(string[] args)
{
    List<Item> item = new List<Item>();
    item.Sort();
}
```

> 注意大小写：**C# 区分大小写**，`Sort()` 不是 `sort()`，`Main` 不是 `main`。

**3. 通过委托函数排序**

不想改类本身，就把比较规则当参数传进去：

```csharp
public static void Main(string[] args)
{
    List<Item> item = new List<Item>();
    item.Sort(Sortp);   // 按 Sortp 的规则排序
}

static int Sortp(Item a, Item b)
{
    if (a.id > b.id)
        return 1;
    else
        return -1;
}
```

### 1.3 Stack　//堆栈

**定义**：后进先出（LIFO）。只允许在一端操作，最后压进去的最先出来。

```csharp
Stack st = new Stack();
```

| 操作 | 方法 | 说明 |
|------|------|------|
| 增 | `Push()` | 压栈 |
| 取 | `Pop()` | 移除并返回栈顶 |
| 查 | `Peek()` | 返回栈顶但不移除 |
| | `Contains(object item)` | 是否存在元素 |
| 改 | `Clear()` | 清空 |

遍历：

```csharp
// 迭代器遍历
foreach (var item in st)
{
}

// 转成 object 数组
object[] array = st.ToArray();
for (int i = 0; i < array.Length; i++)
{
}

// 循环弹栈
while (st.Count > 0)
{
    st.Pop();
}
```

### 1.4 Queue　//队列

**定义**：先进先出（FIFO）。从尾部进，从头部出。

```csharp
Queue qu = new Queue();
```

| 操作 | 方法 | 说明 |
|------|------|------|
| 增 | `Enqueue(object item)` | 添加到队尾 |
| 取 | `Dequeue()` | 移除并返回队头 |
| 查 | `Peek()` | 返回队头但不移除 |
| | `Contains(object item)` | 是否存在元素 |
| 改 | `Clear()` | 清空 |

遍历写法与 Stack 相同：

```csharp
foreach (var item in qu)
{
}

object[] array = qu.ToArray();
for (int i = 0; i < array.Length; i++)
{
}

// 循环出队
while (qu.Count > 0)
{
    qu.Dequeue();
}
```

### 1.5 Hashtable　//哈希表

**定义**：键值对集合。用键的哈希值算出一个位置来存，所以按**键**查找接近 O(1)，但没有顺序。键必须唯一。

```csharp
Hashtable hashtable = new Hashtable();
```

| 操作 | 方法 | 说明 |
|------|------|------|
| 增 | `Add(object key, object value)` | 键重复会抛异常 |
| 删 | `Remove(key)` | 按键删 |
| | `Clear()` | 清空 |
| 查 | `hashtable[key]` | 按键取 |
| | `ContainsKey(object item)` | 是否存在这个键 |
| | `ContainsValue(object item)` | 是否存在这个值 |
| 改 | `hashtable[0] = 1` | 键已存在就是改，不存在则是增 |

遍历：

```csharp
// 所有键
foreach (object item in hashtable.Keys)
{
}

// 所有值
foreach (object item in hashtable.Values)
{
}

// 键值一起
foreach (DictionaryEntry item in hashtable)
{
    Console.WriteLine(item.Key + " " + item.Value);
}

// 迭代器
IDictionaryEnumerator myEnumerator = hashtable.GetEnumerator();
bool flag = myEnumerator.MoveNext();
while (flag)
{
    flag = myEnumerator.MoveNext();
}
```

> `item.Key`、`item.Value` 首字母大写。写成 `key` 编译不过。

## 二、泛型相关

**定义**：把类型本身当作参数。`List<int>` 里的 `int` 就是那个参数，编译时确定，运行时不需要装箱拆箱，也就没有哈希表那类"元素都是 object"的开销。

### 2.1 泛型类和接口

声明时在类名后面挂一个尖括号，里面是占位符（习惯用 `T`）：

```csharp
class TestClass<T>
{
}

TestClass<int> a = new TestClass<int>();
```

### 2.2 泛型方法

泛型方法的类型参数是跟着**方法**走的，调用时由实参推断，和类本身是不是泛型无关。

```csharp
// 普通类中的泛型方法
class Test
{
    public void TestFun<T>(T value)
    {
        Console.WriteLine(value);
    }
}

// 泛型类中的泛型方法：T 来自类，E 来自方法，两者互不影响
class Test<T>
{
    public void TestFun<E>(E value)
    {
        Console.WriteLine(value);
    }
}
```

### 2.3 泛型约束

`where 泛型字母 : 约束`。约束的意思是"别让 T 是什么都行"——加了约束，编译器才允许你在方法体里按这个类型调成员。

| 约束 | 写法 | 含义 | 反例 |
|------|------|------|------|
| 值类型约束 | `where T : struct` | 只能是 `int`、`float`、`bool` 这类 | 不能传 `string` |
| 引用类型约束 | `where T : class` | 只能是类 | 不能传 `int` |
| 无参构造约束 | `where T : new()` | 必须有无参构造函数，这样才能 `new T()` | 抽象类 |
| 类约束 | `where T : Test1` | 必须是 `Test1` 或其子类 | 无关的类 |
| 接口约束 | `where T : Intf` | 必须实现该接口 | 没实现的类 |
| 泛型约束 | `where T : K` | `T` 得是另一个泛型参数的子类 | 写反了方向 |

```csharp
class Test<T> where T : struct
{
    public void TestFun<K>(K value) where K : struct
    {
    }
}

// 多个参数就写多个 where
class Test<T, K> where T : struct where K : class
{
}
```

### 2.4 常用泛型数据结构

#### List　//可变长度的泛型数组

```csharp
List<int> list = new List<int>();
```

增删查改与 ArrayList 完全一致（`Add`、`AddRange`、`Insert`、`Remove`、`RemoveAt`、`Clear`、`Contains`、`IndexOf`），区别只在元素类型固定，不装箱。

遍历：

```csharp
for (int i = 0; i < list.Count; i++)
{
}

foreach (var item in list)
{
}
```

#### Dictionary　//带泛型的哈希表

```csharp
Dictionary<int, string> dictionary = new Dictionary<int, string>();
```

| 操作 | 方法 | 说明 |
|------|------|------|
| 增 | `Add(1, "num1")` | 键重复抛 `ArgumentException` |
| 删 | `Remove(key)` | 按键删 |
| | `Clear()` | 清空 |
| 查 | `ContainsKey(object item)` | 是否存在这个键 |
| | `ContainsValue(object item)` | 是否存在这个值 |
| 改 | `dictionary[1] = "233"` | 键已存在就是改 |

> **Tips**：用下标取值时，如果键找不到会直接抛 `KeyNotFoundException`。不确定就先 `ContainsKey`，或者用 `TryGetValue`。

遍历：

```csharp
foreach (int item in dictionary.Keys)
{
}

foreach (string item in dictionary.Values)
{
}

foreach (KeyValuePair<int, string> item in dictionary)
{
    Console.WriteLine(item.Key + " " + item.Value);
}
```

#### LinkedList　//可变长度的双向链表

**定义**：每个节点存着自己的值、前一个节点和后一个节点。按下标访问要一步步走，但插入和删除不用搬动后面的元素——这是它和 List 最大的区别。

```csharp
LinkedList<int> linkedList = new LinkedList<int>();
```

| 操作 | 方法 | 说明 |
|------|------|------|
| 增 | `AddLast(元素)` | 加在尾部 |
| | `AddFirst(元素)` | 加在头部 |
| | `AddAfter(n, 元素)` | 加在节点 n 之后 |
| | `AddBefore(n, 元素)` | 加在节点 n 之前 |
| 删 | `RemoveFirst()` | 删头 |
| | `RemoveLast()` | 删尾 |
| | `Remove(元素)` | 删指定元素 |
| | `Clear()` | 清空 |
| 查 | `.First` | 头节点 |
| | `.Last` | 尾节点 |
| | `.Find(元素)` | 找不到返回 null |
| | `Contains(元素)` | 是否存在 |
| 改 | `.Value = 114514` | 改节点的值 |

遍历：

```csharp
foreach (int item in linkedList)
{
}

LinkedListNode<int> node = linkedList.First;
while (node != null)
{
    node = node.Next;
}
```

### 2.5 泛型栈和队列

用法和非泛型版本一模一样，只是元素类型固定了：

```csharp
Stack<int> stack = new Stack<int>();
Queue<object> queue = new Queue<object>();
```

> 一点选择上的经验：元素少或者图省事，`List<T>` 基本够用；要频繁按下标读写就 List；要频繁在中间插删、并且已经拿到节点，才轮到 LinkedList；`Dictionary` 只在需要按键查找时用。

## 三、委托事件

### 3.1 委托

**定义**：函数的容器。它把方法本身当成值来存、来传、来调，所以可以先把"要做什么"存起来，等到合适的时候再执行。

```csharp
// 声明：public delegate 返回值类型 名字(参数列表);
public delegate void MyDelegate();
```

```csharp
class Test
{
    delegate void Fn();

    static void Main(string[] args)
    {
        // 调用方式一：new + Invoke
        Fn f = new Fn(Fun);
        f.Invoke();

        // 调用方式二：直接赋值，像调普通方法一样调
        Fn f2 = Fun;
        f2();
    }

    static void Fun()
    {
        Console.WriteLine("22");
    }
}
```

> 注意 C# 的字符串用**双引号**，单引号是 `char`，写 `'22'` 编译不过。

#### 多播委托

一个委托变量可以挂多个方法，调用时按挂载顺序依次执行。

```csharp
Fun ff = Fun;
ff += Fun;   // 增
ff -= Fun;   // 删
```

#### 系统自带的常用委托

自己声明委托有点啰嗦，框架准备了两个常用的：

| 委托 | 签名 | 说明 |
|------|------|------|
| `Action` | 无参、无返回值 | |
| `Action<int, string>` | 有参、无返回值 | 参数最多 16 个 |
| `Func<int>` | 无参、返回 `int` | **最后一个类型参数是返回值** |
| `Func<int, int>` | 传 `int` 返回 `int` | |

```csharp
Action action = Fun;              // 常用无参无返回值委托
Action<int, string> a2 = Fun2;    // 常用有参无返回值委托
Func<int, int> f = Fun3;          // 传入 int 返回 int
```

### 3.2 事件

**定义**：一种特殊的成员，是委托外面包了一层访问控制。它本质上还是委托，只是不许外部随便调用和赋值。

```csharp
public event Action myEvent;
```

限制：

- 在定义它的类**外面**不能调用，也不能用 `=` 赋值；
- 但可以 `+=` / `-=` 增删函数（这正是外部订阅事件的唯一入口）；
- 不能当函数里的局部变量用。

### 3.3 匿名函数

**定义**：没有名字的函数。用在"这个函数只在这里用一次、没必要单独起个名字"的场合，比如给委托或事件赋值、往方法里传委托参数时。

```csharp
delegate (参数列表)
{
}
```

```csharp
// 无参无返回
Action a = delegate ()
{
    Console.WriteLine("114514");
};
a();

// 有参
Action<int, string> a2 = delegate (int i, string s)
{
};

// 有返回
Func<int> a3 = delegate ()
{
    return 0;
};
```

> 赋值语句末尾的 `;` 别丢，`}` 后面要跟一个。

**缺点**：写进委托后没有名字可以引用，也就没法单独把它移除。

### 3.4 Lambda 表达式

**定义**：匿名函数的简写，`=>` 读作"goes to"。左边是参数，右边是函数体。

```csharp
Action a = (参数列表) =>
{
    // 函数体
};
```

```csharp
// 有参
Action<int> a2 = (int value) =>
{
};

// 有返回：Action 没有返回值，要返回值得用 Func
Func<string, int> a3 = (value) =>
{
    return 0;
};
```

### 3.5 闭包

**定义**：内层函数用到了外层的局部变量，外层函数已经执行完了，那个变量还得活着。编译器会把这类变量从栈搬到堆上，让内层函数一直能访问——这个被延长了寿命的变量就是闭包。

```csharp
public class Test
{
    public event Action ac;

    public Test()
    {
        int value = 10;

        ac = () =>
        {
            Console.WriteLine(value);
        };

        value += 10;
    }
}
```

这里 `ac` 打印出来是 **20**，不是 10。闭包捕获的是**变量本身**，不是它当时的值，所以后来那句 `value += 10` 也算数。想固定住当时的值，得在内部另存一份。

## 四、协变逆变

### 4.1 作用

`out` 和 `in` 是加在**泛型参数**上的修饰符，只对委托和接口有意义。

```csharp
// out：用 out 修饰的泛型只能出现在返回值位置
delegate T TestOut<out T>(T v);

// in：用 in 修饰的泛型只能出现在参数位置
delegate void TestIn<in T>(T t);
```

**协变**（`out`）让子类可以当父类用；**逆变**（`in`）让父类可以当子类用。加这两个关键字，是为了让本来就安全的赋值能通过编译。

## 五、多线程

**进程**：一个程序在系统中可分配资源和调度的基本单位。

**线程**：包含在进程中，是系统调度的最小单位。

**多线程**：通过代码开启新的线程，让多件事看起来在同时推进。

### 5.1 线程类 Thread

需要 `using System.Threading;`

```csharp
Thread t = new Thread(Fun);      // 新建一个线程，执行 Fun
static void Fun()
{
}

t.Start();                       // 开启这个线程
t.IsBackground = true;           // 设为后台线程
Thread.Sleep(毫秒);              // 当前线程休眠
lock (引用类型对象)              // 线程加锁，防止同时改同一份数据
{
}
```

> 注意大小写：`t.Start()` 不是 `t.start`。

**前后台线程的区别**（和字面意思不太一样，容易记反）：

| 类型 | 是否阻止进程退出 | 默认情况 |
|------|------------------|----------|
| 前台线程 | 会，进程要等它跑完 | `Main` 主线程、`new Thread()` 新建的线程 |
| 后台线程 | 不会，前台线程全结束后它被直接掐掉 | 线程池线程、`Task` |

所以后台线程适合做"进程关了就该停"的辅助工作；要保证跑到最后的任务别设成后台。

## 六、预处理器指令

**定义**：给编译器看的指令，在真正编译之前先处理，不生成 IL 代码。常用于按环境/平台保留不同代码——比如调试输出只在 Debug 构建里编译。

```csharp
#define test          // 定义符号 test

#if test
    Console.WriteLine("sdsd");   // 有符号 test 才编译这一句
#endif
```

> `#define` 必须写在文件最顶端，所有 `using` 之前。

常用的还有 `#if / #elif / #else / #endif`、`#region / #endregion`（折叠代码，不影响编译）、`#warning`、`#error`。

## 七、反射和特性

### 7.1 反射

**程序集**：编译器编译后的中间产物，在 Windows 上多为 `.dll` 或 `.exe`。

**元数据**：描述数据的数据——有哪些类、类里有哪些字段、方法和构造函数。

**反射**：在程序运行时查看其他程序集或自身的元数据。简单说就是"程序在跑的时候回头看自己长什么样"。

**作用**：运行时才决定要创建哪个对象、调哪个方法，而不是编译时就写死。

#### 7.1.1 Type 类

反射的基础。拿到 `Type`，就等于拿到了这个类的所有描述信息。

```csharp
int a = 1;

Type type = a.GetType();                        // 从对象上取
Type type2 = typeof(int);                       // 从类名取
Type type3 = Type.GetType("System.Int32");      // 用带命名空间的类型名字符串取
```

**获取类中所有公共成员**

```csharp
using System.Reflection;

Type t = typeof(Test);
MemberInfo[] infos = t.GetMembers();
```

**获取构造函数并调用**

```csharp
Type t = typeof(Test);

// 所有构造函数，ConstructorInfo 用来存构造函数信息
ConstructorInfo[] ctors = t.GetConstructors();

// 只要无参的那个（注意是 GetConstructor，单数）
ConstructorInfo ctor = t.GetConstructor(new Type[0]);
ctor.Invoke(null);

// 有参的：先用 Type 数组说明参数类型，Invoke 时再传对应类型的实参
ConstructorInfo ctor2 = t.GetConstructor(new Type[] { typeof(int), typeof(string) });
Test o = ctor2.Invoke(new object[] { 99, "abc" }) as Test;
```

**获取成员变量**

```csharp
// 所有成员变量
FieldInfo[] fieldInfos = t.GetFields();

// 指定名称的公共成员变量
FieldInfo info = t.GetField("j");

// 通过反射读和写对象的值
Test test = new Test();
test.j = 99;

Console.WriteLine(info.GetValue(test));   // 1. 取值：99
info.SetValue(test, 100);                 // 2. 赋值
Console.WriteLine(info.GetValue(test));   // 100
```

**获取成员方法**

```csharp
Type strType = typeof(string);
MethodInfo[] methods = strType.GetMethods();   // 获取全部方法

// 有重载时，用 Type 数组指定参数类型才能唯一定位
Type type = typeof(Calculator);
Calculator calc = new Calculator();

MethodInfo addMethod = type.GetMethod("Add", new Type[] { typeof(int), typeof(int) });
object result = addMethod.Invoke(calc, new object[] { 3, 5 });
Console.WriteLine(result);   // 8
```

#### 7.1.2 Assembly 类

用来加载其他程序集。

```csharp
// 1. 加载指定程序集
Assembly assembly = Assembly.LoadFrom(@"c:\user\...");
Type[] types = assembly.GetTypes();

// 2. 再取出程序集里的某个类
Type icon = assembly.GetType("命名空间.类名");
MemberInfo[] member = icon.GetMembers();
```

#### 7.1.3 Activator 类

用于快速实例化对象，省掉手动找构造函数的步骤。

```csharp
Type testType = typeof(Test);

// 无参构造
Test obj = Activator.CreateInstance(testType) as Test;

// 有参构造：参数依次跟在类型后面
obj = Activator.CreateInstance(testType, 99) as Test;
```

### 7.2 特性

**定义**：附加在类、方法、字段上的额外信息，本身不影响程序运行，但可以被反射读出来。名字按约定以 `Attribute` 结尾，用的时候可以省略后缀。

#### 7.2.1 自定义特性

写一个继承 `Attribute` 的类就行：

```csharp
class MyCustomAttribute : Attribute
{
    public string info;

    public MyCustomAttribute(string info)
    {
        this.info = info;
    }
}
```

#### 7.2.2 使用

```csharp
[MyCustom("备注信息")]
class MyClass
{
}

// 读取
MyClass mc = new MyClass();
Type t = mc.GetType();

if (t.IsDefined(typeof(MyCustomAttribute), false))   // (特性类型, 是否搜索父类)
{
}
```

#### 7.2.3 限制特性的使用范围

```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Struct,
                AllowMultiple = true,
                Inherited = true)]
class MyCustomAttribute : Attribute
{
}
```

| 参数 | 含义 |
|------|------|
| `AttributeTargets` | 特性能够附着在哪种数据上（类、方法、字段……可多选） |
| `AllowMultiple` | 同一个目标上是否允许多个该特性实例 |
| `Inherited` | 特性能否被派生类和重写成员继承 |

#### 7.2.4 系统自带特性

**过时特性**

```csharp
[Obsolete("方法已过时，请使用新方法", false)]   // false-使用时警告，true-使用时报错
```

**调用者信息特性**

```csharp
using System.Runtime.CompilerServices;

void Log(string msg,
         [CallerFilePath] string file = "",      // 哪个文件调用
         [CallerLineNumber] int line = 0,        // 哪一行调用
         [CallerMemberName] string member = "")  // 哪个函数调用
{
}
```

这几个参数不用自己传，编译器会在调用处自动填上，常用于打日志。

**条件编译特性**

`[Conditional("符号名")]` 加在**无返回值**的方法上：没定义这个符号时，编译器会把所有对该方法的调用整句删掉，比在调用处写一堆 `#if` 干净。

```csharp
[Conditional("DEBUG")]
void DebugLog(string msg)
{
}
```

**调用外部 dll 的函数特性**

用 `DllImport` 声明一个外部方法，运行时按名字去 dll 里找：

```csharp
using System.Runtime.InteropServices;

[DllImport("user32.dll")]
static extern int MessageBox(IntPtr hWnd, string text, string caption, uint type);
```

**待补充**：反射获取特性实例、`DllImport` 的字符集与调用约定参数。

## 八、迭代器（光标）

**定义**：一种统一访问聚合对象里各个元素的方式。它把"遍历"这件事抽出来——集合只负责提供数据，光标负责记住走到哪了。`foreach` 能用，靠的就是它。

### 8.1 标准迭代器的实现方式

需要继承两个接口：`IEnumerable`（表示"我能被遍历"）和 `IEnumerator`（表示"我是那个光标"）。

```csharp
class CustomList : IEnumerable, IEnumerator
{
    private int[] list;
    private int position = -1;   // 从 -1 开始的光标

    public CustomList()
    {
        list = new int[] { 1, 2, 3, 4, 5, 6, 7, 8 };
    }

    public IEnumerator GetEnumerator()
    {
        return this;
    }

    public object Current
    {
        get { return list[position]; }
    }

    public bool MoveNext()
    {
        ++position;                     // 移动光标
        return position < list.Length;  // 是否溢出
    }

    public void Reset()
    {
        position = -1;
    }
}

// foreach (int item in list)
```

**foreach 的本质**：

1. 拿到 `in` 后面对象的 `IEnumerator`，通过 `GetEnumerator` 方法获取；
2. 调它的 `MoveNext`；
3. 只要 `MoveNext` 返回 `true`，就读一次 `Current` 并赋值给 `item`。

### 8.2 用 yield return 语法糖实现迭代器

手写上面那一套太啰嗦。`yield return` 让编译器自动生成状态机，你只要把元素挨个"吐"出来。

```csharp
class CustomList2 : IEnumerable
{
    private int[] list;

    public CustomList2()
    {
        list = new int[] { 1, 2, 3, 4, 5, 6, 7, 8 };
    }

    public IEnumerator GetEnumerator()
    {
        for (int i = 0; i < list.Length; i++)
        {
            yield return list[i];   // 暂停在这里，下次 MoveNext 从下一句继续
        }
    }
}
```

> 关键点：`yield return` 不是一次全部返回，而是**每次 `MoveNext` 才执行到下一个 `yield`**，中途整个方法的状态被保存着。

### 8.3 用 yield return 为泛型类实现迭代器

```csharp
class CustomList<T> : IEnumerable
{
    private T[] array;

    public IEnumerator GetEnumerator()
    {
        for (int i = 0; i < array.Length; i++)
        {
            yield return array[i];
        }
    }
}
```

> 更完整的写法是继承泛型接口 `IEnumerable<T>`，它的 `GetEnumerator` 返回 `IEnumerator<T>`，`Current` 就是 `T` 而不是 `object`，又省掉一次装箱。

## 九、特殊语法

### 9.1 var 隐式类型

**定义**：让编译器根据右边的值推断类型。它**不是**"任意类型"，编译后类型是确定的，只是写的时候省了。

```csharp
var i = 123;   // 编译后就是 int
```

限制：必须初始化，不能作为类的成员，只能用于局部变量。

### 9.2 设置对象初始值

```csharp
Person p = new Person { sex = true, Age = 17 };   // 先执行构造函数，再执行大括号
```

### 9.3 设置集合初始值

```csharp
int[] array = new int[] { 1, 2, 3, 4 };
List<int> listInt = new List<int>() { 1, 2, 3, 4 };
```

### 9.4 匿名类型

**定义**：用 `new { }` 临时拼出来的类型，成员只读，只能在当前方法里用。

```csharp
var v = new { age = 18, name = "micha" };   // 只能存成员变量
```

### 9.5 可空类型

**定义**：给值类型加一个 `null` 的可能。`int?` 是 `Nullable<int>` 的简写。

```csharp
int? c = null;

// 判断
if (c.HasValue)
{
}

// 安全取值
int? value = null;
value.GetValueOrDefault();       // 返回该类型的默认值
value.GetValueOrDefault(100);    // 指定一个默认值
```

### 9.6 空合并操作符

**定义**：`??`——左边是 `null` 就返回右边，否则返回左边。

```csharp
int? a = null;
int b = 3;
int result = a ?? b;   // 返回 3
```

### 9.7 内插字符串

**定义**：字符串前面加 `$`，大括号里直接写变量或表达式。

```csharp
Console.WriteLine($"我的名字{name}");
```

> `$` 要写在引号**外面**，写成 `cw.($"...")` 编译不过。

### 9.8 单句逻辑简略写法

```csharp
// 单句可以不加花括号
if (true)
    Console.WriteLine("");

for (int i = 0; i < 3; i++)
    Console.WriteLine("");

// Lambda：参数 => 表达式
Func<int> a = () => 21;

// 方法体用 => 简化
int Add(int x, int y) => x + y;
```

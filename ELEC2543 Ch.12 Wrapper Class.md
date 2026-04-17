#Y2S2 #ELEC2543
# Chapter 12. 包装类

`Pre: ` [[ELEC2543 Ch.11 Variables and Object Comparison]]
`Post:` [[ELEC2543 Ch.13 Interface]]

> [!abstract]
> 包装类 (_Wrapper Class_) 是 Java 类型系统中连接基本类型与对象体系的桥梁机制，属于 `java.lang` 包的核心组成部分。
>
> Java 的基本类型（如 `int`、`double`）不是对象，无法直接用于泛型容器、需要对象参数的方法调用等场景。包装类为每种基本类型提供对应的对象形式，使基本类型值得以融入面向对象的类型体系。Java 5 引入的自动装箱 (_Autoboxing_) 与自动拆箱 (_Unboxing_) 机制进一步消除了手动转换的繁琐，但也引入了与 `==` 比较相关的微妙陷阱——尤其是整数缓存范围（−128-128−128 到 127127127）导致的引用共享行为，与 Python 的小整数缓存机制在表现上相似，但触发条件更为复杂。
>
> 本章涵盖包装类的对应关系与创建方式、自动装箱与拆箱的触发机制、包装类中的静态工具方法与常量，以及 `==` 运算符作用于 `Integer` 对象时因缓存机制产生的行为差异。
```table-of-contents
maxLevel: 3
```
## 12.1 包装类概述

包装类 (*Wrapper Class*) 是 `java.lang` 包中为每种基本类型提供的对应**对象类型**。`java.lang` 包无需显式导入，其中的类在所有 Java 程序中均可直接使用。

八种基本类型与其对应包装类的映射关系如下：

| 基本类型 | 包装类 |
|---------|--------|
| `byte` | `Byte` |
| `short` | `Short` |
| `int` | `Integer` |
| `long` | `Long` |
| `float` | `Float` |
| `double` | `Double` |
| `char` | `Character` |
| `boolean` | `Boolean` |

包装类的核心用途是在需要对象而非基本类型的场合中使用基本类型的值：

- 某些方法要求参数为对象类型，基本类型值无法直接传入。
- 泛型容器（如 `ArrayList<Integer>`）只能存储对象，不能存储基本类型。
- 需要调用与类型相关的工具方法（如字符串转整数）时，包装类提供了对应的静态方法。

> [!note]
> 包装类名称与基本类型名称的对应规律：大多数包装类名为基本类型名首字母大写，但有两个例外—— `int` 对应 `Integer`，`char` 对应 `Character`。

> [!note]
> Java 的类型系统严格区分基本类型 (*Primitive Type*) 与对象类型 (*Object Type*)。这一区分在语法层面导致了基本类型无法直接应用于以下两类典型场景：
>
> **1. 需要对象引用作为形式参数的方法调用**
>
> 某些 API 方法（尤其是接受通用 Object 类型参数的方法）要求传入的数据必须为对象。基本类型因不属于对象体系，无法通过编译期的类型检查。
>
> ```java
> // 假设某方法要求参数为 Object 类型
> void processObject(Object obj) { ... }
>
> int num = 42;
> processObject(num);  // 编译错误：int 无法转换为 java.lang.Object
> ```
>
> 通过 `Integer` 包装类，`num` 经自动装箱后可作为对象引用传入：
>
> ```java
> processObject(num);  // 经 autoboxing 后等价于 processObject(Integer.valueOf(42))
> ```
>
> **2. 泛型容器与集合框架的类型约束**
>
> Java 的泛型机制 (*Generics*) 要求类型参数必须为引用类型。以 `ArrayList` 为例（参见 Ch.8），其声明 `ArrayList<E>` 中的 `E` 不能替换为基本类型：
>
> ```java
> ArrayList<int> list;     // 编译错误：Type argument cannot be of primitive type
> ArrayList<Integer> list; // 合法：使用包装类作为类型参数
> ```
>
> 向 `ArrayList<Integer>` 添加基本类型 `int` 时，编译器自动插入装箱指令；取出时自动拆箱。若无包装类，基本类型值无法存储于此类容器中。
>
> **3. 需要表示"无值"状态的场景**
>
> 基本类型始终持有默认值（如 `int` 默认为 `0`），无法表达"值缺失"的语义。包装类对象引用可为 `null`，适用于数据库查询结果、可选配置等可能为空的场景：
>
> ```java
> Integer age = null;  // 表示年龄信息尚未录入
> ```

## 12.2 包装类对象的创建

包装类对象有两种创建方式。

第一种是使用 `new` 构造器，将基本类型值作为参数传入：

```java
Integer age = new Integer(40);
```

第二种是使用静态工厂方法 `valueOf()`：

```java
Integer age = Integer.valueOf(40);
```

两种方式均创建一个封装了 `int 40` 的 `Integer` 对象，`age` 存储指向该对象的引用。

包装对象是不可变的 (*Immutable*)，与 `String` 对象相同：一旦创建，其内部封装的值不可更改。对包装对象执行算术运算（如 `obj++`）时，实际上是先拆箱取出值、完成运算，再将结果装箱为一个新对象，原对象不被修改。

> [!warning]
> `new Integer(int)` 构造器自 Java 9 起已被标记为废弃 (*Deprecated*)，推荐使用 `Integer.valueOf(int)`。
>
> 两者的关键区别在于：`new Integer(40)` 每次都在堆上创建一个全新对象；而 `Integer.valueOf(40)` 在值处于缓存范围（$-128$ 到 $127$）内时会返回缓存中已有的对象，不创建新对象。
>
> > [!quote]
> > 这一区别直接影响 `==` 比较的结果，将在 12.5 节中详细讨论。

## 12.3 自动装箱与拆箱

自动装箱 (*Autoboxing*) 是指将基本类型值赋给对应包装类型变量时，编译器自动完成从基本类型到包装对象的转换，无需手动调用 `valueOf()` 或 `new`：

```java
Integer obj;
int num = 42;
obj = num;  // 自动装箱：等价于 obj = Integer.valueOf(42)
```

自动拆箱 (*Unboxing*) 是其逆过程——将包装对象赋给基本类型变量时，编译器自动提取对象内部封装的值：

```java
Integer obj = Integer.valueOf(40);
int num = obj;  // 自动拆箱：等价于 num = obj.intValue()
```

装箱与拆箱也可在表达式中隐式触发。包装对象参与算术运算时，编译器先对其拆箱，完成运算后若结果需赋给包装类型变量，再重新装箱：

```java
Integer obj = 10;
obj = obj + 5;  // 先拆箱：10 + 5 = 15，再装箱：obj = Integer.valueOf(15)
```

> [!note]
> 自动拆箱时若包装对象为 `null`，运行时将抛出 `NullPointerException`。这是自动拆箱最常见的陷阱：
> > [!bug]
> > ```java
> > Integer obj = null;
> > int num = obj;  // 运行时抛出 NullPointerException
> > ```

> [!example]- 示例：`BoxingDemo.java`
> 演示自动装箱与拆箱在赋值、算术运算及方法参数传递中的行为，以及 `==` 比较 `Integer` 对象时因缓存机制产生的差异。
>
> ```java
> public class BoxingDemo {
>
>     // 接收 Integer 参数，对其执行自增操作
>     public static void incrementInt(Integer x) {
>         System.out.println("before x++, x = " + x);
>         x++;  // 拆箱 → 加一 → 装箱为新对象，原引用 x 指向新对象
>         System.out.println("after x++, x = " + x);
>     }
>
>     public static void main(String[] args) {
>         Integer obj1, obj2;
>
>         // 情形一：obj1 = obj2 = 1000（超出缓存范围）
>         obj1 = obj2 = 1000;
>         System.out.println("obj1 = obj2 = 1000;");
>         if (obj1 == obj2)
>             System.out.println("The two objects are equal in memory.");
>         else
>             System.out.println("The two objects are NOT equal in memory.");
>
>         // 情形二：obj1 自动装箱，obj2 用 new 创建
>         obj1 = 100;
>         obj2 = new Integer(100);
>         System.out.println("\nobj1 = 100; obj2 = new Integer(100)");
>         if (obj1 == obj2)
>             System.out.println("The two objects are equal in memory.");
>         else
>             System.out.println("The two objects are NOT equal in memory.");
>
>         // 情形三：两者均用 new 创建
>         obj1 = new Integer(100);
>         obj2 = new Integer(100);
>         System.out.println("\nobj1 = new Integer(100); obj2 = new Integer(100);");
>         if (obj1 == obj2)
>             System.out.println("The two objects are equal in memory.");
>         else
>             System.out.println("The two objects are NOT equal in memory.");
>
>         // 情形四：自增后比较（缓存范围内）
>         obj1 = 100;
>         obj2 = 101;
>         obj1++;
>         System.out.println("obj1 = 100; obj2 = 101; obj1++;");
>         if (obj1 == obj2)
>             System.out.println("obj1 == obj2");
>         else
>             System.out.println("obj1 =/= obj2");
>
>         // 情形五：obj2 用 new 创建，obj1 自增后与其比较
>         obj1 = 100;
>         obj2 = new Integer(101);
>         obj1++;
>         System.out.println("\nobj1 = 100; obj2 = new Integer(101); obj1++;");
>         if (obj1 == obj2)
>             System.out.println("obj1 == obj2");
>         else
>             System.out.println("obj1 =/= obj2");
>
>         // 情形六：obj1 用 new 创建，自增后与 obj2 比较
>         obj1 = new Integer(100);
>         obj2 = 101;
>         obj1++;
>         System.out.println("\nobj1 = new Integer(100); obj2 = 101; obj1++;");
>         if (obj1 == obj2)
>             System.out.println("obj1 == obj2");
>         else
>             System.out.println("obj1 =/= obj2");
>
>         // 方法参数传递：Integer 按引用副本传递，方法内自增不影响原变量
>         System.out.println("before x++, obj1 = " + obj1);
>         incrementInt(obj1);
>         System.out.println("after x++, obj1 = " + obj1);
>     }
> }
> ```
>
> 输出：
> ```
> obj1 = obj2 = 1000;
> The two objects are equal in memory.
>
> obj1 = 100; obj2 = new Integer(100)
> The two objects are NOT equal in memory.
>
> obj1 = new Integer(100); obj2 = new Integer(100);
> The two objects are NOT equal in memory.
>
> obj1 = 100; obj2 = 101; obj1++;
> obj1 == obj2
>
> obj1 = 100; obj2 = new Integer(101); obj1++;
> obj1 =/= obj2
>
> obj1 = new Integer(100); obj2 = 101; obj1++;
> obj1 =/= obj2
>
> before x++, obj1 = 101
> before x++, x = 101
> after x++, x = 102
> after x++, obj1 = 101
> ```
>
> **情形一：链式赋值 `obj1 = obj2 = 1000` **
> ```java
> obj1 = obj2 = 1000;
> ```
> - 链式赋值从右向左执行：首先 `obj2 = 1000` 触发自动装箱，调用 `Integer.valueOf(1000)`。
> - 由于 `1000` 超出缓存范围，创建一个新 `Integer` 对象。
> - 随后 `obj1 = obj2` 将 `obj2` 的引用直接赋给 `obj1`，两者指向**同一个对象**，不再创建新对象。
> - `obj1 == obj2` 比较引用，结果为 `true`。
>
> **情形二：`obj1` 自动装箱，`obj2` 用 `new` 创建**
> ```java
> obj1 = 100;               // 自动装箱：Integer.valueOf(100)，返回缓存对象
> obj2 = new Integer(100);  // 强制创建新对象，不使用缓存
> ```
> - `obj1` 指向缓存中的 `Integer(100)` 对象；`obj2` 指向堆上新创建的对象。
> - 两者地址不同，`obj1 == obj2` 为 `false`。
>
> **情形三：两者均用 `new` 创建**
> ```java
> obj1 = new Integer(100);
> obj2 = new Integer(100);
> ```
> - `new` 每次都在堆上创建独立对象，两者地址不同。
> - `obj1 == obj2` 为 `false`。
>
> **情形四：自增后比较（缓存范围内）**
> ```java
> obj1 = 100;   // 指向缓存 Integer(100)
> obj2 = 101;   // 指向缓存 Integer(101)
> obj1++;       // 拆箱 → 100+1=101 → Integer.valueOf(101) → 返回缓存 Integer(101)
> ```
> - `obj1++` 的结果通过 `Integer.valueOf(101)` 装箱，`101` 在缓存范围内，返回与 `obj2` 相同的缓存对象。
> - `obj1 == obj2` 为 `true`。
>
> **情形五：`obj2` 用 `new` 创建**
> ```java
> obj1 = 100;
> obj2 = new Integer(101);  // 强制创建新对象
> obj1++;                   // Integer.valueOf(101)，返回缓存对象
> ```
> - `obj1` 自增后指向缓存中的 `Integer(101)`；`obj2` 指向堆上独立创建的 `Integer(101)`。
> - 两者地址不同，`obj1 == obj2` 为 `false`。
>
> **情形六：`obj1` 用 `new` 创建后自增**
> ```java
> obj1 = new Integer(100);  // 堆上新对象
> obj2 = 101;               // 缓存对象
> obj1++;                   // 拆箱 → 101 → Integer.valueOf(101) → 缓存对象
> ```
> - `obj1++` 先对 `new Integer(100)` 拆箱得 `100`，加一得 `101`，再通过 `Integer.valueOf(101)` 装箱，返回缓存中的 `Integer(101)`。
> - `obj2 = 101` 同样指向缓存中的 `Integer(101)`，两者为同一对象。
> - `obj1 == obj2` 为 `true`。
>
> > [!note]
> > 情形五与情形六的对比揭示了一个关键规律：`==` 的结果取决于两个引用是否指向同一对象，而非值是否相等。只要任意一方通过 `new` 创建（绕过缓存），即使值相同，`==` 也为 `false`。自增操作（`obj++`）内部通过 `Integer.valueOf()` 装箱，因此会利用缓存；而 `new Integer()` 始终创建新对象。
>
> **方法参数传递：`incrementInt(obj1)` **
> ```java
> public static void incrementInt(Integer x) {
>     x++;  // 拆箱 → 加一 → 装箱为新对象，x 指向新对象
> }
> ```
> - `Integer` 是对象类型，传入方法时传递的是引用的副本。
> - 方法内 `x++` 等价于 `x = Integer.valueOf(x.intValue() + 1)`，使形式参数 `x` 指向一个新的 `Integer` 对象。
> - 这只改变了形式参数副本的指向，不影响调用处的 `obj1`，与 9.3 节中"修改引用指向不影响原变量"的规则一致。
> - 方法返回后 `obj1` 仍为 `101`。

## 12.4 包装类中的静态方法与常量

包装类除封装基本类型值外，还提供了一组静态工具方法与常量，用于处理与该类型相关的常见操作。

各包装类均包含将字符串解析为对应基本类型值的静态方法，命名规律为 `parseXxx(String)`：

```java
int num    = Integer.parseInt("42");             // 将字符串 "42" 转换为 int 42
double d   = Double.parseDouble("3.14");         // 将字符串 "3.14" 转换为 double 3.14
boolean b  = Boolean.parseBoolean("true");       // 将字符串 "true" 转换为 boolean true
```

- `parseInt` 等方法在字符串无法被解析为对应类型时（如 `Integer.parseInt("abc")`）会抛出 `NumberFormatException`。
- 这类方法是处理命令行参数或用户输入时将字符串转为数值的标准做法。

`valueOf()` 静态方法既可接收基本类型值，也可接收字符串，返回对应的包装对象：

```java
Integer obj1 = Integer.valueOf(42);      // 从 int 创建
Integer obj2 = Integer.valueOf("42");    // 从 String 创建，等价于 Integer.parseInt("42") 并装箱
```

各包装类还定义了与类型范围相关的常量，以 `Integer` 为例：

```java
System.out.println(Integer.MIN_VALUE);  // -2147483648，即 -2^31
System.out.println(Integer.MAX_VALUE);  //  2147483647，即  2^31 - 1
```

其他包装类的对应常量：

| 包装类 | 最小值常量 | 最大值常量 |
|--------|-----------|-----------|
| `Byte` | `Byte.MIN_VALUE`（$-128$） | `Byte.MAX_VALUE`（$127$） |
| `Short` | `Short.MIN_VALUE`（$-32768$） | `Short.MAX_VALUE`（$32767$） |
| `Integer` | `Integer.MIN_VALUE`（$-2^{31}$） | `Integer.MAX_VALUE`（$2^{31}-1$） |
| `Long` | `Long.MIN_VALUE`（$-2^{63}$） | `Long.MAX_VALUE`（$2^{63}-1$） |
| `Float` | `Float.MIN_VALUE`（最小正值） | `Float.MAX_VALUE` |
| `Double` | `Double.MIN_VALUE`（最小正值） | `Double.MAX_VALUE` |

> [!note]
> `Float.MIN_VALUE` 与 `Double.MIN_VALUE` 表示的是该类型可表示的**最小正数**（约为 $1.4 \times 10^{-45}$ 和 $4.9 \times 10^{-324}$），而非负数方向的最小值。若需要负方向的极值，应使用 `-Float.MAX_VALUE` 或 `-Double.MAX_VALUE`。这与整数类型的 `MIN_VALUE` 语义不同，是常见的混淆点。

## 12.5 `==` 运算符与整数缓存

`==` 作用于 `Integer` 等包装类对象时，比较的是两个引用是否指向内存中的**同一个对象**，而非封装值是否相等。要比较两个包装对象的值，应使用 `equals` 方法。

Java 的 `Integer.valueOf()` 方法对范围在 $-128$ 到 $127$ 之间的整数实施缓存 (*Integer Cache*)：该范围内的每个值对应一个预先创建并存储在缓存池中的 `Integer` 对象，每次调用 `Integer.valueOf(n)`（包括自动装箱）时，若 $n$ 在此范围内，直接返回缓存中已有的对象，不创建新对象。超出此范围时，每次调用均在堆上创建新对象。

`new Integer(n)` 始终在堆上创建新对象，完全绕过缓存机制，无论 $n$ 是否在缓存范围内。

这一机制导致 `==` 对 `Integer` 对象的比较结果依赖于对象的创建方式与值的范围：

```java
Integer a = 100;           // 自动装箱，使用缓存
Integer b = 100;           // 自动装箱，使用缓存，与 a 指向同一对象
System.out.println(a == b); // true

Integer c = 1000;          // 自动装箱，超出缓存范围，创建新对象
Integer d = 1000;          // 自动装箱，超出缓存范围，创建另一个新对象
System.out.println(c == d); // false

Integer e = new Integer(100); // 强制创建新对象，绕过缓存
Integer f = 100;              // 使用缓存
System.out.println(e == f);   // false
```

> [!example]- 例题：In-Class Exercise (1) ——装箱/拆箱分析与新对象创建判断
>
> 分析以下代码片段，回答三个问题：各变量的最终值、每条赋值语句中发生的装箱/拆箱操作，以及每条赋值语句是否创建了新的 `Integer` 对象。
>
> ```java
> // 课件原题代码片段
> Integer intObj1, intObj2;   // 第 1 行：声明两个 Integer 引用变量
> int i1, i2;                 // 第 2 行：声明两个 int 基本类型变量
> i1 = 1000;                  // 第 3 行
> intObj1 = i1;               // 第 4 行
> intObj2 = intObj1;          // 第 5 行
> i2 = intObj2 + 1;           // 第 6 行
> intObj2 = intObj2 + i2;     // 第 7 行
> ```
>
> **最终值**
>
> | 变量 | 最终值 |
> |------|--------|
> | `i1` | `1000` |
> | `i2` | `1001` |
> | `intObj1` | `1000` |
> | `intObj2` | `2001` |
>
> **逐行分析**
>
> 第 3 行 `i1 = 1000`：
> ```java
> i1 = 1000;
> ```
> - 基本类型赋值，无装箱/拆箱，不创建对象。
>
> 第 4 行 `intObj1 = i1`：
> ```java
> intObj1 = i1;  // 自动装箱：Integer.valueOf(1000)
> ```
> - `int` 赋给 `Integer`，触发**自动装箱**。
> - `1000` 超出缓存范围（$-128$ 到 $127$），在堆上**创建新 `Integer` 对象**，`intObj1` 指向该对象。
>
> 第 5 行 `intObj2 = intObj1`：
> ```java
> intObj2 = intObj1;  // 引用赋值，无装箱/拆箱
> ```
> - `Integer` 引用赋给 `Integer` 引用，直接复制引用，`intObj2` 与 `intObj1` 指向同一对象。
> - 无装箱/拆箱，**不创建新对象**。
>
> 第 6 行 `i2 = intObj2 + 1`：
> ```java
> i2 = intObj2 + 1;  // 自动拆箱：intObj2.intValue() + 1
> ```
> - `intObj2` 参与算术运算，触发**自动拆箱**，提取值 `1000`，计算 `1000 + 1 = 1001`。
> - 结果赋给基本类型 `i2`，无装箱，**不创建新对象**。
>
> 第 7 行 `intObj2 = intObj2 + i2`：
> ```java
> intObj2 = intObj2 + i2;
> // 等价于：intObj2 = Integer.valueOf(intObj2.intValue() + i2)
> ```
> - `intObj2` 参与算术运算，触发**自动拆箱**，提取值 `1000`；`i2` 为基本类型 `1001`。
> - 计算 `1000 + 1001 = 2001`，结果赋给 `Integer` 变量，触发**自动装箱**。
> - `2001` 超出缓存范围，在堆上**创建新 `Integer` 对象**，`intObj2` 指向该新对象。
> - 原来 `intObj1` 与 `intObj2` 共同指向的 `Integer(1000)` 对象，此后仍由 `intObj1` 持有引用，不成为垃圾。
>
> **汇总**
>
> | 行号 | 装箱/拆箱 | 是否创建新对象 |
> |------|----------|--------------|
> | 第 3 行 | 无 | 否 |
> | 第 4 行 | 装箱 | 是（`1000` 超出缓存范围） |
> | 第 5 行 | 无 | 否 |
> | 第 6 行 | 拆箱 | 否 |
> | 第 7 行 | 拆箱 + 装箱 | 是（`2001` 超出缓存范围） |

> [!example]- 例题：In-Class Exercise (2) ——六组 `==` 比较输出分析
>
> 分析 `TestInteger_new.java` 中六组代码片段，给出每组 `System.out.println(obj1 == obj2)` 的输出。`obj1` 与 `obj2` 均为 `Integer` 类型。
>
> ```java
> // TestInteger_new.java（课件原题）
> Integer obj1;
> Integer obj2;
>
> // Exercise 1
> System.out.print("Exercise 1: ");
> obj1 = 100;
> obj2 = 101;
> obj1++;
> System.out.println(obj1 == obj2);
>
> // Exercise 2
> System.out.print("Exercise 2: ");
> obj1 = 1000;
> obj2 = 1001;
> obj1++;
> System.out.println(obj1 == obj2);
>
> // Exercise 3
> System.out.print("Exercise 3: ");
> obj1 = obj2 = 1000;
> System.out.println(obj1 == obj2);
>
> // Exercise 4
> System.out.print("Exercise 4: ");
> obj1 = 100;
> obj2 = new Integer(100);
> System.out.println(obj1 == obj2);
>
> // Exercise 5
> System.out.print("Exercise 5: ");
> obj1 = new Integer(100);
> obj2 = new Integer(100);
> System.out.println(obj1 == obj2);
>
> // Exercise 6
> System.out.print("Exercise 6: ");
> obj1 = new Integer(100);
> obj2 = 101;
> obj1++;
> System.out.println(obj1 == obj2);
> ```
>
> 输出：
> ```
> Exercise 1: true
> Exercise 2: false
> Exercise 3: true
> Exercise 4: false
> Exercise 5: false
> Exercise 6: true
> ```
>
> **Exercise 1**
> ```java
> obj1 = 100;   // 自动装箱，Integer.valueOf(100)，缓存对象 A
> obj2 = 101;   // 自动装箱，Integer.valueOf(101)，缓存对象 B
> obj1++;       // 拆箱 → 101 → Integer.valueOf(101) → 缓存对象 B
> ```
> - `obj1++` 后 `obj1` 指向缓存中的 `Integer(101)`，与 `obj2` 指向同一缓存对象。
> - `obj1 == obj2` 为 `true`。
>
> **Exercise 2**
> ```java
> obj1 = 1000;  // 自动装箱，超出缓存范围，创建新对象 X
> obj2 = 1001;  // 自动装箱，超出缓存范围，创建新对象 Y
> obj1++;       // 拆箱 → 1001 → Integer.valueOf(1001) → 超出缓存，创建新对象 Z
> ```
> - `obj1` 自增后指向新对象 Z，`obj2` 指向新对象 Y，两者地址不同。
> - `obj1 == obj2` 为 `false`。
>
> **Exercise 3**
> ```java
> obj1 = obj2 = 1000;
> // 等价于：obj2 = Integer.valueOf(1000)（创建新对象），obj1 = obj2（引用赋值）
> ```
> - 链式赋值：`obj2` 先装箱得到新对象，`obj1` 直接复制 `obj2` 的引用，两者指向同一对象。
> - `obj1 == obj2` 为 `true`。
>
> **Exercise 4**
> ```java
> obj1 = 100;              // Integer.valueOf(100)，缓存对象
> obj2 = new Integer(100); // 强制创建新对象，绕过缓存
> ```
> - `obj1` 指向缓存对象，`obj2` 指向堆上独立创建的对象，地址不同。
> - `obj1 == obj2` 为 `false`。
>
> **Exercise 5**
> ```java
> obj1 = new Integer(100); // 新对象 P
> obj2 = new Integer(100); // 新对象 Q
> ```
> - 两次 `new` 各自在堆上创建独立对象，地址不同。
> - `obj1 == obj2` 为 `false`。
>
> **Exercise 6**
> ```java
> obj1 = new Integer(100); // 新对象，值为 100
> obj2 = 101;              // Integer.valueOf(101)，缓存对象
> obj1++;                  // 拆箱 → 101 → Integer.valueOf(101) → 缓存对象
> ```
> - `obj1++` 对 `new Integer(100)` 拆箱得 `100`，加一得 `101`，再通过 `Integer.valueOf(101)` 装箱，返回缓存中的 `Integer(101)`。
> - `obj2` 同样指向缓存中的 `Integer(101)`，两者为同一对象。
> - `obj1 == obj2` 为 `true`。
>
> > [!note]
> > Exercise 4 与 Exercise 6 的对比说明：即使值在缓存范围内，只要任意一方通过 `new` **直接**创建（而非经由自增等触发 `valueOf()` 的操作），`==` 就可能为 `false`。`new` 创建的对象在参与运算并重新装箱后，其结果才会回归缓存机制；但 `new` 创建的对象本身永远不在缓存中。

---
`Pre: ` [[ELEC2543 Ch.11 Variables and Object Comparison]]
`Post:` [[ELEC2543 Ch.13 Interface]]

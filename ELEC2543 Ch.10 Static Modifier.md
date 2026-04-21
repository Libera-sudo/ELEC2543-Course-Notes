#Y2S2 #ELEC2543
# Chapter 10. `static` 修饰符

`Pre: ` [[ELEC2543 Ch.9 Method]]
`Post:` [[ELEC2543 Ch.11 Variables and Object Comparison]]

> [!abstract]
> `static` 修饰符 (*Static Modifier*) 是面向对象语言中实现类级别共享机制的核心语言特性。它将成员的归属从对象 (*Object*) 实例提升到类 (*Class*) 本身，使成员的生命周期与类同步，不再依赖任何具体对象的创建与销毁。
>
> Java 的 `static` 在语义上与 Python 的类变量、C++ 的静态成员相近，但在访问规则上更为严格：编译器在编译期就对"静态方法不得直接引用实例成员"等约束做检查，不允许绕过。这一取向体现了 Java 以显式归属关系控制类与对象边界的设计偏好。
>
> 本章涵盖两个核心模块：静态方法 (*Static Method*) 的定义规则与调用方式，以及静态变量 (*Static Variable*) 的共享语义与生命周期，最后以一个典型示例展示二者如何协同维护类级别共享状态。

```table-of-contents
maxLevel: 3
```

## 10.1 静态方法

静态方法 (*Static Method*) 是由 `static` 修饰的方法；`static` 作为修饰符 (*Modifier*)，核心语义是将该成员与类本身绑定，而非与类的某个具体对象实例绑定。按惯例，修饰符的书写顺序为可见性修饰符在前、`static` 在后，如 `public static`，但编译器不强制要求此顺序。

静态方法也称为类方法 (*Class Method*)，通过类名直接调用，无需创建对象。`Math` 类中的方法均为静态方法，例如 `Math.cos(90)`、`Math.sqrt(n)`；`main` 方法也是静态方法，JVM 在启动时无需实例化任何对象即可直接调用它。

> [!note]
> `main` 方法必须声明为 `static`，否则 JVM 无法启动程序。JVM 启动时尚未创建任何对象，若 `main` 为实例方法，JVM 就需要先创建该类的对象才能调用 `main`，而创建对象本身又可能依赖 `main` 的执行，形成循环依赖。

静态方法对变量的访问存在以下限制：

- 静态方法不能引用实例变量 (*Instance Variable*)，因为实例变量在对象创建之前不存在。
- 静态方法可以引用静态变量，因为静态变量与类同生命周期，不依赖对象存在。
- 静态方法可以引用局部变量 (*Local Variable*)，局部变量在方法调用时创建，与对象无关。
- 同一个类中，静态方法可以直接调用其他静态方法，无需通过类名或对象名。

> [!warning]
> 静态方法中引用实例变量会直接被编译器拒绝。静态方法在类加载时即存在，此时未必已有任何对象实例，实例变量尚未分配内存地址；若允许此类引用，运行时将无法解析目标变量的位置。
>
> > [!bug]
> >
> > ```java
> > public class Counter {
> >     private int value = 0;  // 实例变量
> >
> >     public static void increment() {
> >         value++;  // 编译错误
> >     }
> > }
> > ```
> >
> > ```
> > java: non-static variable value cannot be referenced from a static context
> > ```
>
> 正确写法是将方法改为实例方法，通过 `this`（显式或隐式）访问实例变量；若必须保持静态，则需显式接收对象引用作为参数：
>
> ```java
> public void increment() {
>     this.value++;                      // 方案一：实例方法
> }
>
> public static void increment(Counter c) {
>     c.value++;                         // 方案二：静态方法 + 对象引用参数
> }
> ```

> [!example]- 示例：`MyMath.java` / `MyMathDemo.java`
>
> `MyMath.java`、`MyMathDemo.java` 共同演示静态方法的定义、跨类通过类名调用，以及同类静态方法之间的直接互调。`MyMath.java` 定义一组静态工具方法（奇偶与质数判断）；`MyMathDemo.java` 为主程序，随机生成整数并调用 `MyMath.isPrime` 判断结果。
>
> > [!info]- UML 框图
> >
> > ```
> > ┌──────────────────┐
> > │ MyMathDemo       │
> > │  + main          │
> > └────────┬─────────┘
> >          │ 调用
> >          v
> > ┌──────────────────────────────────┐
> > │ MyMath                           │
> > │  + static isOdd(int): boolean    │
> > │  + static isEven(int): boolean   │
> > │  + static isPrime(int): boolean  │
> > └──────────────────────────────────┘
> > ```
>
> ```java
> // MyMath.java —— 静态工具方法集合
> public class MyMath {
>     // 判断 n 是否为奇数
>     public static boolean isOdd(int n) {
>         return (n % 2) == 1;
>     }
>
>     // 判断 n 是否为偶数
>     public static boolean isEven(int n) {
>         return (n % 2) == 0;
>     }
>
>     // 判断 n 是否为质数
>     public static boolean isPrime(int n) {
>         if (n <= 1) return false;       // 1 及以下不是质数
>         if (n == 2) return true;        // 2 是最小的质数
>         if (isEven(n)) return false;    // 大于 2 的偶数不是质数（直接调用同类静态方法）
>
>         // 若 n 有因子，则必有一个 <= sqrt(n)，故只需检验到 sqrt(n)
>         for (int i = 3; i <= Math.sqrt(n) + 1; i++) {
>             if (n % i == 0) {
>                 return false;
>             }
>         }
>         return true;
>     }
> }
> ```
>
> ```java
> // MyMathDemo.java —— 随机生成整数并判断是否为质数
> public class MyMathDemo {
>     public static void main(String[] args) {
>         int num = (int) (Math.random() * 20);   // 0~19 之间的随机整数
>
>         System.out.print(num + " is ");
>
>         // 通过类名直接调用静态方法，无需创建 MyMath 对象
>         if (MyMath.isPrime(num) == false)
>             System.out.print("not ");
>
>         System.out.println("a prime number.");
>     }
> }
> ```
>
> 输出（示例，结果随机）：
>
> ```
> 13 is a prime number.
> ```
>
> 或：
>
> ```
> 14 is not a prime number.
> ```
>
> **静态方法的声明**
>
> ```java
> public static boolean isEven(int n) {
>     return (n % 2) == 0;
> }
> ```
>
> - `public static` 表明该方法属于类本身，可通过 `MyMath.isEven(n)` 直接调用。
> - 方法体内只使用参数 `n`（局部变量），不涉及任何实例变量，符合静态方法的访问规则。
>
> **同类静态方法互调**
>
> ```java
> if (isEven(n)) return false;
> ```
>
> - `isPrime` 在同一类内直接调用 `isEven`，无需写 `MyMath.isEven(n)`。
> - 编译器将省略类名的调用解析为 `MyMath.isEven(n)`。
>
> **质数判断的循环边界**
>
> ```java
> for (int i = 3; i <= Math.sqrt(n) + 1; i++)
> ```
>
> - 若 $n$ 有因子，则必有一个 $\leq \sqrt{n}$，因此只需检验到 $\sqrt{n}$。
> - 循环起点为 $3$（$2$ 已在前面单独处理），步长为 $1$。
>
> **跨类通过类名调用静态方法**
>
> ```java
> if (MyMath.isPrime(num) == false)
> ```
>
> - `MyMathDemo` 未创建任何 `MyMath` 对象，直接以类名 `MyMath` 调用静态方法，这是静态方法的标准调用方式。
>
> > [!note]
> > `isPrime` 的循环以步长 $1$ 递增，会检验偶数值 `i`（如 `i = 4, 6, ...`）。由于已在循环前排除了偶数 `n`，这些检验不影响正确性，只是略有冗余。更高效的写法是将步长改为 $2$（`i += 2`），仅检验奇数因子。

## 10.2 静态变量

每个对象拥有独立的数据空间，各实例变量互不干扰。静态变量 (*Static Variable*) 不同，该变量在内存中只存一份副本，由该类的所有对象共享，因此也称为类变量 (*Class Variable*)。

其声明格式为：

```java
<可见性> static <类型> <变量名>
```

例如：

```java
private static float price;
```

静态变量的关键特性如下：

- 内存空间在类被首次引用时分配，早于任何对象的创建。
- 该类所有已实例化的对象共享同一个静态变量。
- 任一对象修改静态变量的值，其他对象访问到的值随之改变。
- 静态变量可在声明时直接初始化，如 `private static int count = 0`。

静态变量可声明为 `public`，通过类名直接从外部访问，如 `ClassName.variableName`；但通常声明为 `private`，再通过静态方法提供受控访问，以维护封装 (*Encapsulation*)。

> [!note]
> 静态变量与实例变量的本质区别在于归属层级：实例变量属于对象，静态变量属于类。可以将静态变量理解为"所有对象共用的全局状态"，实例变量则是"每个对象私有的局部状态"。
>
> 在内存模型上，实例变量存储在堆 (*Heap*) 上每个对象各自的内存区域中；静态变量存储在方法区 (*Method Area*) 中，与类的元数据存放在一起，具体布局依 JVM 实现而定。

## 10.3 静态成员的协同使用

静态方法与静态变量常配合使用，构成一种典型的类级别状态管理模式：由静态变量维护跨对象的共享状态，由静态方法提供对该状态的访问接口。这种模式无需创建额外对象即可查询类的全局信息。

> [!example]- 示例：`Slogan.java` / `SloganCounter.java`
>
> `Slogan.java`、`SloganCounter.java` 共同演示静态变量与静态方法如何协同维护类级别共享状态。`Slogan.java` 使用静态变量追踪自身已创建的实例总数，并通过静态方法 `getCount` 对外暴露；`SloganCounter.java` 为主程序，创建若干 `Slogan` 对象后通过类名调用 `Slogan.getCount` 获取总计数。
>
> > [!info]- UML 框图
> >
> > ```
> > ┌──────────────────┐
> > │ SloganCounter    │
> > │  + main          │
> > └────────┬─────────┘
> >          │ 调用
> >          v
> > ┌────────────────────────────────┐
> > │ Slogan                         │
> > │  - phrase : String             │
> > │  - static count : int          │
> > │  + Slogan(String)              │
> > │  + toString()                  │
> > │  + static getCount(): int      │
> > └────────────────────────────────┘
> > ```
>
> ```java
> // Slogan.java —— 表示一条口号，并记录已创建的实例总数
> public class Slogan {
>     private String phrase;          // 实例变量：每个对象独有的口号内容
>     private static int count = 0;   // 静态变量：全类共享，记录实例化次数
>
>     // 构造器：设置口号内容，并将计数加一
>     public Slogan(String str) {
>         phrase = str;
>         count++;
>     }
>
>     // 返回该口号的字符串表示
>     public String toString() {
>         return phrase;
>     }
>
>     // 静态方法：返回已创建的实例数量，通过类名调用
>     public static int getCount() {
>         return count;
>     }
> }
> ```
>
> ```java
> // SloganCounter.java —— 创建若干 Slogan 对象并输出创建总数
> public class SloganCounter {
>     public static void main(String[] args) {
>         Slogan obj;                 // 声明引用变量，尚未创建对象
>
>         obj = new Slogan("Remember the Alamo.");
>         System.out.println(obj);
>
>         obj = new Slogan("Don't Worry. Be Happy.");
>         System.out.println(obj);
>
>         obj = new Slogan("Live Free or Die.");
>         System.out.println(obj);
>
>         obj = new Slogan("Talk is Cheap.");
>         System.out.println(obj);
>
>         obj = new Slogan("Write Once, Run Anywhere.");
>         System.out.println(obj);
>
>         System.out.println();
>         // 通过类名调用静态方法，无需持有任何 Slogan 对象
>         System.out.println("Slogans created: " + Slogan.getCount());
>     }
> }
> ```
>
> 输出：
>
> ```
> Remember the Alamo.
> Don't Worry. Be Happy.
> Live Free or Die.
> Talk is Cheap.
> Write Once, Run Anywhere.
>
> Slogans created: 5
> ```
>
> **静态变量的声明与初始化**
>
> ```java
> private static int count = 0;
> ```
>
> - `count` 属于 `Slogan` 类本身，在类首次被引用时初始化为 `0`，早于任何对象的创建。
> - `phrase` 是实例变量，每个 `Slogan` 对象拥有独立副本；`count` 只有一份，全类共享。
>
> **构造器中递增静态变量**
>
> ```java
> public Slogan(String str) {
>     phrase = str;
>     count++;
> }
> ```
>
> - 每次 `new Slogan(...)` 触发构造器，`count++` 使共享计数器递增。
> - 构造器作为实例方法，同时访问实例变量 `phrase` 与静态变量 `count`。
>
> **静态方法访问静态变量**
>
> ```java
> public static int getCount() {
>     return count;
> }
> ```
>
> - `getCount` 为静态方法，只访问静态变量 `count`，符合静态方法的访问规则。
> - 若此处改为访问实例变量 `phrase`，编译器将报错。
>
> **通过类名调用静态方法**
>
> ```java
> System.out.println("Slogans created: " + Slogan.getCount());
> ```
>
> - 使用类名 `Slogan` 而非某个具体对象调用 `getCount`，明确表达该方法属于类级别。
> - 此时 `obj` 引用的是最后一个创建的 `Slogan` 对象，但计数器记录的是全部 $5$ 次实例化。
>
> > [!note]
> > `SloganCounter` 中变量 `obj` 被反复赋值为新创建的 `Slogan` 对象。每次赋值后，前一个对象不再被 `obj` 引用，但 `count` 已在构造时完成递增，不受引用丢失影响。静态变量的生命周期与对象引用无关，持续到程序结束。

---
`Pre: ` [[ELEC2543 Ch.9 Method]]
`Post:` [[ELEC2543 Ch.11 Variables and Object Comparison]]

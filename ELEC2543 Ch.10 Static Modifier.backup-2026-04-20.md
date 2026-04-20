#Y2S2 #ELEC2543
# Chapter 10. 修饰符

`Pre: ` [[ELEC2543 Ch.9 Method]]
`Post:` [[ELEC2543 Ch.11 Variables and Object Comparison]]

> [!abstract]
> `static` 修饰符 (_Static Modifier_)  Java 类级别共享机制的核心语言特性。
>
> 面向对象程序设计中，成员（变量与方法）默认归属于某个具体的对象实例，每个对象拥有独立的数据副本。Java 中 `static` 将成员与类本身绑定，而非与任何特定对象实例绑定。这意味着静态成员在类首次被引用时即分配内存，独立于对象的创建与销毁生命周期，所有实例共享同一份数据或方法入口。这与 Python 的类变量在语义上相近，但 Java 通过编译期类型检查对静态与非静态成员的访问施加了更严格的约束。
>
> 本章涵盖两个核心模块：静态方法 (_Static Method_) 的定义规则与调用方式，以及静态变量 (_Static Variable_) 的共享语义与生命周期。
```table-of-contents
maxLevel: 3
```
## 10.1 静态方法

静态方法 (*Static Method*) 是由 `static` 修饰的方法。`static` 是一个修饰符，可用于声明变量或方法的属性，其核心语义是将该成员与类本身绑定，而非与类的某个具体对象实例绑定。

按照惯例，修饰符的书写顺序为可见性修饰符在前、`static` 在后，如 `public static`，但编译器不强制要求此顺序。

静态方法也称为类方法 (*Class Method*)，通过类名直接调用，无需创建对象。`Math` 类中的方法均为静态方法，例如 `Math.cos(90)`、`Math.sqrt(n)`。`main` 方法也是静态方法，JVM 在启动时无需实例化任何对象即可直接调用它。

> [!note]
> `main` 必须是 `static` ：
>
> JVM 启动时尚未创建任何对象，若 `main` 是实例方法，JVM 就需要先创建该类的对象才能调用 `main`，而创建对象本身又可能依赖 `main` 的执行，形成循环依赖。

静态方法对变量的访问存在以下限制：

- 静态方法不能引用实例变量 (*Instance Variable*)，因为实例变量在对象创建之前不存在。
- 静态方法可以引用静态变量，因为静态变量与类同生命周期，不依赖对象存在。
- 静态方法可以引用局部变量 (*Local Variable*)，局部变量在方法调用时创建，与对象无关。
- 同一个类中，静态方法可以直接调用其他静态方法，无需通过类名或对象名。

> [!note]
> 在静态方法中引用实例变量会导致编译错误：
> > [!bug]
> >
> > ```java
> > public class Counter {
> >     private int value = 0;  // 实例变量：每个 Counter 对象拥有独立副本
> >
> >     // 静态方法尝试访问实例变量
> >     public static void increment() {
> >         value++;  // 编译错误：无法从静态上下文中引用非静态变量 value
> >     }
> > }
> > ```
> >
> >
> > ```
> > java: non-static variable value cannot be referenced from a static context
> > ```
>
> **错误分析**
> - 静态方法 `increment()` 在类加载时即存在于方法区，此时可能没有任何 `Counter` 对象被创建。
> - 实例变量 `value` 存储于堆内存的对象实例内部，若对象不存在，则 `value` 无内存地址。
> - 编译器拒绝此类引用，以避免在运行时出现空引用或内存访问异常。
>
> **正确写法**
> ```java
> public class Counter {
>     private int value = 0;
>
>     // 方案一：改为实例方法，通过 this 引用（隐式或显式）
>     public void increment() {
>         this.value++;  // 正确：实例方法依附于具体对象，value 已分配内存
>     }
>
>     // 方案二：若必须保持静态，则需显式传入对象引用
>     public static void increment(Counter c) {
>         if (c != null) {
>             c.value++;  // 正确：通过对象引用访问其实例变量
>         }
>     }
> }
> ```
>
> ```java
> // 使用示例
> Counter c = new Counter();
> Counter.increment(c);  // 静态方法通过参数接收对象引用
> // 或
> c.increment();         // 实例方法直接操作自身状态
> ```

> [!example]- 示例：`MyMath.java` / `MyMathDemo.java`
> 演示如何定义一组静态工具方法，并在另一个类中通过类名调用，同时展示静态方法之间的直接互调。
>
> ```java
> // MyMath.java
> // 演示静态方法的定义，static 修饰符置于可见性修饰符之后
>
> public class MyMath {
>
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
>         // 只需检验到 sqrt(n)，若存在因子必有一个 <= sqrt(n)
>         for (int i = 3; i <= Math.sqrt(n) + 1; i++) {
>             if (n % i == 0) {
>                 return false;           // 找到因子，不是质数
>             }
>         }
>         return true;
>     }
> }
> ```
>
> ```java
> // MyMathDemo.java
> // 随机生成一个数，判断其是否为质数
>
> public class MyMathDemo {
>
>     public static void main(String[] args) {
>         // 生成 0~19 之间的随机整数
>         int num = (int) (Math.random() * 20);
>
>         System.out.print(num + " is ");
>
>         // 通过类名调用静态方法，无需创建 MyMath 对象
>         if (MyMath.isPrime(num) == false)
>             System.out.print("not ");
>
>         System.out.println("a prime number.");
>     }
> }
> ```
>
> 输出（示例，结果随机）：
> ```
> 13 is a prime number.
> ```
> 或
> ```
> 14 is not a prime number.
> ```
>
> **静态方法的声明**
> ```java
> public static boolean isEven(int n) {
>     return (n % 2) == 0;
> }
> ```
> - `public static` 表明该方法属于类本身，可通过 `MyMath.isEven(n)` 直接调用。
> - 方法体内只使用了参数 `n`（局部变量），不涉及任何实例变量，符合静态方法的访问规则。
>
> **静态方法直接调用同类静态方法**
> ```java
> if (isEven(n)) return false;
> ```
> - `isPrime` 在同一类内直接调用 `isEven`，无需写 `MyMath.isEven(n)`。
> - 这等价于 `MyMath.isEven(n)`，编译器会自动解析。
>
> **质数判断的循环上界**
> ```java
> for (int i = 3; i <= Math.sqrt(n) + 1; i++)
> ```
> - 若 $n$ 有因子，则必有一个因子 $\leq \sqrt{n}$，因此只需检验到 $\sqrt{n}$ 即可。
> - 步长为 2（隐含：已排除偶数，但此处 `i++` 仍逐一递增，包含偶数 `i`）。
>
> **通过类名调用静态方法**
> ```java
> if (MyMath.isPrime(num) == false)
> ```
> - `MyMathDemo` 中未创建任何 `MyMath` 对象，直接以类名 `MyMath` 调用静态方法。
> - 这是静态方法的标准调用方式。
>
> > [!note]
> > `isPrime` 的循环中 `i` 以步长 1 递增，会检验偶数值（如 `i=4, 6, ...`），但由于已在循环前排除了偶数 `n`，这些检验不影响正确性，只是略有冗余。更高效的写法是将步长改为 2（`i += 2`），仅检验奇数因子。

## 10.2 静态变量

通常而言，每个对象拥有独立的数据空间，各实例变量互不干扰。

静态变量 (*Static Variable*) 不同，该变量在内存中**只存在一份副本**，由该类的所有对象共享，因此也称为类变量 (*Class Variable*)。

```java
/*
声明格式：
visibility static type nameOfVariable;
*/

private static float price;
```

静态变量的关键特性如下：

- 内存空间在类被首次引用时分配，早于任何对象的创建。
- 该类所有已实例化的对象共享同一个静态变量。
- 静态变量的值被任意一个对象修改，其于其他对象处的值也随之改变。
- 静态变量可以在声明时**直接初始化**，如 `private static int count = 0`。

> [!note]
> 静态变量与实例变量的本质区别在于归属层级：实例变量属于对象，静态变量属于类。可以将静态变量理解为"所有对象共用的全局状态"，而实例变量是"每个对象私有的局部状态"。
>
> 在内存模型上，实例变量存储在堆 (*Heap*) 上每个对象各自的内存区域中；静态变量存储在方法区 (*Method Area*) 中，与类的元数据存放在一起，具体行为依 JVM 实现而定。

> [!note]
> 静态变量可声明为 `public`，通过类名直接从外部访问，如 `ClassName.variableName`。
>
> 但静态变量通常被声明为 `private`，并通过静态方法提供受控访问，以维护封装性。

## 10.3 静态成员的协同使用

静态方法与静态变量常配合使用，构成一种典型的类级别状态管理模式：由静态变量维护跨对象的共享状态，由静态方法提供对该状态的访问接口。

这种模式无需创建额外对象即可查询类的全局信息。

> [!example]- 示例：`Slogan.java` + `SloganCounter.java`
> 演示静态变量与静态方法协同工作，追踪某个类的对象实例化次数。
>
> ```java
> // Slogan.java
> // 表示一条口号字符串，同时记录已创建的实例总数
>
> public class Slogan {
>     private String phrase;          // 实例变量：每个对象独有的口号内容
>     private static int count = 0;   // 静态变量：所有对象共享，记录实例化次数
>
>     // 构造方法：设置口号内容，并将计数加一
>     public Slogan(String str) {
>         phrase = str;
>         count++;    // 每次创建新对象时，类级别计数器递增
>     }
>
>     // 返回该口号的字符串表示
>     public String toString() {
>         return phrase;
>     }
>
>     // 静态方法：返回已创建的实例数量，通过类名调用
>     public static int getCount() {
>         return count;   // 访问静态变量，合法
>     }
> }
> ```
>
> ```java
> // SloganCounter.java
> // 演示 static 修饰符的使用，创建若干 Slogan 对象并输出创建总数
>
> public class SloganCounter {
>
>     public static void main(String[] args) {
>         Slogan obj;   // 声明引用变量，尚未创建对象
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
> ```java
> private static int count = 0;
> ```
> - `count` 属于 `Slogan` 类本身，在类首次被引用时初始化为 `0`，早于任何对象的创建。
> - `phrase` 是实例变量，每个 `Slogan` 对象拥有独立的 `phrase`；`count` 只有一份，所有对象共享。
>
> **构造方法中递增静态变量**
> ```java
> public Slogan(String str) {
>     phrase = str;
>     count++;
> }
> ```
> - 每次调用 `new Slogan(...)` 时构造方法执行一次，`count++` 使共享计数器递增。
> - 构造方法是实例方法，可以同时访问实例变量（`phrase`）和静态变量（`count`）。
>
> **静态方法访问静态变量**
> ```java
> public static int getCount() {
>     return count;
> }
> ```
> - `getCount()` 是静态方法，只访问静态变量 `count`，符合静态方法的访问规则。
> - 若此处改为访问实例变量 `phrase`，编译器将报错。
>
> **通过类名调用静态方法**
> ```java
> System.out.println("Slogans created: " + Slogan.getCount());
> ```
> - 使用类名 `Slogan` 而非某个具体对象调用 `getCount()`，明确表达该方法属于类级别。
> - 此时 `obj` 引用的是最后一个创建的 `Slogan` 对象，但计数器记录的是全部 5 次实例化。
>
> > [!note]
> > 注意 `SloganCounter` 中，变量 `obj` 被反复赋值为新创建的 `Slogan` 对象。每次赋值后，前一个对象不再被 `obj` 引用，但 `count` 已经在构造时完成了递增，不受引用丢失的影响。
> >
> > 静态变量的生命周期与对象引用无关，持续到程序结束。

---
`Pre: ` [[ELEC2543 Ch.9 Method]]
`Post:` [[ELEC2543 Ch.11 Variables and Object Comparison]]

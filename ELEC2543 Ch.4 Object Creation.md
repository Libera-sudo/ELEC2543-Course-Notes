#Y2S2  #ELEC2543
# Chapter 4. 创建对象

`Pre: ` [[ELEC2543 Ch.3 Introduction to Classes]]
`Post:` [[ELEC2543 Ch.5 Data Visibility]]

> [!abstract]
> 对象创建 (*Object Creation*) 是面向对象编程 (*Object-Oriented Programming*) 的基础操作，涉及内存分配、引用绑定与对象生命周期管理。
>
> 与 C/C++ 不同，Java 将对象创建与内存回收的细节封装于 JVM 之中：程序员通过 `new` 运算符实例化对象，而无需手动释放内存；引用变量存储的是对象地址而非对象本体，这一设计深刻影响赋值、别名与参数传递的行为。
>
> 本章涵盖以下核心模块：对象引用变量与基本类型变量的区别、`new` 运算符与构造器调用、引用赋值与别名现象、垃圾回收机制 (*Garbage Collection*)、以及 `this` 引用的作用与用法。
```table-of-contents
maxLevel: 3
```

## 4.1 对象创建

在 Java 中，变量声明分为两类：持有基本类型值的变量，以及持有对象引用 (*Reference*) 的变量。

```java
int num;    // 基本类型变量，未初始化
Die die1;   // 引用变量，未初始化，不创建任何 Die 对象
```

声明语句如 `int num;` 或 `Die die1;` 仅创建变量槽位，不分配对象内存，两者均处于未初始化状态。引用变量存储的是对象在堆内存 (*Heap Memory*) 中的地址，对象本体必须另行创建。未初始化的引用变量在内存图中通常表示为 `--`，表示尚未指向任何对象。

> [!note]
> 声明引用变量与创建对象是两个独立步骤。仅声明 `Die die1;` 时，不存在任何 `Die` 对象；试图通过未初始化的引用调用方法将抛出 `NullPointerException`。

使用 `new` 运算符创建对象，其一般形式为：

```java
new <类名>(<变量>)
```

`new` 在堆内存中为对象分配空间，并调用对应的构造器 (*Constructor*) 完成初始化。创建对象的过程称为实例化 (*Instantiation*)，所创建的对象称为该类的一个实例 (*Instance*)。构造器是与类同名的特殊方法，负责设置对象的初始状态。

```java
die1 = new Die(5);   // 调用带参构造器，faceValue 初始化为 5
```

基本类型变量直接存储值本身；引用变量则存储对象的**内存地址**，并通过该地址间接访问对象。

```java
int num = 42;         // num 直接存储整数值 42
Die die1 = new Die(); // die1 存储 Die 对象的地址，faceValue 默认为 1
Die die2 = new Die(2);// die2 存储另一个 Die 对象的地址，faceValue 为 2
```

内存示意：

```
num  → [ 42 ]

die1 → [ address ] >>> [ faceValue = 1 ]

die2 → [ address ] >>> [ faceValue = 2 ]
```

`num` 的方框内直接保存值 `42`；`die1` 与 `die2` 的方框内保存地址，并分别指向堆中两个独立对象。`faceValue` 是类 `Die` 的实例变量 (*Instance Variable*)，每个对象独立持有一份。

## 4.2 赋值与别名

基本类型赋值会把右侧的值复制一份存入左侧变量，因此两个变量在赋值后相互独立。

```java
int num1 = 38;
int num2 = 96;
num2 = num1;   // num2 获得 num1 的值的副本
// 此后 num1 = 38, num2 = 38，互不影响
```

引用类型赋值复制的是地址而非对象本体，因此赋值后两个变量会指向**同一对象**。

```java
Die die1 = new Die();    // faceValue = 1
Die die2 = new Die(2);   // faceValue = 2
die2 = die1;             // die2 获得 die1 的地址副本
```

赋值后内存示意：

```
die1 → [ address ] >>> [ faceValue = 1 ]
                              ↑
die2 → [ address ]  ──────────┘

// 原 faceValue = 2 的对象不再被任何引用指向
```

`die2` 原先指向的对象（`faceValue = 2`）失去所有引用，成为垃圾 (*Garbage*)。`die1` 与 `die2` 现在互为**别名** (*Alias*)，通过任意一个引用修改对象，另一个看到的状态都会同步改变。

#### 别名的风险

通过一个引用修改对象时，所有别名都会受到影响，因为底层始终只有一个对象。别名是引用赋值的直接结果，若未意识到状态正在共享，就容易产生意外修改。

> [!example]- 示例：`MyInteger.java` / `TestMyInteger.java`
>
> 这一组程序展示了引用赋值如何产生别名，以及别名如何让多个引用共同观察同一个对象状态。
>
> > [!info]- UML 框图
> >
> > ```
> > TestMyInteger.java
> >         |
> >         v
> >    MyInteger.java
> > ```
>
> `MyInteger.java` 定义一个只包含单个整数状态的简单类，用来观察引用赋值后的对象共享现象。
>
> ```java
> // MyInteger.java
> public class MyInteger {
>     private int num;
>
>     MyInteger(int i) {
>         num = i;
>     }
>
>     public void changeValue(int newValue) {
>         num = newValue;
>     }
>
>     public String toString() {
>         return "num = " + num;
>     }
> }
> ```
>
> `TestMyInteger.java` 负责创建对象、执行引用赋值，并输出赋值前后的状态变化。
>
> ```java
> // TestMyInteger.java
> public class TestMyInteger {
>     public static void main(String[] args) {
>         MyInteger obj1, obj2;
>
>         obj1 = new MyInteger(100);
>         obj2 = new MyInteger(200);
>
>         System.out.println(obj1);  // num = 100
>         System.out.println(obj2);  // num = 200
>
>         obj2 = obj1;               // obj2 成为 obj1 的别名
>         obj1.changeValue(300);     // 修改唯一对象的 num
>
>         System.out.println(obj1);  // num = 300
>         System.out.println(obj2);  // num = 300
>     }
> }
> ```
>
> 输出：
>
> ```
> num = 100
> num = 200
> num = 300
> num = 300
> ```
>
> **对象初始化**
> ```java
> obj1 = new MyInteger(100);
> obj2 = new MyInteger(200);
> ```
> - 创建两个独立的 `MyInteger` 对象，`obj1.num = 100`，`obj2.num = 200`。
> - 前两行 `println` 分别调用各自对象的 `toString()`，输出互不干扰。
>
> **引用赋值**
> ```java
> obj2 = obj1;
> ```
> - `obj2` 获得 `obj1` 所存地址的副本，两者指向同一对象（`num = 100`）。
> - 原 `num = 200` 的对象失去所有引用，成为垃圾，等待垃圾回收。
>
> **通过别名修改**
> ```java
> obj1.changeValue(300);
> ```
> - `changeValue` 将该唯一对象的 `num` 改为 `300`。
> - `obj1` 与 `obj2` 指向同一对象，故两者的 `toString()` 均输出 `num = 300`。

> [!question] 习题：识别垃圾对象
> 在第 2 节示例程序 `TestMyInteger.java` 中，是否产生了垃圾对象？若有，描述其内容。
>
> > [!check]-
> > 程序确实产生了垃圾对象，关键发生在引用重新赋值之后。
> >
> > ```java
> > obj1 = new MyInteger(100);  // 对象 A：num = 100
> > obj2 = new MyInteger(200);  // 对象 B：num = 200
> > obj2 = obj1;                // obj2 转而指向对象 A
> > ```
> >
> > **引用重绑定**
> >
> > - 执行 `obj2 = obj1` 后，`obj2` 不再指向原先 `num = 200` 的对象，而是改为指向对象 A。
> > - 原先由 `new MyInteger(200)` 创建的对象失去全部引用，因此成为垃圾对象。
> >
> > **结论**
> >
> > - 程序产生了一个垃圾对象，即最初由 `new MyInteger(200)` 创建、内部状态为 `num = 200` 的 `MyInteger` 实例。

> [!question] 习题：拷贝构造器 (*Copy Constructor*)
> 已知 `Die` 类新增如下构造器与方法：
>
> ```java
> public Die(Die die) {
>     faceValue = die.getFaceValue();
> }
>
> public int getFaceValue() {
>     return faceValue;
> }
> ```
>
> 求以下代码的输出：
>
> ```java
> Die die1 = new Die();       // faceValue = 1（默认值）
> Die die2 = new Die(3);      // faceValue = 3
> Die die3 = new Die(die2);   // 拷贝构造器，复制 die2 当前状态
> die1.setFaceValue(4);       // die1.faceValue = 4
> die2.setFaceValue(5);       // die2.faceValue = 5
> System.out.println(die1);   // 4
> System.out.println(die2);   // 5
> System.out.println(die3);   // ?
> ```
>
> > [!check]-
> > 这道题的关键在于区分“复制对象状态”和“复制引用地址”。拷贝构造器会创建新对象，因此后续对 `die2` 的修改不会影响 `die3`。
> >
> > **拷贝构造**
> >
> > ```java
> > Die die3 = new Die(die2);
> > ```
> >
> > - 这一步创建了一个**全新的** `Die` 对象，并把 `die2` 当时的 `faceValue` 复制给新对象。
> > - 因此 `die3` 与 `die2` 指向不同对象，只是初始状态相同。
> >
> > **后续修改**
> >
> > ```java
> > die2.setFaceValue(5);
> > ```
> >
> > - 这一步只会修改 `die2` 所指对象的状态，不会影响 `die3`。
> >
> > 输出：
> >
> > ```
> > 4
> > 5
> > 3
> > ```

## 4.3 垃圾回收

当一个对象不再被任何引用变量指向时，程序无法再访问它，该对象即成为垃圾 (*Garbage*)。

Java 由 JVM 自动周期性地执行垃圾回收，将垃圾对象占用的堆内存归还系统以供后续使用，程序员无需手动释放内存。

这降低了内存泄漏 (*Memory Leak*) 的风险。C 和 C++ 中程序员须手动调用 `free` / `delete` 释放内存，管理不当易引发内存泄漏或悬空指针 (*Dangling Pointer*)。

```java
Die die1 = new Die();    // 对象 A：faceValue = 1
Die die2 = new Die(2);   // 对象 B：faceValue = 2
die2 = die1;             // die2 转而指向对象 A
// 对象 B 不再被任何引用指向，成为垃圾
```

> [!note]
> 垃圾回收的具体触发时机由 JVM 决定，程序员无法精确控制。Java 提供 `System.gc()` 作为建议性调用，但 JVM 不保证立即执行回收。具体行为依 JVM 实现而定。

## 4.4 `this` 引用

`this` 是 Java 中的隐式引用，在实例方法或构造器内部指向当前正在执行该方法的对象本身。

当构造器参数名与实例变量 (*Instance Variable*) 同名时，`this.变量名` 明确指向实例变量，裸变量名则指向参数。

`this` 也可用于在方法内将当前对象作为参数传递给其他方法，或从方法中返回当前对象。

```java
public class Account {
    String name;
    long acctNumber;
    double balance;

    public Account(String name, long acctNumber, double balance) {
        this.name = name;           // this.name 是实例变量，name 是参数
        this.acctNumber = acctNumber;
        this.balance = balance;
    }
}
```

- 若不使用 `this`，直接写 `name = name;` 实为参数自赋值，实例变量不会被更新。
- 构造器参数与实例变量命名一致是常见的 Java 惯用写法，`this` 是消除歧义的标准手段。

> [!note]
> `this` 仅在实例方法与构造器中有效，静态方法 (*Static Method*) 中不存在 `this`，因为静态方法不依附于任何具体对象。

> [!question] 习题：`copy()` 方法的别名陷阱
> 设 `Die` 类中有如下方法：
>
> ```java
> public Die copy() {
>     return this;
> }
> ```
>
> 求以下代码的输出：
>
> ```java
> Die die1 = new Die();       // faceValue = 1（默认值）
> Die die2 = new Die(3);      // faceValue = 3
> Die die3 = die2.copy();     // 返回 this，即 die2 本身的引用
> die1.setFaceValue(4);       // die1.faceValue = 4
> die2.setFaceValue(5);       // die2.faceValue = 5
> System.out.println(die1);   // 4
> System.out.println(die2);   // 5
> System.out.println(die3);   // ?
> ```
>
> > [!check]-
> > 这道题考察的是 `this` 返回的并不是“当前对象的副本”，而是“当前对象本身的引用”。
> >
> > **返回 `this`**
> >
> > ```java
> > Die die3 = die2.copy();
> > ```
> >
> > - `copy()` 返回的是 `die2` 当前所指对象的引用本身，并不会创建新对象。
> > - 因此 `die3` 与 `die2` 成为别名，指向同一对象。
> >
> > **后续修改**
> >
> > ```java
> > die2.setFaceValue(5);
> > ```
> >
> > - 这一步修改的是两者共同指向的唯一对象，所以 `die3` 观察到的状态也同步变为 5。
> >
> > 输出：
> >
> > ```
> > 4
> > 5
> > 5
> > ```
> >
> > > [!note]
> > > 习题 2 与习题 3 的本质区别在于：拷贝构造器创建独立的新对象，而 `return this` 只返回现有对象的引用。若希望 `copy()` 实现真正的复制，应改为 `return new Die(this);`。

---
`Pre: ` [[ELEC2543 Ch.3 Introduction to Classes]]
`Post:` [[ELEC2543 Ch.5 Data Visibility]]

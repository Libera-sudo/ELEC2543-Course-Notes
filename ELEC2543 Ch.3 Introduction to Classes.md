#Y2S2 #ELEC2543
# Chapter 3. 类的基础

`Pre: ` [[ELEC2543 Ch.2 Expression & Java Syntax]]
`Post:` [[ELEC2543 Ch.4 Object Creation]]

> [!abstract]
> 类的基础内容位于面向对象程序设计 (*Object-Oriented Programming*) 的核心位置，讨论如何用统一的抽象结构描述对象的状态与行为，并据此组织程序中的数据与操作。
>
> Java 的类机制体现了封装 (*Encapsulation*) 的设计思想，将数据与行为绑定在同一实体内，并通过访问修饰符 (*Access Modifier*) 限制外部访问。与允许更自由组织源文件的语言相比，Java 对公有类与文件名的一致性、对象创建流程与内存管理方式都有更明确的语法约束。
>
> 本章具体介绍类的结构定义、构造器 (*Constructor*)、`toString()` 方法、实例数据 (*Instance Data*) 与作用域 (*Scope*)、访问器 (*Accessor*) 与修改器 (*Mutator*)，以及统一建模语言类图 (*Unified Modeling Language Class Diagram*) 的基本规范。
```table-of-contents
maxLevel: 3
```

## 3.1 类的定义与结构

在 Java 中，类 (*Class*) 是描述一组具有相同状态与行为的对象的蓝图 (*Blueprint*)。

对象 (*Object*) 具有两个核心要素：

- 状态 (*State*)：由属性 (*Attributes*) 定义，对应类中的数据声明。
- 行为 (*Behavior*)：由操作 (*Operations*) 定义，对应类中的方法声明。

一个 Java 文件只能包含一个公共类，且文件名必须与类名完全一致。

一个程序可由多个类（多个 `.java` 文件）组成，但**有且仅有一个类**包含 `main` 方法作为程序入口。

包含 `main` 方法的类仅是程序的起点，真正的面向对象设计体现在其他自定义类的定义与使用上。

```java
/*
类的基本结构
*/

public class ClassName {
    // 数据声明（定义状态）
    int size, weight;
    char category;

    // 方法声明（定义行为）
    public void someMethod() {
        // ...
    }
}
```

标准类库（如 `System`、`Math`）中的类是预先定义好的；实际开发中需要根据需求自行设计类。

以下这一组程序把类的定义与程序入口拆分到两个文件中，并通过对象创建、方法调用与字符串输出来展示类的基本使用方式。

> [!example]- 示例：`Die.java` / `RollingDice.java`
>
> 这一组程序展示了如何先定义 `Die` 类的状态与行为，再由 `RollingDice` 创建对象并驱动整个程序流程。
>
> > [!info]- UML 框图
> >
> > ```
> > RollingDice.java
> >      |
> >      v
> >    Die.java
> > ```
>
> `Die.java` 定义骰子的实例数据、构造器以及对外可调用的方法。
>
> ```java
> // Die.java
> public class Die {
>     private final int MAX = 6;   // 骰子的最大面值常量
>     private int faceValue;       // 当前面值，属于实例数据
>
>     // 构造器：把初始面值设为 1
>     public Die() {
>         faceValue = 1;
>     }
>
>     // 掷骰子并返回新的面值
>     public int roll() {
>         faceValue = (int)(Math.random() * MAX) + 1;
>         return faceValue;
>     }
>
>     // 修改当前面值
>     public void setFaceValue(int value) {
>         faceValue = value;
>     }
>
>     // 读取当前面值
>     public int getFaceValue() {
>         return faceValue;
>     }
>
>     // 返回面值的字符串表示
>     public String toString() {
>         String result = Integer.toString(faceValue);
>         return result;
>     }
> }
> ```
>
> `RollingDice.java` 负责创建两个 `Die` 对象，并按顺序调用它们的方法观察对象状态的变化。
>
> ```java
> // RollingDice.java
> public class RollingDice {
>     public static void main(String[] args) {
>         Die die1, die2;
>         int sum;
>
>         die1 = new Die();    // 创建第一个骰子对象
>         die2 = new Die();    // 创建第二个骰子对象
>
>         die1.roll();         // 两个对象分别掷一次
>         die2.roll();
>         System.out.println("Die One: " + die1 + ", Die Two: " + die2);
>
>         die1.roll();         // 再次更新 die1 的面值
>         die2.setFaceValue(4); // 直接把 die2 设为 4
>         System.out.println("Die One: " + die1 + ", Die Two: " + die2);
>
>         sum = die1.getFaceValue() + die2.getFaceValue();
>         System.out.println("Sum: " + sum);
>
>         sum = die1.roll() + die2.roll(); // 直接使用返回值求和
>         System.out.println("Die One: " + die1 + ", Die Two: " + die2);
>         System.out.println("New sum: " + sum);
>     }
> }
> ```
>
> 输出：
>
> 由于运行结果具有随机性，以下给出一种可能的输出。
>
> ```
> Die One: 5, Die Two: 2
> Die One: 1, Die Two: 4
> Sum: 5
> Die One: 4, Die Two: 2
> New sum: 6
> ```
>
> **`Die.java` 中的类定义**
>
> ```java
> private final int MAX = 6;
> private int faceValue;
> ```
>
> - `MAX` 以 `final` 修饰，为不可更改的常量，表示骰子的最大面值。
> - `faceValue` 是实例数据，每个 `Die` 对象都拥有独立副本。
> - `private` 限制外部直接访问，因此外部代码只能通过方法间接读写面值。
>
> **`Die.java` 中的初始化与行为**
>
> ```java
> public Die() {
>     faceValue = 1;
> }
>
> public int roll() {
>     faceValue = (int)(Math.random() * MAX) + 1;
>     return faceValue;
> }
> ```
>
> - 构造器名称与类名相同，没有返回类型；对象创建时自动调用，并把 `faceValue` 初始化为 1。
> - `Math.random()` 返回 $[0.0, 1.0)$ 的随机浮点数，乘以 `MAX` 后得到 $[0.0, 6.0)$，转换为 `int` 后落在 $[0, 5]$，再加 1 得到 $[1, 6]$。
>
> **`Die.java` 中的对外接口**
>
> ```java
> public void setFaceValue(int value) {
>     faceValue = value;
> }
>
> public int getFaceValue() {
>     return faceValue;
> }
>
> public String toString() {
>     String result = Integer.toString(faceValue);
>     return result;
> }
> ```
>
> - `setFaceValue()` 是修改器，负责更新对象状态；`getFaceValue()` 是访问器，负责读取对象状态。
> - `toString()` 把整数面值转换为字符串，因此对象参与字符串拼接时会输出可读的点数，而不是默认的对象标识。
>
> **`RollingDice.java` 中的对象使用**
>
> ```java
> die1 = new Die();
> die2 = new Die();
> ```
>
> - 两个引用变量分别指向两个独立对象，因此后续状态变化彼此独立。
> - `new Die()` 不仅分配对象内存，也会立即调用构造器完成初始化。
>
> **`RollingDice.java` 中的状态变化与输出**
>
> ```java
> die1.roll();
> die2.roll();
> System.out.println("Die One: " + die1 + ", Die Two: " + die2);
>
> die1.roll();
> die2.setFaceValue(4);
> sum = die1.getFaceValue() + die2.getFaceValue();
> ```
>
> - 第一次输出展示两个对象各自掷骰后的状态，字符串拼接中的 `die1` 与 `die2` 会自动调用各自的 `toString()`。
> - 第二次操作分别通过 `roll()` 与 `setFaceValue()` 更新状态，再通过 `getFaceValue()` 读取状态求和，说明对象既可以通过行为自主变化，也可以通过接口被外部显式修改。

## 3.2 数据作用域与实例数据

作用域 (*Scope*) 指程序中某个数据可被引用的范围。按声明位置划分，Java 中的数据主要分为实例数据 (*Instance Data*) 与局部变量 (*Local Variable*) 两类。

- 实例数据在类级别 (*Class Level*) 声明，位于所有方法之外；该类的所有方法均可访问，且每个对象实例都拥有独立副本。
- 局部变量在方法内部声明，只在当前方法内部有效，方法执行结束后销毁；例如 `Die` 类中 `toString()` 方法内的 `result` 变量，只能在 `toString()` 内部使用。

```java
public class Die {
    private int faceValue;   // 实例数据，所有方法可访问

    public String toString() {
        String result = Integer.toString(faceValue);  // 局部变量，仅 toString() 内有效
        return result;
    }
}
```

类的定义本身不分配内存，只描述数据的类型与结构。每次通过 `new` 创建对象时，JVM 为该对象的所有实例数据单独分配内存。所有对象共享同一份方法定义，但各自维护独立的数据空间，这正是两个对象能够拥有不同状态的根本原因。

以 `RollingDice` 中的两个 `Die` 对象为例：

```tikz
\usepackage{amsmath}
\begin{document}
\begin{tikzpicture}[>=stealth, font=\Large, scale=1.0]

\definecolor{axiscolor}{RGB}{200,200,200}
\definecolor{boxfill}{RGB}{170,20,100}
\definecolor{valuefill}{RGB}{100,160,255}
\definecolor{refbox}{RGB}{200,200,200}
\definecolor{labelcolor}{RGB}{255,100,100}

% ===== die1 =====
% Label
\node[axiscolor, font=\Large\bfseries\ttfamily] at (0, 4) {die1};

% Reference box
\draw[axiscolor, line width=1.2pt, fill=refbox!15] (1.2, 3.6) rectangle (2.4, 4.4);
\draw[axiscolor, line width=1.5pt, fill=axiscolor] (1.5, 3.85) rectangle (2.1, 4.15);

% Arrow from ref box to object
\draw[axiscolor, line width=1.5pt, ->] (2.4, 4) -- (4.5, 4);

% Object rounded box (die1)
\draw[black, line width=1.5pt, fill=boxfill, rounded corners=12pt] (4.5, 3.2) rectangle (9.8, 4.8);

% faceValue label
\node[black, font=\Large\bfseries\ttfamily] at (6.3, 4) {faceValue};

% Value box inside object
\draw[black, line width=1.2pt, fill=valuefill] (8.2, 3.5) rectangle (9.3, 4.5);
\node[white, font=\LARGE\bfseries] at (8.75, 4) {5};

% ===== die2 =====
% Label
\node[axiscolor, font=\Large\bfseries\ttfamily] at (0, 1) {die2};

% Reference box
\draw[axiscolor, line width=1.2pt, fill=refbox!15] (1.2, 0.6) rectangle (2.4, 1.4);
\draw[axiscolor, line width=1.5pt, fill=axiscolor] (1.5, 0.85) rectangle (2.1, 1.15);

% Arrow from ref box to object
\draw[axiscolor, line width=1.5pt, ->] (2.4, 1) -- (4.5, 1);

% Object rounded box (die2)
\draw[black, line width=1.5pt, fill=boxfill, rounded corners=12pt] (4.5, 0.2) rectangle (9.8, 1.8);

% faceValue label
\node[black, font=\Large\bfseries\ttfamily] at (6.3, 1) {faceValue};

% Value box inside object
\draw[black, line width=1.2pt, fill=valuefill] (8.2, 0.5) rectangle (9.3, 1.5);
\node[white, font=\LARGE\bfseries] at (8.75, 1) {2};

\end{tikzpicture}
\end{document}
```

- `die1.faceValue` 与 `die2.faceValue` 是两块独立的内存，修改其中一个不影响另一个。
- `roll()`、`toString()` 等方法定义只存在一份，由所有 `Die` 实例共用。

> [!note]
> 局部变量在使用前必须显式赋值，否则编译器报错。实例数据若未在构造器中初始化，Java 会赋予默认值（`int` 为 `0`，`boolean` 为 `false`，引用类型为 `null`）。

## 3.3 构造器

构造器 (*Constructor*) 是在对象被创建时自动调用的特殊方法，通常用于初始化实例数据。

- 构造器名称必须与类名完全相同。
- 构造器**没有返回类型**，不需要 `void`。

若程序员未定义任何构造器，编译器自动提供一个无参默认构造器 (*Default Constructor*)，其方法体为空。

同一个类可以定义多个构造器，但**参数列表必须不同**（参数个数或类型不同），这称为构造器重载 (*Constructor Overloading*)。

通过 `new ClassName(参数)` 调用，Java 根据传入参数自动匹配对应的构造器。

```java
public class Die {
    private int faceValue;

    // 无参构造器：初始化为 1
    public Die() {
        faceValue = 1;
    }

    // 带参构造器：初始化为指定值
    public Die(int v) {
        faceValue = v;
    }

    // 带参构造器：初始化为随机值（1~6）
    public Die(boolean random) {
        faceValue = (int)(Math.random() * 6) + 1;
    }
}
```

> [!note]
> 一旦程序员定义了任意一个构造器，编译器将**不再**自动提供默认无参构造器。若仍需使用无参构造，必须显式定义。

> [!question] 习题：Test Your Understanding 2 — 编写随机初始化的构造器
> 为 `Die` 类编写一个构造器，将 `faceValue` 初始化为 1~6 之间的随机整数。
>
> > [!check]-
> > `Die` 类的无参构造器可以直接完成随机初始化，核心在于把 `Math.random()` 产生的浮点数映射到 $[1, 6]$ 的整数区间。
> >
> > ```java
> > public class Die {
> >     private int faceValue;
> > 
> >     public Die() {
> >         faceValue = (int)(Math.random() * 6) + 1; // 生成 1 到 6 的随机整数
> >     }
> > }
> > ```
> >
> > **构造器主体**
> >
> > ```java
> > faceValue = (int)(Math.random() * 6) + 1;
> > ```
> >
> > - `Math.random()` 返回 $[0.0, 1.0)$ 的随机浮点数，乘以 6 后范围变为 $[0.0, 6.0)$。
> > - 强制转换为 `int` 后小数部分被截去，因此结果落在 $[0, 5]$；再加 1 后得到 $[1, 6]$。

> [!question] 习题：Test Your Understanding 3 — 带参构造器初始化 Movie 类
> 为 `Movie` 类编写构造器，通过参数初始化 `title` 和 `director` 两个实例变量。
>
> > [!check]-
> > 带参构造器的作用是把对象创建时传入的外部数据写入实例变量，从而让对象在创建瞬间就处于有效初始状态。
> >
> > ```java
> > public class Movie {
> >     private String title;
> >     private String director;
> > 
> >     public Movie(String theTitle, String theDirector) {
> >         title = theTitle;         // 初始化电影标题
> >         director = theDirector;   // 初始化导演姓名
> >     }
> > }
> > ```
> >
> > **参数声明**
> >
> > ```java
> > public Movie(String theTitle, String theDirector)
> > ```
> >
> > - 构造器接收两个 `String` 类型参数，分别对应电影标题与导演姓名。
> > - 参数名写成 `theTitle` 与 `theDirector`，可以避免与实例变量同名时产生歧义。
> >
> > **赋值语句**
> >
> > ```java
> > title = theTitle;
> > director = theDirector;
> > ```
> >
> > - 这两条语句把传入参数分别写入实例变量，完成对象初始化。

> [!note]
> 若参数名与实例变量名相同（如均为 `title`），则需使用 `this.title = title` 加以区分。`this` 关键字指代当前对象，后续章节将详细介绍。

## 3.4 `toString()` 方法

当程序需要将对象转换为**字符串**形式输出时，Java 会自动调用 `toString()` 方法来决定输出的具体内容。

`toString()` 的返回类型为 `String`，用于返回对象的字符串表示。

> [!note]
> `toString()` 是定义在 `Object` 类中的方法，所有 Java 类均继承自 `Object`，因此每个类都默认拥有 `toString()`。

以下两种情况下，Java 编译器自动调用 `toString()`：

- 对象与字符串拼接时： `"Die value: " + die1`
- 对象作为 `println()` 的参数时：  `System.out.println(die1)`

值得注意的是，`toString()` 输出格式为 `类名@哈希码`，如 `Student@1fee6fc`，通常没有实际意义。

`toString()` 默认继承 `Object` 类的实现。覆写 (*Overriding*) `toString()` 后，输出内容由程序员自定义，可返回对象有意义的状态描述。

> [!example]- 示例：未覆写 `toString()` 的输出结果
>
> 这个版本保留 `Object` 默认提供的 `toString()` 实现，因此打印对象时只能看到 `类名@哈希码` 形式的结果。
>
> ```java
> class Student {
>     int rollno;
>     String name;
>     String city;
> 
>     Student(int rollno, String name, String city) {
>         this.rollno = rollno;
>         this.name = name;
>         this.city = city;
>     }
> 
>     public static void main(String args[]) {
>         Student s1 = new Student(101, "Raj", "lucknow");      // 创建第一个学生对象
>         Student s2 = new Student(102, "Vijay", "ghaziabad");  // 创建第二个学生对象
> 
>         System.out.println(s1);  // 自动调用默认的 toString()
>         System.out.println(s2);
>     }
> }
> ```
>
> 输出：
>
> ```
> Student@1fee6fc
> Student@1eed786
> ```
>
> **对象创建**
>
> ```java
> Student s1 = new Student(101, "Raj", "lucknow");
> Student s2 = new Student(102, "Vijay", "ghaziabad");
> ```
>
> - 带参构造器在对象创建时同时初始化 `rollno`、`name` 与 `city` 三个实例变量。
>
> **打印对象**
>
> ```java
> System.out.println(s1);
> System.out.println(s2);
> ```
>
> - `println()` 遇到对象参数时会自动调用该对象的 `toString()`。
> - 由于这里没有覆写，实际调用的是 `Object` 提供的默认实现，因此输出形式为 `类名@哈希码`。

> [!example]- 示例：覆写 `toString()` 后的输出结果
>
> 为 `Student` 补上 `toString()` 后，打印对象时就可以直接得到对象状态的字符串表示。
>
> ```java
> class Student {
>     int rollno;
>     String name;
>     String city;
> 
>     Student(int rollno, String name, String city) {
>         this.rollno = rollno;
>         this.name = name;
>         this.city = city;
>     }
> 
>     public String toString() {
>         return rollno + " " + name + " " + city; // 拼接对象的主要状态
>     }
> 
>     public static void main(String args[]) {
>         Student s1 = new Student(101, "Raj", "lucknow");
>         Student s2 = new Student(102, "Vijay", "ghaziabad");
> 
>         System.out.println(s1);
>         System.out.println(s2);
>     }
> }
> ```
>
> 输出：
>
> ```
> 101 Raj lucknow
> 102 Vijay ghaziabad
> ```
>
> **`toString()` 方法体**
>
> ```java
> return rollno + " " + name + " " + city;
> ```
>
> - 方法把三个实例变量按空格顺序拼接成字符串返回，因此输出直接反映对象状态。
> - `rollno` 虽然是 `int`，但在字符串拼接表达式中会自动转换为 `String`。
>
> **打印对象**
>
> ```java
> System.out.println(s1);
> System.out.println(s2);
> ```
>
> - 这一处仍然是 `println()` 自动调用 `toString()`；区别只在于现在调用的是程序员覆写后的版本。

> [!note]
> 覆写 `toString()` 时，方法签名必须与 `Object` 类中的定义完全一致。
>
> 返回类型、方法名、参数列表（无参）均不能改变，否则构成重载而非覆写，`println` 时仍会调用 `Object` 的默认实现。

## 3.5 类的定义规范与代码审查

类的定义一般形式为：

```java
public class <类名> {
    <类体>
}
```

```java
/*
类名首字母大写，这是 Java 的命名惯例。
*/
public class ClassName {
    // 数据声明
    // 方法声明（构造器、访问器、修改器、其他方法）
}
```

- 实例数据通常以 `private` 修饰，外部无法直接访问，需通过方法间接操作。
- 并非每个类都需要 `main` 方法，`main` 方法仅出现在程序入口类中。

访问器 (*Accessor*) 与修改器 (*Mutator*) 是封装中最常见的一对方法类别。访问器也称 `getter`，用于返回某个实例变量的当前值而不修改对象状态；修改器也称 `setter`，用于接收参数并更新某个实例变量的值。两者配合 `private` 字段，是实现封装 (*Encapsulation*) 的标准方式。

```java
public class Child {
    private int age;

    // 访问器
    public int getAge() {
        return age;
    }

    // 修改器
    public void setAge(int newAge) {
        age = newAge;
    }
}
```

> [!example]- 示例：`SimpleDieGame1.java`
>
> `SimpleDieGame1.java` 演示如何复用同一个 `Die` 对象，并把两次 `roll()` 的返回值保存到局部变量中完成比较。
>
> ```java
> public class SimpleDieGame1 {
>     public static void main(String[] args) {
>         Die die = new Die();       // 创建骰子对象
>
>         int johnValue = die.roll();  // John 掷骰
>         int maryValue = die.roll();  // Mary 掷骰
>
>         System.out.println("John's facevalue is " + johnValue + ".");
>         System.out.println("Mary's facevalue is " + maryValue + ".");
>
>         if (johnValue > maryValue)
>             System.out.println("John wins the game!");
>         else if (maryValue > johnValue)
>             System.out.println("Mary wins the game!");
>         else
>             System.out.println("Tie.");
>     }
> }
> ```
>
> **实例化与掷骰**
> ```java
> Die die = new Die();
> int johnValue = die.roll();
> int maryValue = die.roll();
> ```
> - 创建一个 `Die` 对象，`faceValue` 初始化为 1。
> - 调用两次 `roll()`，每次返回新的随机面值并分别存储到 `johnValue` 与 `maryValue`。
> - 两次调用共用同一个 `die` 对象，每次调用都会更新其 `faceValue`。
>
> **判断逻辑**
> ```java
> if (johnValue > maryValue)
>     System.out.println("John wins the game!");
> else if (maryValue > johnValue)
>     System.out.println("Mary wins the game!");
> else
>     System.out.println("Tie.");
> ```
> - 三路分支覆盖所有情况：John 胜、Mary 胜、平局。
> - 使用的是局部变量 `johnValue` 与 `maryValue`，而非 `die.getFaceValue()`，因为第二次 `roll()` 已覆盖 `faceValue`。
>
> > [!warning]
> > 课件中 `SimpleDieGame1.java` 包含 `int value = die.faceValue;` 这一行，由于 `faceValue` 是 `private` 字段，此写法**无法通过编译**。正确做法是通过 `die.getFaceValue()` 访问。

> [!example]- 示例：`SimpleDieGame2.java`
>
> `SimpleDieGame2.java` 演示带参构造器、`while` 循环与 `toString()` 如何配合，持续掷骰直到再次得到面值 6。
>
> ```java
> public class SimpleDieGame2 {
>     public static void main(String[] args) {
>         Die die = new Die(6);   // 使用带参构造器，初始面值为 6
>
>         System.out.println("The current faceValue is " + die + ".");
>
>         int newValue = die.roll();
>
>         while (newValue != 6) {
>             System.out.println("The current faceValue is " + die + ".");
>             newValue = die.roll();
>         }
>
>         System.out.println("The final faceValue is " + die + ".");
>     }
> }
> ```
>
> **实例化**
> ```java
> Die die = new Die(6);
> ```
> - 调用带参构造器 `Die(int v)`，将 `faceValue` 初始化为 6。
> - 字符串拼接中 `die` 自动调用 `toString()`，输出当前面值。
>
> **循环逻辑**
> ```java
> int newValue = die.roll();
> while (newValue != 6) {
>     System.out.println("The current faceValue is " + die + ".");
>     newValue = die.roll();
> }
> ```
> - 进入循环前先掷一次，若结果不为 6 则继续循环。
> - 每次循环体内先输出当前面值，再掷骰更新 `newValue`。
> - `die` 在字符串拼接中调用 `toString()`，输出的是 `roll()` 更新后的 `faceValue`。
>
> **终止输出**
> ```java
> System.out.println("The final faceValue is " + die + ".");
> ```
> - 循环结束时 `newValue == 6`，输出最终面值。

> [!question] 习题：Test Your Understanding 4 — 为 Child 类编写访问器与修改器
> 为拥有实例变量 `age` 的 `Child` 类编写访问器与修改器。
>
> > [!check]-
> > 访问器负责读取状态，修改器负责更新状态；两者共同组成对 `private` 实例变量的标准封装接口。
> >
> > ```java
> > public class Child {
> >     private int age;
> > 
> >     public int getAge() {
> >         return age; // 返回当前年龄
> >     }
> > 
> >     public void setAge(int newAge) {
> >         age = newAge; // 用参数更新年龄
> >     }
> > }
> > ```
> >
> > **访问器**
> >
> > ```java
> > public int getAge() {
> >     return age;
> > }
> > ```
> >
> > - 返回类型写为 `int`，因为它返回的是 `age` 当前保存的整数值。
> > - 方法体只有 `return`，因此它读取状态但不改变对象状态。
> >
> > **修改器**
> >
> > ```java
> > public void setAge(int newAge) {
> >     age = newAge;
> > }
> > ```
> >
> > - 返回类型是 `void`，因为修改器的职责是更新状态而不是返回结果。
> > - 参数 `newAge` 作为新的输入值写入实例变量 `age`。

## 3.6 UML 类图

统一建模语言 (*Unified Modeling Language, UML*) 类图是一种静态结构图，用于可视化系统中类的结构与类间关系，是面向对象系统设计的标准工具。

每个类在 UML 中表示为一个矩形，分为三个区域：

```
┌─────────────────┐
│    ClassName    │  ← 类名
├─────────────────┤
│  - attribute1   │  ← 属性（实例数据）及其类型
│  - attribute2   │
├─────────────────┤
│  + method1()    │  ← 操作（方法）及其签名
│  + method2()    │
└─────────────────┘
```

属性格式：`可见性 属性名 : 类型`

操作格式：`<可见性> <方法名>(<参数列表>) : <返回类型>`

方法签名 (*Method Signature*)：方法名与参数列表的组合，用于唯一标识一个方法。

| 符号  | 含义                | 对应 Java 修饰符 |
| --- | ----------------- | ----------- |
| `+` | 公有 (*Public*)     | `public`    |
| `-` | 私有 (*Private*)    | `private`   |
| `#` | 受保护 (*Protected*) | `protected` |

#### 类间关系 (*Inter-Class Relationships*)

依赖关系 (*Dependency*) 指一个类使用另一个类的方法，在 UML 中以**虚线箭头**表示，箭头指向被使用的类。
- 例如：`RollingDice` 使用 `Die` 的方法，`RollingDice` 指向 `Die`。

继承关系 (*Inheritance*) 指子类继承父类的属性与方法，在 UML 中以**实线空心箭头**表示，箭头指向父类。
- 例如：`Cat`、`Dog`、`Bird` 继承自 `Animal`，三者均指向 `Animal`。

以 `Die` 类与 `RollingDice` 类为例：

```
┌──────────────────────────┐
│          Die             │
├──────────────────────────┤
│  - faceValue : int       │
│  - MAX : int             │
├──────────────────────────┤
│  + Die()                 │
│  + Die(v : int)          │
│  + roll() : int          │
│  + getFaceValue() : int  │
│  + setFaceValue(int)     │
│  + toString() : String   │
└──────────────────────────┘
            ↑ (虚线箭头)
┌──────────────────────────┐
│       RollingDice        │
├──────────────────────────┤
│                          │
├──────────────────────────┤
│  + main(String[]) : void │
└──────────────────────────┘
```

继承关系示例：

```
              ┌───────────┐
              │  Animal   │
              ├───────────┤
              │ - name    │
              ├───────────┤
              │ + eat()   │
              └───────────┘
              ↑     ↑    ↑
        ┌─────┘     │    └─────┐
   ┌────┴──┐   ┌────┴──┐  ┌────┴──┐
   │  Cat  │   │  Dog  │  │ Bird  │
   └───────┘   └───────┘  └───────┘
```

> [!question] 习题：Test Your Understanding 5 — 绘制动物类 UML 类图
> 设计三个动物类继承自 `Animal`，每个类给出两个状态与两个行为。
>
> > [!check]-
> > 这道题的关键是先抽取所有动物共享的共性放入父类，再把各自特有的状态与行为分配到对应子类中。
> >
> > ```
> > ┌─────────────────────────┐
> > │         Animal          │
> > ├─────────────────────────┤
> > │  - name : String        │
> > │  - age : int            │
> > ├─────────────────────────┤
> > │  + eat() : void          │
> > │  + sleep() : void        │
> > └─────────────────────────┘
> >          ↑      ↑      ↑
> >    ┌─────┘      │      └──────┐
> > ┌──┴───────┐ ┌──┴───────┐ ┌──┴───────┐
> > │   Cat    │ │   Dog    │ │   Bird   │
> > ├──────────┤ ├──────────┤ ├──────────┤
> > │-color    │ │-breed    │ │-canFly   │
> > │-indoor   │ │-trained  │ │-wingSpan │
> > ├──────────┤ ├──────────┤ ├──────────┤
> > │+purr()    │ │+fetch()   │ │+sing()    │
> > │+climb()   │ │+bark()    │ │+fly()     │
> > └──────────┘ └──────────┘ └──────────┘
> > ```
> >
> > **`Animal` 类**
> >
> > - `name` 与 `age` 是所有动物共有的状态，因此应定义在父类中。
> > - `eat()` 与 `sleep()` 是所有动物都具备的基本行为，因此也放在父类中统一声明。
> >
> > **子类的扩展**
> >
> > - `Cat` 额外包含 `color`、`indoor` 以及 `purr()`、`climb()`。
> > - `Dog` 额外包含 `breed`、`trained` 以及 `fetch()`、`bark()`。
> > - `Bird` 额外包含 `canFly`、`wingSpan` 以及 `sing()`、`fly()`。
> > - 三个子类通过继承箭头指向 `Animal`，表示它们共享父类定义的共性结构。

---
`Pre: ` [[ELEC2543 Ch.2 Expression & Java Syntax]]
`Post:` [[ELEC2543 Ch.4 Object Creation]]

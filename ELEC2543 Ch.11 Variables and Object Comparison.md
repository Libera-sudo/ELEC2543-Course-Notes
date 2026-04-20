#Y2S2 #ELEC2543
# Chapter 11. 变量与对象比较

`Pre: ` [[ELEC2543 Ch.10 Static Modifier]]
`Post:` [[ELEC2543 Ch.12 Wrapper Class]]

> [!abstract]
> 变量比较与对象比较是类型系统中数据等价性判断的基础语义问题，直接决定何为“相等”以及在何种层面上相等，属于面向对象程序设计中类型安全与语义明确性的核心议题。
>
> Java 严格区分基本类型与对象引用的比较语义：`==` 对基本类型比较值、对对象引用比较地址，而内容等价性的判断依赖 `equals` 方法的定义与覆写 (*Overriding*)。这一取向要求程序员在自定义类中显式声明“相等”的含义，而非默认沿用语言内建行为，与 C++ 通过重载 `operator==` 隐式引入结构相等、或 Python 通过 `eq` 与 `hash` 定义对象等价的方式形成对照。
>
> 本章涵盖四个核心模块：浮点数比较中的舍入误差与容差法、字符比较所依赖的 Unicode 顺序、字符串的 `equals` 与 `compareTo` 方法及词典序规则，以及对象比较中 `==` 与 `equals` 的语义差异与自定义 `equals` 方法的实现。

```table-of-contents
maxLevel: 3
```

## 11.1 浮点数比较

浮点数在计算机中以二进制近似表示，某些十进制小数无法被精确编码，导致运算结果存在微小的舍入误差 (*Rounding Error*)。直接使用 `==` 比较两个浮点数，可能因这类误差而得到错误的 `false` 结果，即使两个值在数学上应当相等。

容差法 (*Tolerance-Based Comparison*) 是处理浮点数相等性判断的标准做法：若两个浮点数之差的绝对值小于预设的容差 (*Tolerance*)，则认为它们“本质上相等”。

```java
final double TOLERANCE = 0.000001;

if (Math.abs(f1 - f2) < TOLERANCE)
    System.out.println("Essentially equal");
```

- `Math.abs(f1 - f2)` 计算两数之差的绝对值，消除正负方向的影响。
- `TOLERANCE` 声明为 `final`，其具体值根据应用场景的精度需求设定，如科学计算可能需要更小的容差。
- 该方法本质上将“精确相等”放宽为“足够接近”，是工程实践中的常规处理方式。

> [!warning]
> 直接对 `float` 或 `double` 使用 `==` 判断相等，会因舍入误差产生反直觉的 `false` 结果；浮点数的相等性判断必须使用容差法。`==` 对 `int`、`char`、`boolean` 等基本类型的比较是精确的，不存在此问题，容差比较仅适用于 `float` 与 `double`。
>
> > [!bug]
> >
> > ```java
> > double a = 0.1 + 0.2;
> > double b = 0.3;
> > System.out.println(a == b);                      // a 实际为 0.30000000000000004
> > System.out.println(Math.abs(a - b) < 0.000001);
> > ```
> >
> > ```
> > false
> > true
> > ```

> [!note]
> 容差的设定需结合数据量级与应用场景，常见策略如下：
>
> - **绝对容差**：适用于数值范围固定的领域
>   - 常规工程：`1e-6` ~ `1e-9`
>   - 金融货币：`0.01` 或 `0.001`（最小计价单位）
>   - 图形像素：`1e-3` ~ `1e-4`
>
> - **相对容差**：适用于跨数量级的比较
>   ```java
>   Math.abs(a - b) <= TOLERANCE * Math.max(Math.abs(a), Math.abs(b))
>   ```
>
> - **ULP 基准**：基于浮点表示精度
>   ```java
>   Math.abs(a - b) <= Math.ulp(a) * n  // n 为允许的误差位数
>   ```

## 11.2 字符比较

Java 的 `char` 类型基于 Unicode 字符集 (*Unicode Character Set*)。Unicode 为每个字符分配唯一的数值编码，这一编码天然定义了字符之间的大小顺序，因此可以直接对 `char` 使用关系运算符（`<`、`>`、`==` 等）进行比较。

```java
char c1 = 'A';  // Unicode 值为 65
char c2 = 'a';  // Unicode 值为 97

if (c1 < c2)
    System.out.println("'A' comes before 'a'");
```

Unicode 中部分常用字符的数值范围如下：

| 字符范围          | Unicode 值范围 |
| ------------- | ----------- |
| `'0'` – `'9'` | 48 – 57     |
| `'A'` – `'Z'` | 65 – 90     |
| `'a'` – `'z'` | 97 – 122    |

> [!note]
> 大写字母的 Unicode 值整体小于小写字母，因此 `'Z'`（90）仍小于 `'a'`（97）。这一顺序在 11.3 节中将直接决定字符串的词典序结果。
>
> Java 的 `char` 占 2 字节，支持完整的 Unicode 基本多文种平面 (*Basic Multilingual Plane*, BMP)，编码范围为 `\u0000` 到 `\uFFFF`，与 C/C++ 的单字节 `char` 不同。

## 11.3 字符串比较

`String` 是对象类型，不能使用 `==` 或 `<`、`>` 等关系运算符进行内容比较。`String` 类提供了两个专用方法处理字符串的相等性与顺序判断。

`equals` 方法判断两个字符串是否包含完全相同的字符序列，返回 `boolean`：

```java
if (name1.equals(name2))
    System.out.println("Same name");
```

`compareTo` 方法按词典序 (*Lexicographic Ordering*) 比较两个字符串的大小，返回 `int`：

```java
int result = name1.compareTo(name2);
```

- 返回 `0`：`name1` 与 `name2` 内容相同。
- 返回负值：`name1` 在词典序中排在 `name2` 之前（`name1` 较小）。
- 返回正值：`name1` 在词典序中排在 `name2` 之后（`name1` 较大）。

词典序的比较规则基于各字符的 Unicode 值，逐字符从左到右依次比较，直到出现不同字符或某字符串结束为止：

- 大写字母的 Unicode 值整体小于小写字母，因此任意以小写字母（`'a'` – `'z'`）开头的字符串，其词典序都高于任何以大写字母（`'A'` – `'Z'`）开头的字符串。
	- 比如：`"Generation"` 排在 `"fun"` 之前。
- 若一个字符串是另一个字符串的前缀，则较短的字符串排在前面。
	- 比如：`"book"` 排在 `"bookcase"` 之前。

词典序的典型用法如下：

```java
if (name1.compareTo(name2) < 0)
    System.out.println(name1 + " comes first");
else if (name1.compareTo(name2) == 0)
    System.out.println("Same name");
else
    System.out.println(name2 + " comes first");
```

> [!note]
> `compareTo` 返回的具体整数值为两字符串第一个不同字符的 Unicode 差值（`name1.charAt(i) - name2.charAt(i)`），而非固定的 $-1$ 或 $1$。实际使用中只需判断其正负与零，不应依赖具体数值。

> [!warning]
> `==` 用于字符串时比较的是引用地址，而非字符串内容。两个内容相同但独立创建的 `String` 对象，`==` 结果为 `false`，`equals` 结果为 `true`。字符串的内容比较必须使用 `equals`，这也是 Java 初学者最常见的错误之一。
>
> > [!bug]
> >
> > ```java
> > String s1 = new String("hello");
> > String s2 = new String("hello");
> > System.out.println(s1 == s2);        // 比较的是引用地址
> > System.out.println(s1.equals(s2));   // 比较的是字符序列
> > ```
> >
> > ```
> > false
> > true
> > ```

## 11.4 对象比较

`==` 运算符作用于对象引用时，比较的是两个引用是否指向内存中的同一个对象，即检测两者是否互为别名 (*Alias*)。即使两个对象的所有实例变量值完全相同，只要它们是独立创建的，`==` 结果仍为 `false`。

```java
DieWithEquals die1 = new DieWithEquals();
DieWithEquals die2 = new DieWithEquals();

System.out.println(die1 == die2);  // 两个独立对象，地址不同
```

`equals` 方法继承自 `Object` 类，其默认实现与 `==` 语义相同——仅比较引用地址，不比较对象内容。若需要按内容判断相等性，必须在自定义类中覆写 `equals` 方法，明确定义“相等”的条件。`String` 类已覆写了 `equals`，使其比较字符序列内容而非引用地址，这正是 11.3 节中 `name1.equals(name2)` 能正确判断字符串内容的原因。

> [!note]
> `equals` 的默认行为来自所有类的根父类 `Object`，其实现等价于 `return (this == obj)`。若不覆写，调用 `equals` 与直接使用 `==` 效果完全相同，无法区分内容相同但独立创建的两个对象。

> [!example]- 示例：`DieWithEquals.java` / `DieEqualsDemo.java`
>
> `DieWithEquals.java`、`DieEqualsDemo.java` 共同演示 `equals` 被覆写后 `==` 与 `equals` 在对象比较中的语义差异。`DieWithEquals.java` 在原有骰子类基础上覆写 `equals`，使其按点数判断相等；`DieEqualsDemo.java` 为主程序，创建两个独立的 `DieWithEquals` 对象，并分别用 `==` 与 `equals` 比较。
>
> > [!info]- UML 框图
> >
> > ```
> > ┌──────────────────┐
> > │ DieEqualsDemo    │
> > │  + main          │
> > └────────┬─────────┘
> >          │ 调用
> >          ▼
> > ┌──────────────────────────────────────┐
> > │ DieWithEquals                        │
> > │  - MAX : int                         │
> > │  - faceValue : int                   │
> > │  + DieWithEquals()                   │
> > │  + roll(): int                       │
> > │  + setFaceValue(int): void           │
> > │  + getFaceValue(): int               │
> > │  + toString(): String                │
> > │  + equals(DieWithEquals): boolean    │
> > └──────────────────────────────────────┘
> > ```
>
> ```java
> // DieWithEquals.java —— 覆写 equals 以按点数判断相等的骰子类
> public class DieWithEquals {
>     private final int MAX = 6;  // 最大点数
>     private int faceValue;      // 当前点数
>
>     // 构造器：初始点数为 1
>     public DieWithEquals() {
>         faceValue = 1;
>     }
>
>     // 掷骰子，返回随机点数
>     public int roll() {
>         faceValue = (int)(Math.random() * MAX) + 1;
>         return faceValue;
>     }
>
>     // 点数修改器
>     public void setFaceValue(int value) {
>         faceValue = value;
>     }
>
>     // 点数访问器
>     public int getFaceValue() {
>         return faceValue;
>     }
>
>     // 返回点数的字符串表示
>     public String toString() {
>         return Integer.toString(faceValue);
>     }
>
>     // 覆写 equals：两个骰子点数相同则视为相等
>     public boolean equals(DieWithEquals d) {
>         return d.faceValue == faceValue;
>     }
> }
> ```
>
> ```java
> // DieEqualsDemo.java —— 对比 == 与 equals 在对象比较中的行为差异
> public class DieEqualsDemo {
>     public static void main(String[] args) {
>         DieWithEquals die1, die2;
>
>         die1 = new DieWithEquals();  // faceValue = 1
>         die2 = new DieWithEquals();  // faceValue = 1
>
>         // == 比较引用地址
>         if (die1 == die2)
>             System.out.println("die1 == die2");
>         else
>             System.out.println("die1 =/= die2");
>
>         // equals 比较点数内容
>         if (die1.equals(die2))
>             System.out.println("die1 is equal to die2");
>         else
>             System.out.println("die1 is not equal to die2");
>     }
> }
> ```
>
> 输出：
>
> ```
> die1 =/= die2
> die1 is equal to die2
> ```
>
> **`==` 的引用比较**
>
> ```java
> die1 = new DieWithEquals();
> die2 = new DieWithEquals();
> if (die1 == die2)
> ```
>
> - `new` 两次分别在堆上创建两个独立的 `DieWithEquals` 对象，`die1` 与 `die2` 存储的是不同的内存地址。
> - `==` 比较这两个地址，结果为 `false`，输出 `die1 =/= die2`。
>
> **覆写后的 `equals` 比较**
>
> ```java
> public boolean equals(DieWithEquals d) {
>     return d.faceValue == faceValue;
> }
> ```
>
> - 两个对象均由无参构造器创建，`faceValue` 均初始化为 `1`，`equals` 比较两者的 `faceValue` 字段，`1 == 1` 为 `true`，输出 `die1 is equal to die2`。
> - `d.faceValue` 直接访问了另一个 `DieWithEquals` 对象的 `private` 字段，这在同一类的方法内部是合法的——Java 的访问控制以类为单位，而非以对象为单位。

## 11.5 自定义 `equals` 方法

为自定义类覆写 `equals` 方法时，需要逐一比较所有决定“相等”的实例变量。基本类型字段使用 `==` 比较，对象类型字段（如 `String`）使用其自身的 `equals` 方法比较。

> [!question] 习题：为 `Student` 类定义 `equals` 方法
> 为 `Student` 类定义 `equals` 方法，使 `s1.equals(s2)` 在 `lastname`、`firstname`、`age` 三个字段均相同时返回 `true`，否则返回 `false`。
>
> ```java
> // Student.java 骨架
> public class Student {
>     private String lastname;
>     private String firstname;
>     private int age;
>
>     public Student(String lastname, String firstname, int age) {
>         this.lastname  = lastname;
>         this.firstname = firstname;
>         this.age       = age;
>     }
>
>     // TODO: 定义 equals 方法
> }
> ```
>
> > [!check]-
> >
> > `Student.java`、`StudentTester.java` 共同演示自定义 `equals` 的完整实现与测试。`Student.java` 覆写 `equals`，对三个字段按各自类型选择比较方式；`StudentTester.java` 为主程序，构造两个仅 `firstname` 不同的 `Student` 对象，验证 `equals` 能区分出内容差异。
> >
> > > [!info]- UML 框图
> > >
> > > ```
> > > ┌──────────────────┐
> > > │ StudentTester    │
> > > │  + main          │
> > > └────────┬─────────┘
> > >          │ 调用
> > >          ▼
> > > ┌──────────────────────────────────┐
> > > │ Student                          │
> > > │  - lastname  : String            │
> > > │  - firstname : String            │
> > > │  - age       : int               │
> > > │  + Student(String, String, int)  │
> > > │  + equals(Student): boolean      │
> > > └──────────────────────────────────┘
> > > ```
> >
> > ```java
> > // Student.java —— 含姓名与年龄的学生类，覆写 equals 进行内容比较
> > public class Student {
> >     private String lastname;
> >     private String firstname;
> >     private int age;
> >
> >     // 构造器：初始化三个字段
> >     public Student(String lastname, String firstname, int age) {
> >         this.lastname  = lastname;
> >         this.firstname = firstname;
> >         this.age       = age;
> >     }
> >
> >     // 覆写 equals：三个字段均相同则视为相等
> >     public boolean equals(Student s) {
> >         if (lastname.equals(s.lastname) &&
> >             firstname.equals(s.firstname) &&
> >             s.age == age)
> >             return true;
> >         else
> >             return false;
> >     }
> > }
> > ```
> >
> > ```java
> > // StudentTester.java —— 创建两个 Student 对象并测试 equals
> > public class StudentTester {
> >     public static void main(String[] args) {
> >         Student s1 = new Student("Howly", "David", 20);
> >         Student s2 = new Student("Howly", "John",  20);
> >
> >         System.out.print("The two students are ");
> >         if (s1.equals(s2) == false)
> >             System.out.print("not ");
> >         System.out.println("the same.");
> >     }
> > }
> > ```
> >
> > 输出：
> >
> > ```
> > The two students are not the same.
> > ```
> >
> > **`String` 字段的比较**
> >
> > ```java
> > lastname.equals(s.lastname) && firstname.equals(s.firstname)
> > ```
> >
> > - `lastname` 与 `firstname` 是 `String` 对象，必须使用 `equals` 方法比较字符序列内容，不能使用 `==`。
> > - `==` 比较的是引用地址，只有在字符串驻留 (*String Interning*) 机制恰好令两者共享同一引用时才偶然返回 `true`，不构成通用判据。
> >
> > **`int` 字段的比较**
> >
> > ```java
> > s.age == age
> > ```
> >
> > - `age` 是基本类型 `int`，直接使用 `==` 比较值，结果精确可靠。
> >
> > **测试结果分析**
> >
> > ```java
> > Student s1 = new Student("Howly", "David", 20);
> > Student s2 = new Student("Howly", "John",  20);
> > ```
> >
> > - `s1` 与 `s2` 的 `lastname` 均为 `"Howly"`、`age` 均为 `20`，但 `firstname` 分别为 `"David"` 与 `"John"`。
> > - `firstname.equals(s.firstname)` 返回 `false`，整个条件为 `false`，`equals` 返回 `false`，输出 `The two students are not the same.`。

> [!warning]
> 课件原版 `Student.java` 使用 `s.lastname == lastname` 比较字符串。字符串字面量在编译期会被放入常量池，因此字面量创建的两个字符串常常共享同一引用，`==` 恰好返回 `true`；但一旦字符串通过 `new String(...)` 或用户输入创建，`==` 将返回 `false`，导致逻辑错误。正确写法应始终使用 `equals` 比较字符串内容。
>
> > [!note]
> > 字符串驻留：字符串字面量在编译期被放入常量池，相同字面量共享同一引用；`new String("...")` 则强制在堆上创建新对象，不进入常量池，因而不共享引用。

---
`Pre: ` [[ELEC2543 Ch.10 Static Modifier]]
`Post:` [[ELEC2543 Ch.12 Wrapper Class]]

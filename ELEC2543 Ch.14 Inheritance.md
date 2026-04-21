#Y2S2 #ELEC2543
# Chapter 14. 继承

`Pre: ` [[ELEC2543 Ch.13 Interface]]
`Post:` [[ELEC2543 Ch.15 Polymorphism]]

> [!abstract]
> 继承 (*Inheritance*) 是面向对象程序设计中实现代码复用与类型层次化组织的核心机制，属于 Java 类型系统设计的基础支柱之一。
>
> Java 采用单继承模型：每个类至多有一个直接父类，通过 `extends` 关键字建立派生关系。子类自动获得父类的方法与数据，同时可通过方法覆写 (*Method Overriding*) 定制行为。这一设计避免了多重类继承中的命名冲突问题，同时以接口机制补偿多重行为约定的需求。与 Python 支持多重继承不同，Java 在编译期严格约束继承链，使类型关系更为清晰可预测。
>
> 本章涵盖子类的创建语法与 `extends` 关键字、`protected` 访问修饰符、`super` 引用的两种用途、方法覆写与变量遮蔽、类层次结构与 `Object` 类、抽象类与抽象方法、接口层次结构，以及继承中的可见性规则与面向继承的设计原则。

```table-of-contents
maxLevel: 3
```

## 14.1 创建子类

继承允许程序员从一个已有类派生出新类，新类自动获得父类定义的所有方法与实例变量，并可在此基础上添加新成员或修改已有行为。这一机制的核心价值在于软件复用：已经过设计、实现与测试的代码可以直接被新类利用，无需重复编写。

派生关系通过 `extends` 关键字在类头中声明：

```java
public class Car extends Vehicle
{
    // 类内容
}
```

- 被继承的类称为父类 (*Parent Class*) / 超类 (*Superclass*) / 基类 (*Base Class*)。
- 派生出的新类称为派生类 (*Derived Class*) / 子类 (*Subclass*)。
- 继承关系在 UML 类图中以实线空心三角箭头表示，箭头指向父类。
```
        +--------------------------+
        |         Book             |
	    +--------------------------+
	    |     # pages: int         |
	    +--------------------------+
	    | + setPages (int) : void  |
	    | + getPages () : int      |
	    +--------------------------+
	               △
	                │
	                │ extends
	                │
	  +-------------------------------+
	  |        Dictionary             |
	  +-------------------------------+
	  |     - definitions: int        |
	  +-------------------------------+
	  | + computeRatio () : double    |
	  | + setDefinitions (int) : void |
	  | + getDefinitions () : int     |
	  +-------------------------------+
```

正确的继承应当建立 is-a 关系 (*Is-A Relationship*)：子类是父类的一个更具体的版本。例如，`Dictionary` 是一种 `Book`，`Car` 是一种 `Vehicle`。若无法用"子类 is-a 父类"这一句话自然描述两个类的关系，则该继承设计通常存在问题。

#### `protected` 修饰符 (*Protected Modifier*)

父类中声明为 `private` 的成员无法在子类中直接按名引用；声明为 `public` 的成员虽可在子类中访问，但将实例变量声明为 `public` 违反封装原则。`protected` 修饰符提供了介于两者之间的访问级别：

- `protected` 成员可在子类中直接按名访问。
- `protected` 成员对同一包 (*Package*) 内的其他类也可见。
- 在 UML 类图中，`protected` 成员以 `#` 符号标注。
```
        +--------------------------+
        |         Book             |
	    +--------------------------+
	    |     # pages: int         |   <-- protected: 子类与同包类可见
	    +--------------------------+
	    | + setPages (int) : void  |
	    | + getPages () : int      |
	    +--------------------------+
	               △
	                │
	                │ extends
	                │
	  +-------------------------------+
	  |        Dictionary             |
	  +-------------------------------+
	  |     - definitions: int        |<-- protected: 子类与同包类可见
	  +-------------------------------+
	  | + computeRatio () : double    |
	  | + setDefinitions (int) : void |<-- 可直接访问继承的 #pages
	  | + getDefinitions () : int     |
	  +-------------------------------+
```

```java
public class Book
{
    protected int pages = 1500;  // 子类可直接访问

    public void setPages(int numPages) { pages = numPages; }
    public int getPages() { return pages; }
}
```

`Dictionary` 继承了 `Book` 的 `pages` 字段（`protected`）与 `setPages`、`getPages` 方法（`public`），同时新增了自己的 `definitions` 字段与相关方法。

```java
public class Dictionary extends Book
{
    private int definitions = 52500;

    public double computeRatio()
    {
        return (double) definitions / pages;  // 直接访问父类 protected 字段
    }

    public void setDefinitions(int numDefinitions) { definitions = numDefinitions; }
    public int getDefinitions() { return definitions; }
}
```

`computeRatio()` 中直接使用 `pages`，无需通过 `getPages()` 间接访问，这正是 `protected` 的设计意图。

> [!note]
> `protected` 的可见范围比 `private` 宽，但比 `public` 窄。在实践中，将实例变量声明为 `protected` 仍会削弱封装性——子类可以不经过父类的访问器/修改器直接修改父类数据。更严格的做法是将父类字段保持 `private`，子类通过继承的公开方法间接操作，这将在 14.4 节中进一步说明。

> [!example]- 示例：`Words.java` / `Book.java` / `Dictionary.java`
> 演示子类继承父类方法与 `protected` 字段的基本用法。
>
> ```java
> // Book.java —— 父类，表示一本书
> public class Book
> {
>     protected int pages = 1500;  // protected：子类可直接访问
>
>     // 页数修改器
>     public void setPages(int numPages)
>     {
>         pages = numPages;
>     }
>
>     // 页数访问器
>     public int getPages()
>     {
>         return pages;
>     }
> }
> ```
>
> ```java
> // Dictionary.java —— 子类，表示一本字典
> public class Dictionary extends Book
> {
>     private int definitions = 52500;  // 子类自有字段
>
>     // 计算每页定义数，直接使用继承的 protected 字段 pages
>     public double computeRatio()
>     {
>         return (double) definitions / pages;
>     }
>
>     // 定义数修改器
>     public void setDefinitions(int numDefinitions)
>     {
>         definitions = numDefinitions;
>     }
>
>     // 定义数访问器
>     public int getDefinitions()
>     {
>         return definitions;
>     }
> }
> ```
>
> ```java
> // Words.java —— 驱动类，实例化子类并调用继承与自有方法
> public class Words
> {
>     public static void main(String[] args)
>     {
>         Dictionary webster = new Dictionary();
>
>         // getPages() 继承自 Book
>         System.out.println("Number of pages: " + webster.getPages());
>         // getDefinitions() 为 Dictionary 自有方法
>         System.out.println("Number of definitions: " + webster.getDefinitions());
>         // computeRatio() 为 Dictionary 自有方法
>         System.out.println("Definitions per page: " + webster.computeRatio());
>     }
> }
> ```
>
> 输出：
> ```
> Number of pages: 1500
> Number of definitions: 52500
> Definitions per page: 35.0
> ```
>
> **继承方法的调用**
> ```java
> webster.getPages()
> ```
> - `getPages()` 定义在 `Book` 中，`Dictionary` 未重新定义该方法，因此直接继承并使用父类版本。
> - `webster` 是 `Dictionary` 对象，但可以调用所有继承自 `Book` 的 `public` 方法。
>
> **`protected` 字段的直接访问**
> ```java
> return (double) definitions / pages;
> ```
> - `pages` 是 `Book` 中声明的 `protected` 字段，`Dictionary` 作为子类可在方法体内直接按名引用，无需通过 `getPages()`。
> - `definitions` 是 `Dictionary` 自有的 `private` 字段，仅在 `Dictionary` 内部可见。
>
> **`computeRatio()` 的类型转换**
> ```java
> return (double) definitions / pages;
> ```
> - `definitions` 与 `pages` 均为 `int`，若不强制转换，整数除法将截断小数部分。
> - `(double) definitions` 将被除数转为 `double`，使整个除法以浮点数运算，得到 `35.0` 而非 `35`。

> [!note]
> 课件提供的 `Dictionary.java` 源文件中，无参构造器版本（`Dictionary()`）与带参构造器版本（`Dictionary(int, int)`）共存，且带参版本注释掉了 `super(numPages)` 调用。
>
> > [!quote]
> > 关于构造器与 `super` 的完整用法，将在 14.3 节中通过 `Book2` / `Dictionary2` 示例详细说明。

## 14.2 方法覆写

方法覆写 (*Method Overriding*) 是指子类定义一个与父类方法具有完全相同签名的方法，以替换父类的实现。运行时，JVM 根据实际对象的类型决定调用哪个版本的方法——若对象是子类实例，则调用子类的覆写版本。

覆写方法时，子类版本的可见性不能低于父类版本。父类方法为 `public`，子类覆写版本也必须为 `public`。

在覆写方法内部，可以通过 `super.methodName()` 显式调用父类的被覆写版本，从而在子类行为的基础上复用父类逻辑：

```java
public class Advice extends Thought
{
    public void message()
    {
        System.out.println("Warning: Dates in calendar are closer than they appear.");
        System.out.println();
        super.message();  // 显式调用父类版本
    }
}
```

`super` 引用在此处的作用是绕过子类的覆写，直接访问父类中定义的方法体。若不使用 `super`，在 `Advice.message()` 内部直接调用 `message()` 将造成无限递归。
若一个方法声明为 `final`，则任何子类均**不能覆写**该方法：

```java
public final void criticalMethod() { ... }
```

变量遮蔽 (*Shadowing Variables*) 是将覆写概念应用于实例变量的情形：子类声明一个与父类同名的变量，子类中该名称将指向子类自己的变量，父类的同名变量被遮蔽。这种做法会导致代码语义混乱，应当避免。

> [!note]
> 覆写 (*Overriding*) 与重载 (*Overloading*) 是两个不同的概念，容易混淆：
>
> | | 覆写 | 重载 |
> |---|---|---|
> | 涉及类 | 父类与子类各一个方法 | 同一类中多个方法 |
> | 方法签名 | 完全相同 | 方法名相同，参数列表不同 |
> | 目的 | 为不同对象类型定义相同操作的不同行为 | 为不同参数类型定义相同操作的不同实现 |
> | 决定时机 | 运行时（动态绑定） | 编译时（静态绑定） |

> [!example]- 示例：`Messages.java` / `Thought.java` / `Advice.java`
> 演示子类覆写父类方法，以及通过 `super` 在覆写方法中显式调用父类版本。
>
> ```java
> // Thought.java —— 父类，定义 message() 方法
> public class Thought
> {
>     // 输出一条消息
>     public void message()
>     {
>         System.out.println("I feel like I'm diagonally parked in a parallel universe.");
>         System.out.println();
>     }
> }
> ```
>
> ```java
> // Advice.java —— 子类，覆写 message() 并通过 super 调用父类版本
> public class Advice extends Thought
> {
>     // 覆写父类的 message()，输出子类消息后再调用父类版本
>     public void message()
>     {
>         System.out.println("Warning: Dates in calendar are closer than they appear.");
>         System.out.println();
>         super.message();  // 显式调用 Thought.message()
>     }
> }
> ```
>
> ```java
> // Messages.java —— 驱动类，分别调用父类与子类对象的 message()
> public class Messages
> {
>     public static void main(String[] args)
>     {
>         Thought parked = new Thought();  // 父类对象
>         Advice dates = new Advice();     // 子类对象
>
>         parked.message();   // 调用 Thought.message()
>         dates.message();    // 调用 Advice.message()（覆写版本）
>     }
> }
> ```
>
> 输出：
> ```
> I feel like I'm diagonally parked in a parallel universe.
>
> Warning: Dates in calendar are closer than they appear.
>
> I feel like I'm diagonally parked in a parallel universe.
>
> ```
>
> **`parked.message()` 的调用**
> ```java
> Thought parked = new Thought();
> parked.message();
> ```
> - `parked` 的实际类型为 `Thought`，调用 `Thought.message()`，输出第一行消息。
>
> **`dates.message()` 的调用**
> ```java
> Advice dates = new Advice();
> dates.message();
> ```
> - `dates` 的实际类型为 `Advice`，JVM 在运行时确定调用 `Advice.message()`（覆写版本）。
> - `Advice.message()` 首先输出自己的消息，随后通过 `super.message()` 调用 `Thought.message()`，输出父类消息。
> - 因此 `dates.message()` 共产生两段输出。
>
> **`super.message()` 的作用**
> ```java
> super.message();
> ```
> - 若此处写 `message()` 而不加 `super`，将递归调用 `Advice.message()` 自身，导致栈溢出。
> - `super.message()` 明确指定调用父类 `Thought` 中定义的版本，绕过子类的覆写。

> [!note]
> 在子类覆写父类方法时，建议显式添加 `@Override` 注解。它会强制编译器检查被标记的方法签名是否与父类中被覆写的方法完全一致，并向代码阅读者表明该方法是有意替换父类行为。
>
> 例如，若将 `message` 误写为 `messages`，未加 `@Override` 时编译器会认为这是一个新方法；添加 `@Override` 后，编译器会提示 "method does not override or implement a method from a supertype"，及时暴露错误。

> [!question] 习题：判断正误
> 判断以下语句的真假。
>
> - 子类可以定义与父类方法同名的方法。
> - 子类可以覆写父类构造器。
> - 子类不能覆写父类的 `final` 方法。
> - 子类覆写父类方法通常被认为是糟糕设计。
> - 子类可以定义与父类变量同名的变量。
>
> > [!check]-
> > 第一项为真，子类可以定义与父类同名的方法；若签名相同，则构成覆写。
> >
> > 第二项为假，构造器不能被继承，因此也不能被覆写。
> >
> > 第三项为真，`final` 方法禁止子类覆写。
> >
> > 第四项为假，覆写本身不是坏设计；在子类需要定制父类行为时，覆写是正常机制。
> >
> > 第五项为真，但不建议这样做，因为子类同名变量会遮蔽父类变量，使代码语义变得混乱。

## 14.3 类层次结构

类层次结构 (*Class Hierarchy*) 由继承关系的链式延伸形成，一个子类可以同时作为另一个类的父类，从而构成多层的树状结构。层次结构中同一父类的多个子类互称为兄弟类 (*Siblings*)。

继承具有传递性，子类不仅继承直接父类的成员，也继承所有祖先类的成员。设计类层次结构时，应将各类共有的特征尽量上移至层次结构的高层，避免在多个子类中重复定义相同的成员。

#### `super` 引用与构造器 (*super Reference and Constructor*)

构造器**无法被继承**，即使父类构造器声明为 `public`，子类也无法直接继承并使用它。

通过调用父类构造器，子类构造器得以初始化对象中属于父类的字段。调用语法为：

```java
super(<参数列表>);
```

该语句必须作为子类构造器的**第一条**语句：

```java
public class Dictionary2 extends Book2
{
    private int definitions;

    public Dictionary2(int numPages, int numDefinitions)
    {
        super(numPages);              // 调用 Book2(int) 构造器，初始化 pages
        definitions = numDefinitions; // 初始化子类自有字段
    }
}
```

若子类构造器未显式调用 `super(...)`，编译器会自动插入对父类无参构造器 `super()` 的调用。若父类没有无参构造器，则编译报错。

> [!note]
> `super(...)` 调用父类构造器与 `super.method()` 调用父类方法是 `super` 的两种不同用法：
> - `super(参数列表)`：只能出现在子类构造器的第一行，用于调用父类构造器。
> - `super.方法名(参数列表)`：可出现在子类的任意方法中，用于调用被覆写的父类方法。

> [!example]- 示例：`Words2.java` / `Book2.java` / `Dictionary2.java`
> 演示子类通过 `super(...)` 调用父类构造器，完成对象中父类部分的初始化。
>
> ```java
> // Book2.java —— 父类，通过构造器接收页数参数
> public class Book2
> {
>     protected int pages;  // 不再使用字段初始化器，由构造器赋值
>
>     // 构造器：初始化页数
>     public Book2(int numPages)
>     {
>         pages = numPages;
>     }
>
>     // 页数修改器
>     public void setPages(int numPages) { pages = numPages; }
>
>     // 页数访问器
>     public int getPages() { return pages; }
> }
> ```
>
> ```java
> // Dictionary2.java —— 子类，通过 super() 调用父类构造器
> public class Dictionary2 extends Book2
> {
>     private int definitions;
>
>     // 构造器：先调用父类构造器初始化 pages，再初始化自有字段
>     public Dictionary2(int numPages, int numDefinitions)
>     {
>         super(numPages);              // 调用 Book2(int numPages)
>         definitions = numDefinitions;
>     }
>
>     // 计算每页定义数
>     public double computeRatio()
>     {
>         return (double) definitions / pages;
>     }
>
>     // 定义数修改器
>     public void setDefinitions(int numDefinitions) { definitions = numDefinitions; }
>
>     // 定义数访问器
>     public int getDefinitions() { return definitions; }
> }
> ```
>
> ```java
> // Words2.java —— 驱动类，通过带参构造器创建 Dictionary2 对象
> public class Words2
> {
>     public static void main(String[] args)
>     {
>         Dictionary2 webster = new Dictionary2(1500, 52500);
>
>         System.out.println("Number of pages: " + webster.getPages());
>         System.out.println("Number of definitions: " + webster.getDefinitions());
>         System.out.println("Definitions per page: " + webster.computeRatio());
>     }
> }
> ```
>
> 输出：
> ```
> Number of pages: 1500
> Number of definitions: 52500
> Definitions per page: 35.0
> ```
>
> **`super(numPages)` 的执行**
> ```java
> public Dictionary2(int numPages, int numDefinitions)
> {
>     super(numPages);
>     definitions = numDefinitions;
> }
> ```
> - `super(numPages)` 必须是构造器的第一条语句，调用 `Book2(int numPages)`，将 `pages` 初始化为传入值。
> - 随后 `definitions = numDefinitions` 初始化子类自有字段。
> - 若省略 `super(numPages)`，编译器将尝试调用 `Book2()` 无参构造器，但 `Book2` 未定义无参构造器，编译报错。

#### `Object` 类 (*Object Class*)

`Object` 是 `java.lang` 包中定义的特殊类，是所有 Java 类层次结构的根。任何未显式声明父类的类，编译器都将其视为 `Object` 的直接子类。因此，Java 中所有类都直接或间接继承自 `Object`。

`Object` 类定义了若干被所有类继承的方法，其中最常用的三个为：

```java
String toString()          // 返回对象的字符串表示
boolean equals(Object obj) // 判断两个对象是否相等
Object clone()             // 创建并返回对象的副本
```

- `toString` 的默认实现返回类名加哈希码（如 `Dictionary@1b6d3586`）；在自定义类中覆写 `toString` 可提供有意义的字符串表示，这正是此前各章节中定义 `toString` 的本质——覆写继承自 `Object` 的版本。
- `equals` 的默认实现等价于 `==`，仅比较引用地址；覆写后可按内容判断相等性。

#### 抽象类 (*Abstract Class*)

抽象类是类层次结构中代表通用概念的占位符，其本身过于抽象而无法被实例化。抽象类通过在类头加 `abstract` 修饰符声明：

```java
public abstract class Product
{
    // 类内容
}
```

抽象类通常包含抽象方法：只有方法头、没有方法体的方法声明。

每个抽象方法必须显式加 `abstract` 修饰符：

```java
public abstract int charge(int base);
```

抽象类与接口的关键区别在于：

- 抽象类可以同时包含抽象方法与具有完整实现的非抽象方法，还可以包含实例变量。
- 接口中所有方法均为抽象方法，且不能包含实例变量。

抽象方法不能声明为 `final` 或 `static` ，`final` 禁止覆写而抽象方法要求被覆写，两者语义矛盾；`static` 方法属于类而非对象，无法参与基于对象类型的动态绑定。

抽象类的子类**必须**覆写父类中所有抽象方法，否则该子类本身也将被视为抽象类，同样无法实例化。

> [!note]
> 一个类声明为 `abstract` 并不要求它必须包含抽象方法，仅加 `abstract` 修饰符即可阻止该类被实例化。
>
> 这在某些设计场景中有用：父类本身逻辑完整，但设计上不应直接创建其对象，只允许通过子类使用。

> [!example]- 示例：`HKUPerson.java` / `Student.java` / `Staff.java` / `TestHKU.java`
> 演示抽象类定义通用结构与抽象方法，子类各自提供具体实现，并通过 `super` 复用父类的构造器与 `toString`。
>
> ```java
> // HKUPerson.java —— 抽象父类，定义姓名、身份证号与抽象收费方法
> abstract public class HKUPerson
> {
>     protected String name, hkid;  // 子类可直接访问
>
>     // 构造器：初始化姓名与身份证号
>     public HKUPerson(String name, String hkid)
>     {
>         this.name = name;
>         this.hkid = hkid;
>     }
>
>     // 返回姓名与身份证号的字符串表示
>     public String toString()
>     {
>         return name + " " + hkid;
>     }
>
>     // 抽象方法：收费计算，由子类实现
>     public abstract int charge(int base);
> }
> ```
>
> ```java
> // Student.java —— 子类，实现 charge() 为 base*2
> public class Student extends HKUPerson
> {
>     private String studID;
>
>     // 构造器：调用父类构造器初始化姓名与身份证号，再初始化学号
>     public Student(String name, String hkid, String studID)
>     {
>         super(name, hkid);
>         this.studID = studID;
>     }
>
>     // 覆写 toString：在父类结果后追加学号
>     public String toString()
>     {
>         String result = super.toString();  // 调用 HKUPerson.toString()
>         return result + " " + studID;
>     }
>
>     // 实现抽象方法：学生收费为 base 的 2 倍
>     public int charge(int base) { return base * 2; }
> }
> ```
>
> ```java
> // Staff.java —— 子类，实现 charge() 为 base*3
> public class Staff extends HKUPerson
> {
>     private String staffID;
>
>     // 构造器：调用父类构造器，再初始化员工编号
>     public Staff(String name, String hkid, String staffID)
>     {
>         super(name, hkid);
>         this.staffID = staffID;
>     }
>
>     // 覆写 toString：在父类结果后追加员工编号
>     public String toString()
>     {
>         String result = super.toString();
>         return result + " " + staffID;
>     }
>
>     // 实现抽象方法：员工收费为 base 的 3 倍
>     public int charge(int base) { return base * 3; }
> }
> ```
>
> ```java
> // TestHKU.java —— 驱动类，测试多态调用 charge() 与 toString()
> public class TestHKU
> {
>     public static void main(String[] args)
>     {
>         Student stud = new Student("STUDENT", "Z123456", "2009234567");
>         Staff staff  = new Staff("STAFF", "A654321", "12345");
>
>         System.out.println("Charge of " + stud  + ":" + stud.charge(50));
>         System.out.println("Charge of " + staff + ":" + staff.charge(50));
>     }
> }
> ```
>
> 输出：
> ```
> Charge of STUDENT Z123456 2009234567:100
> Charge of STAFF A654321 12345:150
> ```
>
> **抽象方法的多态调用**
> ```java
> stud.charge(50)   // 调用 Student.charge()，返回 50*2 = 100
> staff.charge(50)  // 调用 Staff.charge()，返回 50*3 = 150
> ```
> - `charge` 在 `HKUPerson` 中声明为抽象方法，`Student` 与 `Staff` 各自提供不同实现。
> - 运行时根据对象的实际类型决定调用哪个版本，体现多态性。
>
> **`super.toString()` 的复用**
> ```java
> public String toString()
> {
>     String result = super.toString();  // 获取 "STUDENT Z123456"
>     return result + " " + studID;      // 追加 "2009234567"
> }
> ```
> - `super.toString()` 调用 `HKUPerson.toString()`，返回 `name + " " + hkid`。
> - 子类在此基础上追加自有字段，避免重复编写父类已有的逻辑。
>
> **字符串拼接中的隐式 `toString()` 调用**
> ```java
> "Charge of " + stud + ":"
> ```
> - `+` 运算符与字符串拼接时，若操作数为对象，Java 自动调用该对象的 `toString()` 方法。
> - 此处调用的是 `Student.toString()`（覆写版本），而非 `HKUPerson.toString()`。

> [!question] 习题：`Object` 类与抽象类
> 回答以下两个问题。
>
> - `Object` 类中定义了哪些常见方法？
> - 什么是抽象类？
>
> > [!check]-
> > `Object` 类中常见方法包括 `String toString()`、`boolean equals(Object obj)` 与 `Object clone()`。
> >
> > 抽象类是类层次结构中的占位类，用于表示一个过于通用而不应直接实例化的概念。它可以集中定义派生类共有的结构与行为，并要求子类实现其中的抽象方法。

#### 接口层次结构 (*Interface Hierarchy*)

继承机制同样适用于接口：

- 一个接口可以通过 `extends` 从另一个接口派生，子接口继承父接口的所有抽象方法。
- 实现子接口的类必须提供子接口与所有父接口中全部方法的实现。

与类层次结构不同，一个接口可以同时继承多个父接口：

```java
interface Calculator extends Add_Sub, Mul_Div
{
    public void printResult(double result);
}
```

实现 `Calculator` 的类必须实现 `Add_Sub`、`Mul_Div` 与 `Calculator` 三个接口中的**全部方法**。

类层次结构与接口层次结构**相互独立**，不存在重叠。

> [!note]
> Java 不允许类多重继承，但允许接口多重继承。
>
> 当多个父接口中存在**签名完全一致**的抽象方法时，Java 允许这种重复声明，不会导致编译错误。接口中只有抽象方法声明而无实现，而同名方法在语义上被视为同一契约的多次声明，实现类只需提供一份实现即可满足所有父接口的要求。

> [!example]- 示例：`MyCalculator.java`（接口多继承）
> 演示接口通过 `extends` 同时继承多个父接口，实现类须提供所有父接口与子接口中声明的全部方法。
>
> ```java
> // Add_Sub 接口：声明加法与减法方法
> interface Add_Sub
> {
>     public void add(double x, double y);
>     public void subtract(double x, double y);
> }
>
> // Mul_Div 接口：声明乘法与除法方法
> interface Mul_Div
> {
>     public void multiply(double x, double y);
>     public void divide(double x, double y);
> }
>
> // Calculator 接口：同时继承 Add_Sub 与 Mul_Div，并新增 printResult 方法
> interface Calculator extends Add_Sub, Mul_Div
> {
>     public void printResult(double result);
> }
>
> // MyCalculator 类：实现 Calculator 接口，须提供全部 5 个方法的实现
> public class MyCalculator implements Calculator
> {
>     // 实现加法
>     public void add(double x, double y)
>     {
>         double result = x + y;
>         printResult(result);
>     }
>
>     // 实现减法
>     public void subtract(double x, double y)
>     {
>         double result = x - y;
>         printResult(result);
>     }
>
>     // 实现乘法
>     public void multiply(double x, double y)
>     {
>         double result = x * y;
>         printResult(result);
>     }
>
>     // 实现除法
>     public void divide(double x, double y)
>     {
>         double result = x / y;
>         printResult(result);
>     }
>
>     // 实现结果输出方法
>     public void printResult(double result)
>     {
>         System.out.println("The result is : " + result);
>     }
>
>     // 驱动代码
>     public static void main(String args[])
>     {
>         MyCalculator c = new MyCalculator();
>         c.add(5, 10);
>         c.subtract(35, 15);
>         c.multiply(6, 9);
>         c.divide(45, 6);
>     }
> }
> ```
>
> 输出：
> ```
> The result is : 15.0
> The result is : 20.0
> The result is : 54.0
> The result is : 7.5
> ```
>
> **接口继承链**
> ```java
> interface Calculator extends Add_Sub, Mul_Div
> ```
> - `Calculator` 继承了 `Add_Sub` 的 `add`、`subtract` 与 `Mul_Div` 的 `multiply`、`divide`，并新增 `printResult`，共 5 个抽象方法。
> - `MyCalculator implements Calculator` 须实现全部 5 个方法，缺少任何一个均会导致编译错误。
>
> **`printResult` 在其他方法中的调用**
> ```java
> public void add(double x, double y)
> {
>     double result = x + y;
>     printResult(result);
> }
> ```
> - `add`、`subtract`、`multiply`、`divide` 四个方法均将计算结果传入 `printResult`，由后者统一负责输出格式。
> - 这体现了将输出逻辑集中管理的设计思路，修改输出格式时只需改动 `printResult` 一处。
>
> **`divide(45, 6)` 的结果**
> ```java
> c.divide(45, 6);  // 输出 7.5
> ```
> - 参数类型为 `double`，`45` 与 `6` 在传入时自动提升为 `45.0` 与 `6.0`，执行浮点除法，结果为 `7.5` 而非整数 `7`。

## 14.4 可见性

父类中声明为 `private` 的成员在子类中无法直接按名引用，但这并不意味着这些成员不存在于子类对象中。

子类对象在内存中包含完整的父类数据，`private` 成员同样被继承，只是子类代码无法直接访问其名称。子类可以通过调用父类中定义的 `public` 或 `protected` 方法间接操作这些私有成员，这种访问方式称为间接可见性 (*Indirect Visibility*)。

`super` 引用可以在子类方法中用于调用父类的方法，即使当前不存在任何独立的父类对象实例。子类对象本身就包含父类的那一部分，`super` 只是提供了一种明确指向父类成员的语法途径。

> [!example]- 示例：`FoodAnalyzer.java` / `Pizza.java` / `FoodItem.java`
> 演示子类通过继承的公开方法间接访问父类的私有成员，以及 `super` 在构造器中的使用。
>
> ```java
> // FoodItem.java —— 父类，包含私有字段 fatGrams 与私有方法 calories()
> public class FoodItem
> {
>     final private int CALORIES_PER_GRAM = 9;  // 每克脂肪的卡路里数，私有常量
>     private int fatGrams;                      // 脂肪克数，私有字段
>     protected int servings;                    // 份数，子类可直接访问
>
>     // 构造器：初始化脂肪克数与份数
>     public FoodItem(int numFatGrams, int numServings)
>     {
>         fatGrams = numFatGrams;
>         servings = numServings;
>     }
>
>     // 私有方法：计算总卡路里数
>     private int calories()
>     {
>         return fatGrams * CALORIES_PER_GRAM;
>     }
>
>     // 公开方法：计算每份卡路里数，内部调用私有方法 calories()
>     public int caloriesPerServing()
>     {
>         return (calories() / servings);
>     }
> }
> ```
>
> ```java
> // Pizza.java —— 子类，通过 super() 调用父类构造器，固定份数为 8
> public class Pizza extends FoodItem
> {
>     // 构造器：只接收脂肪克数，份数固定为 8
>     public Pizza(int fatGrams)
>     {
>         super(fatGrams, 8);  // 调用 FoodItem(int, int)
>     }
> }
> ```
>
> ```java
> // FoodAnalyzer.java —— 驱动类，创建 Pizza 对象并输出每份卡路里数
> public class FoodAnalyzer
> {
>     public static void main(String[] args)
>     {
>         Pizza special = new Pizza(275);  // 275 克脂肪，8 份
>         System.out.println("Calories per serving: " +
>                            special.caloriesPerServing());
>     }
> }
> ```
>
> 输出：
> ```
> Calories per serving: 309
> ```
>
> **私有成员的间接访问**
> ```java
> public int caloriesPerServing()
> {
>     return (calories() / servings);
> }
> ```
> - `fatGrams` 与 `calories()` 均为 `FoodItem` 的 `private` 成员，`Pizza` 无法直接按名引用它们。
> - `Pizza` 继承了 `caloriesPerServing()` 这一 `public` 方法，该方法在 `FoodItem` 内部调用了 `private` 的 `calories()`。
> - `Pizza` 对象调用 `caloriesPerServing()` 时，执行的是 `FoodItem` 中的方法体，该方法体可以合法访问 `FoodItem` 的私有成员。这就是间接可见性的完整路径：`Pizza` → `caloriesPerServing()`（继承自父类）→ `calories()`（父类私有）→ `fatGrams`（父类私有）。
>
> **`protected` 与 `private` 字段的对比**
> ```java
> final private int CALORIES_PER_GRAM = 9;
> private int fatGrams;
> protected int servings;
> ```
> - `fatGrams` 与 `CALORIES_PER_GRAM` 声明为 `private`，子类无法直接访问，只能通过父类方法间接使用。
> - `servings` 声明为 `protected`，`Pizza` 若需要可直接按名引用，无需通过方法。
>
> **`super(fatGrams, 8)` 的作用**
> ```java
> public Pizza(int fatGrams)
> {
>     super(fatGrams, 8);
> }
> ```
> - `Pizza` 自身没有任何实例变量，所有数据均存储在父类部分。
> - `super(fatGrams, 8)` 调用 `FoodItem(int, int)`，将 `fatGrams` 与 `servings` 初始化为传入值与固定值 `8`。
> - 计算过程：$275 \times 9 = 2475$ 总卡路里，$2475 \div 8 = 309$（整数除法截断）。

> [!note]
> 四种访问修饰符的可见范围汇总如下：
>
> | 修饰符 | 同一类 | 同一包 | 子类 | 其他类 |
> |--------|--------|--------|------|--------|
> | `private` | ✓ | ✗ | ✗ | ✗ |
> | 无修饰符（包级） | ✓ | ✓ | ✗ | ✗ |
> | `protected` | ✓ | ✓ | ✓ | ✗ |
> | `public` | ✓ | ✓ | ✓ | ✓ |
>
> 子类中"无法直接访问"父类 `private` 成员，指的是无法在子类代码中写出该成员的名称。但该成员在对象内存中确实存在，并可通过父类提供的公开方法间接操作。

## 14.5 面向继承的设计

良好的继承设计是面向对象软件设计的重要组成部分。正确构建的继承关系能显著提升代码的可维护性、可扩展性与复用性；反之，滥用或误用继承则会使类层次结构难以理解和修改。以下是继承设计的核心原则：

每一个派生关系都应当满足 is-a 关系，子类必须是父类的一个更具体的版本。若无法用"子类 is-a 父类"自然描述两个类的关系，则应考虑使用组合 (*Composition*) 而非继承。

其余设计原则汇总如下：

- 在设计类层次结构时，预先考虑未来可能的扩展方向，将类设计为易于复用与灵活扩展的形式。
- 将各子类共有的特征（字段与方法）尽量上移至层次结构的高层，避免在多个子类中重复定义相同成员。
- 在子类中适当覆写方法，以定制或改变父类行为；但不应在子类中重新定义（遮蔽）父类的实例变量。
- 让每个类管理自己的数据：子类构造器通过 `super(...)` 调用父类构造器初始化父类部分，而非在子类中直接操作父类字段。
- 覆写 `toString` 与 `equals` 等通用方法，为自定义类提供有意义的实现。
- 使用抽象类表示层次结构中过于通用而不应被直接实例化的概念，将子类共有的结构与行为集中定义在抽象父类中。
- 谨慎选择可见性修饰符：`private` 提供最强封装，`protected` 在子类可访问性与封装性之间取得平衡，`public` 则完全开放。

#### `final` 对继承的限制 (*Restricting Inheritance with* `final` )

`final` 修饰符可以在两个层面限制继承：

将 `final` 应用于方法，禁止任何子类覆写该方法：

```java
public final void criticalMethod()
{
    // 此方法不能被任何子类覆写
}
```

将 `final` 应用于整个类，禁止该类被用作任何子类的父类：

```java
public final class ImmutableClass
{
    // 此类不能被继承
}
```

- `final` 类中的所有方法隐式地也是 `final` 的，因为该类无法被继承，覆写自然无从发生。
- 抽象类不能声明为 `final`：抽象类的存在意义在于被继承并由子类实现其抽象方法，`final` 与 `abstract` 的语义直接矛盾。

> [!note]
> Java 标准库中 `String` 类即被声明为 `final`，因此无法创建 `String` 的子类。这一设计保证了字符串的不可变性与安全性不会被子类破坏。

> [!question] 习题：绘制 UML 类图
> 绘制一个 UML 类图，显示一个继承层次结构，其中包含代表不同类型汽车的类，首先由制造商组织。显示至少其中两个类的一些适当的变量和方法名称。
>
> > [!check]-
> > 一种可行结构是先以通用 `Car` 类作为父类，再按制造商派生出 `ToyotaCar`、`HondaCar` 等子类，并在制造商类下继续派生具体车型。
> >
> > ```
> > Car
> > ├─ vin: String
> > ├─ year: int
> > ├─ start(): void
> > ├─ stop(): void
> > ├─ ToyotaCar
> > │  ├─ hybridMode: boolean
> > │  ├─ enableHybrid(): void
> > │  └─ Prius
> > │     ├─ batteryLevel: int
> > │     └─ charge(): void
> > └─ HondaCar
> >    ├─ ecoMode: boolean
> >    └─ enableEco(): void
> > ```
> >
> > 该结构满足 is-a 关系：`ToyotaCar` 是一种 `Car`，`Prius` 是一种 `ToyotaCar`。变量与方法被放在能覆盖相应对象范围的最高合理层级，避免在多个具体车型中重复定义。

---
`Pre: ` [[ELEC2543 Ch.13 Interface]]
`Post:` [[ELEC2543 Ch.15 Polymorphism]]

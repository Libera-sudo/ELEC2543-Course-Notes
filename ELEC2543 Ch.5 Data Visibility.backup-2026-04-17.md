#Y2S2 #ELEC2543
# Chapter 5. 数据可见性

`Pre: ` [[ELEC2543 Ch.4 Object Creation]]
`Post:` [[ELEC2543 Ch.6 Class Libraries]]

> [!abstract]
> 数据可见性 (*Data Visibility*) 是面向对象编程中管理数据访问权限的基础机制，隶属于封装 (*Encapsulation*) 这一核心设计原则。
>
> Java 通过可见性修饰符 (*Visibility Modifiers*) 严格划定数据与方法的访问边界，与允许直接访问结构体字段的 C 语言不同，Java 鼓励将实例数据声明为 `private`，仅通过公开方法暴露受控接口，从而将对象内部状态与外部客户端隔离。
>
> 本章涵盖数据作用域 (*Data Scope*) 与实例数据 (*Instance Data*) 的概念、封装的原理与优势、`public` / `private` / `protected` 三种可见性修饰符的语义与使用规范、访问器 (*Accessor*) 与修改器 (*Mutator*) 的设计模式，以及方法声明、参数传递与局部数据的完整机制，并以 `Account` 类为综合示例加以演示。
```table-of-contents
maxLevel: 3
```

## 5.1 数据作用域与实例数据

作用域 (*Scope*) 指程序中某个数据可被引用（使用）的范围。

- 类级数据 (*Class Level*)：在类内、方法外声明的变量，该类的所有方法均可访问。
- 局部数据 (*Local Data*) ：在方法内部声明的变量，仅在该方法内可用，方法执行结束后销毁。
- 实例数据 (*Instance Data*) ：类级数据的具体形式，每个对象创建时都会获得属于自己的独立副本，对象存在期间持续存在。

实例数据 (*Instance Data*) 是在类级别声明的变量，每个由该类创建的对象都拥有自己独立的一份副本。类的定义本身**不为实例数据分配内存空间**，仅声明数据类型。

每次通过 `new` 创建对象时，该对象的实例数据变量随之被创建并分配内存。同一个类的所有对象共享方法定义，但各自拥有独立的数据空间，这是两个对象能够处于不同状态的根本原因。

以 `Die` 类为例，`faceValue` 是实例数据，`die1` 与 `die2` 各自持有独立的 `faceValue`：

```
die1                    die2
┌─────────────┐         ┌─────────────┐
│ faceValue=5 │         │ faceValue=2 │
└─────────────┘         └─────────────┘
```

> [!note]
> `Die` 类的 `toString()` 方法中声明的 `String result` 是局部变量，仅在该方法执行期间存在，无法在类的其他方法中被引用。

> [!example]- 示例：`Die.java`
> 表示一个六面骰子，演示实例数据与局部数据的声明位置及访问规则。
>
> ```java
> public class Die
> {
>     private final int MAX = 6;   // 实例数据：最大面值
>     private int faceValue;       // 实例数据：当前面值
>
>     // 构造器：初始化面值为 1
>     public Die()
>     {
>         faceValue = 1;
>     }
>
>     // 掷骰子，返回随机面值
>     public int roll()
>     {
>         faceValue = (int)(Math.random() * MAX) + 1;
>         return faceValue;
>     }
>
>     // 修改器：设置面值
>     public void setFaceValue(int value)
>     {
>         faceValue = value;
>     }
>
>     // 访问器：获取面值
>     public int getFaceValue()
>     {
>         return faceValue;
>     }
>
>     // 返回字符串表示
>     public String toString()
>     {
>         String result = Integer.toString(faceValue); // 局部数据
>         return result;
>     }
> }
> ```
>
> **实例数据的声明**
>
> ```java
> private final int MAX = 6;
> private int faceValue;
> ```
>
> - `MAX` 与 `faceValue` 声明于类体内、方法体外，是实例数据，类中所有方法均可直接引用。
> - `private` 修饰符限制外部直接访问，体现封装原则。
> - `final` 使 `MAX` 成为常量，值不可修改。
>
> **构造器与 `roll()` 对实例数据的访问**
>
> ```java
> public Die() { faceValue = 1; }
>
> public int roll()
> {
>     faceValue = (int)(Math.random() * MAX) + 1;
>     return faceValue;
> }
> ```
>
> - 构造器与 `roll()` 直接读写 `faceValue`，无需任何额外传参，因为它们与 `faceValue` 同属一个对象。
> - `Math.random()` 返回 $[0.0,\ 1.0)$ 的随机数，乘以 `MAX` 后取整再加 1，结果范围为 $[1,\ 6]$。
>
> **局部数据的声明**
>
> ```java
> public String toString()
> {
>     String result = Integer.toString(faceValue);
>     return result;
> }
> ```
>
> - `result` 声明于 `toString()` 方法内部，是局部数据，方法执行结束后即被销毁。
> - 在 `Die` 类的其他方法中无法引用 `result`。

## 5.2 封装

封装是面向对象设计的核心原则之一。封装将对象的内部数据与外部客户端 (*Client*) 隔离，客户端只能通过对象提供的公开接口 (*Interface*) 与之交互，而无需了解其内部实现细节。

- 对象可被视为黑盒 (*Black Box*)：内部工作机制对客户端不可见。
- 对象状态的修改应由该对象自身的方法负责，而非由外部直接操作。
- 封装使客户端难以（甚至无法）直接访问对象的内部变量。

**封装的优势：**

- 保护对象数据不被客户端随意修改。
- 允许在不暴露底层实现的情况下提供访问接口。
- 降低因错误操作导致的程序错误概率。
- 简化程序的维护与理解成本。

> [!example]- 示例：`Car.java`（无封装 vs. 有封装）
> 演示 `public` 实例变量导致的不受控访问问题，以及封装如何加以约束。
>
> **无封装版本**
>
> ```java
> // 无封装版本
> class Car {
>     public float speed = 0;
>     public boolean isReverse = false;
>     public boolean isStarted = false;
> }
>
> class Main {
>     public static void main(String[] args) {
>         Car car = new Car();
>         // No need to start??
>         car.speed = 100; // Turbo mode: directly to 100
>         car.speed = 0;   // Turbo brake
>     }
> }
> ```
>
> **问题分析**
>
> ```java
> car.speed = 100;
> car.speed = 0;
> ```
>
> - `speed`、`isReverse`、`isStarted` 均以 `public` 声明，客户端可不经任何检查直接赋值。
> - 汽车未启动（`isStarted == false`）即可将速度直接设为 100，违背现实逻辑约束。
> - 速度可瞬间归零，绕过任何减速过程，属于非法但语法合法的操作。
>
> **有封装版本**
>
> ```java
> class Car {
>     private float speed = 0;
>     private boolean isReverse = false;
>     private boolean isStarted = false;
>
>     // 启动汽车
>     public void start() {
>         isStarted = true;
>     }
>
>     // 加速：仅在已启动且非倒车状态下有效，速度上限为 200
>     public void accelerate(float amount) {
>         if (isStarted && !isReverse && speed + amount <= 200) {
>             speed += amount;
>         }
>     }
>
>     // 刹车：速度逐步减小，不低于 0
>     public void brake(float amount) {
>         speed = Math.max(0, speed - amount);
>     }
>
>     public float getSpeed() {
>         return speed;
>     }
> }
>
> class Main {
>     public static void main(String[] args) {
>         Car car = new Car();
>         car.start();
>         car.accelerate(30);  // 合法操作，speed = 30
>         car.brake(10);       // 合法操作，speed = 20
>         // car.speed = 100;  // 编译错误：speed 为 private
>     }
> }
> ```
>
> **封装后的访问控制**
>
> ```java
> private float speed = 0;
> private boolean isReverse = false;
> private boolean isStarted = false;
> ```
>
> - 三个实例变量改为 `private`，外部类无法直接读写。
> - 所有状态变更必须经由 `start()`、`accelerate()`、`brake()` 等方法，方法内部可加入逻辑约束。
>
> ```java
> public void accelerate(float amount) {
>     if (isStarted && !isReverse && speed + amount <= 200) {
>         speed += amount;
>     }
> }
> ```
>
> - 加速前检查车辆是否已启动、是否处于前进状态、速度是否超限，确保状态转换符合现实逻辑。

## 5.3 可见性修饰符

可见性修饰符 (*Visibility Modifier*) 是 Java 保留字，用于指定类成员的访问权限范围，是实现封装的核心语言机制。

Java 提供三种可见性修饰符：

- `public`：成员可在任意位置被引用。
- `private`：成员仅在声明它的类内部可被引用。
- `protected`：与继承 (*Inheritance*) 相关，后续章节讨论。

不加任何修饰符时为默认可见性 (*Default Visibility*)，同一包 (*Package*)内的任意类均可访问。

#### 变量的可见性规范

- 实例变量不应声明为 `public`，否则客户端可直接修改其值，破坏封装。
- 常量（`final` 修饰的变量）可声明为 `public`，因为其值无法被修改，不违反封装原则。

  ```java
  public final int MAX = 6; // 合法：客户端可读取但无法修改
  ```

#### 方法的可见性规范

- 提供对象服务的方法声明为 `public`，称为服务方法 (*Service Methods*)，供客户端调用。
- 仅用于辅助其他方法的内部方法声明为 `private`，称为支持方法 (*Support Methods*)，不对外暴露。

推荐的可见性规范：

|     | `public` | `private` |
| --- | -------- | --------- |
| 变量  | 违反封装     | 推荐        |
| 方法  | 提供服务，推荐  | 辅助内部逻辑    |

> [!note]
> `public` 变量允许客户端"伸手"直接修改对象内部状态，绕过任何逻辑约束，因此实例变量应始终声明为 `private`。

> [!note]
> 支持方法是类的实现细节，声明为 `private` 后，即使将来修改或删除该方法，也不会影响客户端代码，有助于降低耦合。

> [!example]- 示例：可见性修饰符的使用
> 演示 `public`、`private` 在变量与方法上的典型用法。
>
> ```java
> /*
> 课件外示例
> */
> public class Circle {
>     // public 常量：客户端可读取，值不可修改
>     public final double PI = 3.14159;
>
>     // private 实例变量：仅类内部可访问
>     private double radius;
>
>     // public 构造器：服务方法，供客户端创建对象
>     public Circle(double radius) {
>         this.radius = radius;
>     }
>
>     // public 服务方法：供客户端调用
>     public double getArea() {
>         return PI * square(radius); // 调用私有支持方法
>     }
>
>     // private 支持方法：仅供类内部使用
>     private double square(double x) {
>         return x * x;
>     }
> }
>
> public class Main {
>     public static void main(String[] args) {
>         Circle c = new Circle(5.0);
>         System.out.println(c.PI);        // 合法：public 常量
>         System.out.println(c.getArea()); // 合法：public 方法
>         // c.radius = 10;               // 编译错误：private 变量
>         // c.square(5);                 // 编译错误：private 方法
>     }
> }
> ```
>
> **变量声明**
>
> ```java
> public final double PI = 3.14159;
> private double radius;
> ```
>
> - `PI` 为 `public final`，客户端可读取其值，但 `final` 禁止赋值修改。
> - `radius` 为 `private`，外部类无法直接读写，必须通过方法访问。
>
> **服务方法与支持方法**
>
> ```java
> public double getArea() {
>     return PI * square(radius);
> }
>
> private double square(double x) {
>     return x * x;
> }
> ```
>
> - `getArea()` 是服务方法，声明为 `public`，定义对外提供的功能接口。
> - `square()` 是支持方法，声明为 `private`，仅被 `getArea()` 调用，客户端不可见。

## 5.4 访问器与修改器

由于实例变量声明为 `private`，类通常需要提供专门的方法供外部读取和修改这些变量。

- 访问器 (*Accessor*) 方法返回某个实例变量的当前值，命名形式为 `getX`。
- 修改器 (*Mutator*) 方法修改某个实例变量的值，命名形式为 `setX`。
- `X` 为对应变量名，首字母大写；访问器与修改器也常合称 `getter` / `setter`。

修改器与直接将变量声明为 `public` 的区别在于：修改器内部可加入逻辑约束，拒绝非法值，而 `public` 变量无法施加任何限制。

> [!example]- 例题：`Point` 类的访问器与拷贝构造器
> 为表示笛卡尔平面上坐标点的 `Point` 类编写访问器方法，以及拷贝构造器 `public Point(Point p)`。
>
> ```java
> public class Point {
>     private double x;
>     private double y;
>
>     // 访问器：返回 x 坐标
>     public double getX() {
>         return x;
>     }
>
>     // 访问器：返回 y 坐标
>     public double getY() {
>         return y;
>     }
>
>     // 拷贝构造器：将 p 的坐标复制到当前对象
>     public Point(Point p) {
>         x = p.x;
>         y = p.y;
>     }
> }
> ```
>
> **访问器方法**
>
> ```java
> public double getX() { return x; }
> public double getY() { return y; }
> ```
>
> - 返回类型与对应实例变量类型一致，均为 `double`。
> - 方法体仅含一条 `return` 语句，直接返回实例变量的当前值。
>
> **拷贝构造器**
>
> ```java
> public Point(Point p) {
>     x = p.x;
>     y = p.y;
> }
> ```
>
> - 参数 `p` 是同类型 `Point` 对象。
> - 在同一个类内部，`private` 变量对同类的其他对象实例同样可见，因此 `p.x` 和 `p.y` 可直接访问，无需经由访问器。
> - 该构造器将 `p` 的坐标值复制到当前对象（`this`）的实例变量中。

> [!example]- 例题：`Die` 类的受限 `setFaceValue`
> 改写 `Die` 类的 `setFaceValue` 方法，使 `faceValue` 仅在参数值位于 $[1,\ \text{MAX}]$ 范围内时才更新。
>
> ```java
> public void setFaceValue(int value) {
>     if ((value >= 1) && (value <= MAX)) {
>         faceValue = value;
>     }
> }
> ```
>
> **条件判断**
>
> ```java
> if ((value >= 1) && (value <= MAX))
> ```
>
> - 同时检查下界（不小于 1）与上界（不超过 `MAX`，即 6），两个条件均满足时才执行赋值。
> - 若 `value` 超出范围，方法不执行任何操作，`faceValue` 保持原值不变。
>
> **与 `public` 变量的对比**
>
> - 若 `faceValue` 为 `public`，客户端可执行 `die.faceValue = 100`，无任何约束。
> - 通过修改器加入范围检查后，非法赋值被静默拒绝，对象状态始终合法。

> [!note]
> 当非法输入被静默忽略时，调用方不会收到任何错误提示。更健壮的设计应在值非法时抛出异常 (*Exception*)，或返回一个表示操作结果的布尔值，后续章节将讨论异常处理机制。

## 5.5 方法声明与参数传递

方法声明 (*Method Declaration*) 指定了方法被调用时所执行的代码。调用方法时，控制流跳转至该方法并执行其代码，执行完毕后返回调用位置继续执行。

#### 控制流 (*Control Flow*)

- 若被调用方法与调用方在同一个类中，直接使用方法名调用：
  ```java
  myMethod();
  ```

- 若被调用方法属于另一个类或对象，通过对象引用调用：
  ```java
  obj.doIt();
  ```

#### 方法头 (*Method Header*)

方法声明以方法头开始，指定返回类型、方法名与参数列表：

```java
char calc(int num1, int num2, String message)
```

- 返回类型：
	`char`，方法执行完毕后返回一个 `char` 值。
- 方法名：
	`calc`。
- 参数列表 (*Parameter List*)：
	指定每个参数的类型与名称。
- 参数列表中的参数名称称为形式参数 (*Formal Parameter*)。

#### 方法体 (*Method Body*)

方法体位于方法头之后，包含具体执行逻辑：

```java
char calc(int num1, int num2, String message)
{
    int sum = num1 + num2;
    char result = message.charAt(sum);
    return result;
}
```

- `sum` 与 `result` 是局部数据，每次方法被调用时创建，方法执行结束后销毁。
- `return result` 将值返回给调用方，返回值类型须与方法头声明的返回类型一致。

#### `return` 语句

- 返回类型为 `void` 的方法不返回任何值，可省略 `return` 语句或使用不带表达式的 `return`。
- 返回类型非 `void` 时，`return` 表达式的类型须与声明的返回类型一致：
  ```java
  return expression;
  ```

#### 参数传递 (*Parameter Passing*)

方法被调用时，调用方提供的实际参数 (*Actual Parameters*) 的值被逐一复制到方法头中对应的形式参数：

```java
ch = obj.calc(25, count, "Hello");
```

```java
char calc(int num1, int num2, String message) { ... }
```

- `25` 复制到 `num1`，`count` 的当前值复制到 `num2`，`"Hello"` 复制到 `message`。
- 这种传递方式称为值传递 (*Pass by Value*)：方法内对形式参数的修改不影响调用方的实际参数。

#### 局部数据的生命周期

- 方法的形式参数本质上也是局部变量，方法调用时自动创建，方法结束时销毁。
- 实例变量声明于类级别，其生命周期与所属对象一致，对象存在则变量存在。

|      | 局部数据   | 实例数据    |
| ---- | ------ | ------- |
| 声明位置 | 方法内部   | 类体内、方法外 |
| 生命周期 | 方法调用期间 | 对象存在期间  |
| 访问范围 | 仅本方法   | 类中所有方法  |

> [!note]
> Java 中基本类型 (*Primitive Types*) 的参数传递始终是值传递，方法内修改形式参数不会影响调用方的变量。对象引用的传递同样是值传递，传递的是引用的副本，但通过该引用仍可修改对象内部状态。

> [!example]- 示例：`calc` 方法的参数传递过程
> 演示实际参数如何复制到形式参数，以及局部变量的创建与销毁。
>
> ```java
> /*
> 课件外示例
> */
> public class Calculator {
>     // 根据两个整数之和，从字符串中取出对应字符
>     public char calc(int num1, int num2, String message) {
>         int sum = num1 + num2;               // 局部变量
>         char result = message.charAt(sum);   // 局部变量
>         return result;
>     }
> }
>
> public class Main {
>     public static void main(String[] args) {
>         Calculator obj = new Calculator();
>         int count = 3;
>         char ch = obj.calc(25, count, "Hello, World!");
>         System.out.println(ch); // 输出：,
>     }
> }
> ```
>
> **调用与参数复制**
>
> ```java
> char ch = obj.calc(25, count, "Hello, World!");
> ```
>
> - `25` 复制到 `num1`，`count`（值为 3）复制到 `num2`，`"Hello, World!"` 复制到 `message`。
> - 实际参数与形式参数相互独立，方法内修改 `num1` 不会影响调用方。
>
> **方法体执行**
>
> ```java
> int sum = num1 + num2;             // sum = 25 + 3 = 28
> char result = message.charAt(sum); // "Hello, World!".charAt(28) 越界！
> ```
>
> > [!note]
> > 此处 `25 + 3 = 28` 超出字符串 `"Hello, World!"` 的长度（13 个字符），实际运行会抛出 `StringIndexOutOfBoundsException`。课件原例仅用于说明参数传递机制，实际使用时应确保索引合法。将调用改为 `obj.calc(2, 3, "Hello, World!")` 则 `sum = 5`，`charAt(5)` 返回 `','`。
>
> **局部变量的销毁**
>
> ```java
> return result;
> ```
>
> - 方法返回后，`sum`、`result`、`num1`、`num2`、`message` 均被销毁，无法在方法外部访问。

## 5.6 银行账户示例

本节以银行账户为场景，综合演示类的设计、封装、访问器与修改器、方法声明及参数传递。

驱动程序 (*Driver Program*) 是专门用于驱动和测试其他类的程序，通常包含 `main` 方法。本例中 `Transactions` 类是驱动程序，`Account` 类是被测试的核心类。

`Account` 类的状态主要由以下实例变量表示：
- 账户号码 `acctNumber`
- 当前余额 `balance`
- 账户持有人姓名 `name`

`Account` 类提供的服务包括：存款 (`deposit`)、取款 (`withdraw`)、计息 (`addInterest`)、查询余额 (`getBalance`)。

#### 构造器注意事项

- 构造器的方法头不指定任何返回类型，包括 `void` 也不写。
- 若误将构造器写为带返回类型的方法，该方法将成为一个与类同名的普通方法，而非构造器，对象创建时不会调用它。
- 若类中未定义任何构造器，Java 自动提供一个无参默认构造器 (*Default Constructor*)。

> [!note]
> 一旦类中显式定义了任意构造器，默认构造器将不再自动提供。若仍需无参构造，须手动定义。

> [!example]- 示例：`Account.java` + `Transactions.java`
> 演示多个银行账户对象的创建与方法调用，验证每个对象独立维护自身状态。
>
> ```java
> // Account.java
> import java.text.NumberFormat;
>
> public class Account
> {
>     private final double RATE = 0.035; // 年利率 3.5%
>     private long acctNumber;
>     private double balance;
>     private String name;
>
>     // 构造器：初始化账户持有人、账号与初始余额
>     public Account(String owner, long account, double initial)
>     {
>         name = owner;
>         acctNumber = account;
>         balance = initial;
>     }
>
>     // 存款：将 amount 加入余额，返回新余额
>     public double deposit(double amount)
>     {
>         balance = balance + amount;
>         return balance;
>     }
>
>     // 取款：扣除金额与手续费，返回新余额
>     public double withdraw(double amount, double fee)
>     {
>         balance = balance - amount - fee;
>         return balance;
>     }
>
>     // 计息：按 RATE 计算并加入余额，返回新余额
>     public double addInterest()
>     {
>         balance += (balance * RATE);
>         return balance;
>     }
>
>     // 访问器：返回当前余额
>     public double getBalance()
>     {
>         return balance;
>     }
>
>     // 返回账户的单行字符串描述
>     public String toString()
>     {
>         NumberFormat fmt = NumberFormat.getCurrencyInstance();
>         return (acctNumber + "\t" + name + "\t" + fmt.format(balance));
>     }
> }
> ```
>
> ```java
> // Transactions.java
> public class Transactions
> {
>     public static void main(String[] args)
>     {
>         Account acct1 = new Account("Ted Murphy",    72354, 102.56);
>         Account acct2 = new Account("Jane Smith",    69713,  40.00);
>         Account acct3 = new Account("Edward Demsey", 93757, 759.32);
>
>         acct1.deposit(25.85);
>
>         double smithBalance = acct2.deposit(500.00);
>         System.out.println("Smith balance after deposit: " + smithBalance);
>
>         System.out.println("Smith balance after withdrawal: " +
>                             acct2.withdraw(430.75, 1.50));
>
>         acct1.addInterest();
>         acct2.addInterest();
>         acct3.addInterest();
>
>         System.out.println();
>         System.out.println(acct1);
>         System.out.println(acct2);
>         System.out.println(acct3);
>     }
> }
> ```
>
> 输出：
> ```
> Smith balance after deposit: 540.0
> Smith balance after withdrawal: 107.75
>
> 72354   Ted Murphy      $132.90
> 69713   Jane Smith      $111.52
> 93757   Edward Demsey   $785.90
> ```
>
> **实例数据声明**
>
> ```java
> private final double RATE = 0.035;
> private long acctNumber;
> private double balance;
> private String name;
> ```
>
> - 所有实例变量声明为 `private`，外部类无法直接访问。
> - `RATE` 为 `private final`，值固定为 0.035，仅供类内部方法使用。
>
> **构造器**
>
> ```java
> public Account(String owner, long account, double initial)
> {
>     name = owner;
>     acctNumber = account;
>     balance = initial;
> }
> ```
>
> - 接收三个参数，分别初始化 `name`、`acctNumber`、`balance`。
> - 无返回类型声明，是合法的构造器形式。
>
> **存款与取款方法**
>
> ```java
> public double deposit(double amount)
> {
>     balance = balance + amount;
>     return balance;
> }
>
> public double withdraw(double amount, double fee)
> {
>     balance = balance - amount - fee;
>     return balance;
> }
> ```
>
> - `deposit` 将 `amount` 加入 `balance` 并返回新余额。
> - `withdraw` 同时扣除取款金额与手续费，返回新余额。
> - 两个方法均直接修改实例变量 `balance`，体现了对象自身管理自身状态的封装原则。
>
> **计息方法**
>
> ```java
> public double addInterest()
> {
>     balance += (balance * RATE);
>     return balance;
> }
> ```
>
> - `balance * RATE` 计算本期利息，加入 `balance`。
> - 等价于 `balance = balance * (1 + RATE)`。
>
> **`toString` 方法**
>
> ```java
> public String toString()
> {
>     NumberFormat fmt = NumberFormat.getCurrencyInstance();
>     return (acctNumber + "\t" + name + "\t" + fmt.format(balance));
> }
> ```
>
> - `NumberFormat.getCurrencyInstance()` 返回当前地区的货币格式化器，将 `balance` 格式化为货币字符串（如 `$132.90`）。
> - `\t` 为制表符，用于对齐输出。
> - `System.out.println(acct1)` 会自动调用 `acct1.toString()`。
>
> **驱动程序中的对象操作**
>
> ```java
> Account acct1 = new Account("Ted Murphy", 72354, 102.56);
> Account acct2 = new Account("Jane Smith", 69713, 40.00);
> Account acct3 = new Account("Edward Demsey", 93757, 759.32);
> ```
>
> - 三个 `Account` 对象各自拥有独立的 `balance`、`name`、`acctNumber`，互不干扰。
>
> ```java
> acct1.deposit(25.85);
> double smithBalance = acct2.deposit(500.00);
> ```
>
> - `acct1.deposit(25.85)` 仅修改 `acct1` 的 `balance`，`acct2` 与 `acct3` 不受影响。
> - `acct2.deposit(500.00)` 的返回值（新余额 540.0）被存入 `smithBalance`。

> [!note]
> 当前 `withdraw` 方法未验证 `amount` 是否为正数，也未检查余额是否充足，可能导致 `balance` 变为负值。更健壮的实现应加入参数合法性校验，或在余额不足时拒绝操作。

---

`Pre: ` [[ELEC2543 Ch.4 Object Creation]]
`Post:` [[ELEC2543 Ch.6 Class Libraries]]

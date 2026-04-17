#Y2S2 #ELEC2543
# Chapter 1. Java 引入

`Post:` [[ELEC2543 Ch.2 Expression & Java Syntax]]

> [!abstract]
> Java 引入内容属于程序设计基础与面向对象程序设计 (*Object-Oriented Programming*) 的起点部分，讨论程序如何被组织、编译与执行，以及类与对象这些核心抽象如何进入实际编程语境。
>
> Java 采用"编写一次，到处运行" (*Write Once, Run Anywhere*) 的设计理念，通过编译器 (*Compiler*) 将源代码转换为与平台无关的字节码 (*Bytecode*)，再由 Java 虚拟机 (*Java Virtual Machine*, *JVM*) 执行。与依赖源代码直接解释执行的语言相比，这一机制更强调统一的运行环境与明确的程序入口约定。
>
> 本章具体介绍 Java 程序的基本结构、`main` 方法作为程序入口的签名规则、类 (*Class*) 与对象 (*Object*) 的基本关系，以及标识符与保留字的命名规范。
```table-of-contents
maxLevel: 3
```
## 1.1 Java 程序结构

Java 程序具有清晰的层次结构，可概括为以下三个层面：

- 类 (*Class*)：所有代码必须位于类内，每个公共类保存在与类名完全一致的 `.java` 文件中。
- 方法 (*Method*)：方法定义在类内；每个独立的 Java 程序**必须**包含一个名为 `main` 的方法作为入口。
- 语句 (*Statement*)：语句是实际执行操作的代码单元，位于方法体内部。

> [!example]- 示例：`HelloWorld.java`
>
> 这个最小程序展示 Java 的基本执行入口，以及类、`main` 方法与输出语句之间的组织关系。
>
> ```java
> // 文件名必须是 HelloWorld.java（与类名完全一致）
>
> public class HelloWorld {
>
>     // 程序入口
>     public static void main(String[] args) {
>         // 向标准输出打印一行文本
>         System.out.println("Hello World!");
>     }
>
> }
> ```
>
> 输出：
>
> ```
> Hello World!
> ```
>
> **类定义**
> ```java
> public class HelloWorld { ... }
> ```
> - `public class` 声明一个公共类，类名 `HelloWorld` 须与文件名完全一致。
> - Java 使用大括号 `{}` 划定代码的作用域 (*Scope*)，缩进 (*Indentation*) 不具有语义作用。
>
> **程序入口**
> ```java
> public static void main(String[] args) { ... }
> ```
> - `main` 是 JVM 寻找程序起始点的约定标识，签名须完全一致。
> - 关键字含义详见本节 `main` 方法签名解析。
>
> **输出语句**
> ```java
> System.out.println("Hello World!");
> ```
> - `System.out` 是标准输出流对象，`println` 方法打印传入的字符串并在末尾换行。
> - JVM 启动后顺序执行该语句，在屏幕上打印 `Hello World!`。

> [!note]
>
> `public static void main(String[] args)` 是 JVM 识别程序入口的方法签名，各组成部分的含义如下：
>
> - `public`：访问修饰符 (*Access Modifier*)。主方法必须全局可见，使 JVM 在外部环境启动时有权限调用它；若改为 `private`，将报错"找不到主方法"。
>
> - `static`：使该方法成为类级别的方法，无需依赖具体的对象实例即可直接调用。若 `main` 为非静态方法，JVM 将陷入"需要对象才能调用 `main`，但程序尚未启动"的循环依赖。
>
> - `void`：返回类型，表示该方法不返回任何数据。`main` 方法结束即程序终止，JVM 无处理返回值的需求。
>
> - `main`：方法的标识符 (*Identifier*)，非保留字，但是 JVM 寻找程序起始点的约定标识。
>
> - `String[] args`：形参列表，用于接收命令行参数。
> - `String`：数组元素类型为字符串。
> - `[]`：表示该参数是数组，可接收多个字符串输入。
> - `args`：参数名，可替换为其他合法标识符（如 `argv`），但 `args` 是常见约定命名。


## 1.2 面向对象编程概念

在 Java 中，对象 (*Object*) 是现实世界事物的数字化映射，每个对象具有两个核心特征：

- 状态 (*State*)：描述对象的属性，即它当前"包含什么数据"。
- 行为 (*Behavior*)：对象能执行的操作；行为同时可以改变对象自身的状态。

类 (*Class*) 是创建对象的模板，规范了该类型对象应有的：

- 状态：由实例变量 (*Instance Variables* / *Fields*) 描述。
- 行为：由方法 (*Methods*) 定义。

在 Java 中，一切代码必须位于类内，包括作为程序入口的 `main` 方法。

下面的示例给出了一个简单的类定义。

> [!example]- 示例：`BankAccount.java`
>
> 这个类定义展示字段如何表示对象状态，以及方法如何封装对象行为。
>
> ```java
> public class BankAccount {
>
>     // 账号字段，表示账户编号
>     private String accountNumber;
>     // 余额字段，表示当前账户状态
>     private double balance;
>
>     // 存款：将传入金额累加到余额
>     public void deposit(double amount) {
>         balance += amount;
>     }
>
>     // 取款：仅在余额足够时才扣减
>     public void withdraw(double amount) {
>         if (balance >= amount) {
>             balance -= amount;
>         }
>     }
>
>     // 访问器：返回当前余额
>     public double getBalance() {
>         return balance;
>     }
> }
> ```
>
> **实例变量（状态）**
> ```java
> private String accountNumber;
> private double balance;
> ```
> - 实例变量 (*Instance Variables*) 定义对象的状态 (*State*)，在类级别声明，类中所有方法均可访问。
> - `private` 关键字实现封装 (*Encapsulation*)：外部代码不能直接访问这些变量，只能通过方法操作。
> - 类中只声明变量类型与名称，具体数值在对象实例化时分配。
>
> **方法（行为）**
> ```java
> public void deposit(double amount) {
>     balance += amount;
> }
>
> public void withdraw(double amount) {
>     if (balance >= amount) {
>         balance -= amount;
>     }
> }
>
> public double getBalance() {
>     return balance;
> }
> ```
> - 方法 (*Methods*) 定义对象的行为 (*Behavior*)，所有由该类创建的对象共享方法定义，但各自维护独立的状态。
> - `deposit` 将传入金额累加至 `balance`，修改当前对象的状态。
> - `withdraw` 在余额充足时才执行扣减，体现了方法内部对状态变更的逻辑约束。
> - `getBalance` 返回当前对象 `balance` 的值，是典型的访问器 (*Accessor*) 方法。

> [!note]
>
> 同一个类可以实例化为多个彼此独立的对象，类负责提供统一的结构定义，而每个对象各自保存自己的状态：
>
> ```java
> BankAccount johnsAccount = new BankAccount();   // 使用类作为模板，创建对象
> BankAccount marysAccount = new BankAccount();   // 使用同一个类，创建另一个独立对象
>
> johnsAccount.deposit(1000);   // 只修改 john 的余额状态
> marysAccount.deposit(500);    // 只修改 mary 的余额状态
> ```
>
> `johnsAccount` 与 `marysAccount` 均由 `BankAccount` 类创建，方法定义完全相同，但各自维护独立的 `accountNumber` 与 `balance`。这说明类共享的是行为定义，对象独立的是运行时状态。

继承 (*Inheritance*) 是面向对象程序设计的另一项核心机制。

- 被继承的类称为父类 (*Superclass* / *Parent Class*) 或基类 (*Base Class*)。
- 继承该类的类称为子类 (*Subclass* / *Child Class*) 或派生类 (*Derived Class*)。

子类自动获得父类中所有非私有的成员（字段与方法），并可以在此基础上继续添加新的属性和方法，或通过覆写 (*Overriding*) 改变继承而来的方法行为。

> [!note]
> 银行先定义基础的 `Account` 类，在此基础上派生出 `BankAccount` 与 `CreditCardAccount`；`BankAccount` 进一步细分为 `SavingsAccount` 与 `CheckingAccount`。
>
> 下层类自动继承上层的通用属性，无需在每个子类中重复声明"账号"等基础变量。


## 1.3 标识符和保留字

标识符 (*Identifier*) 是程序中用于命名各种元素的名称，Java 对其有如下语法规则：

- 只能由字母、数字、下划线 `_` 和美元符号 `$` 组成。
- **不能**以数字开头。
- 区分大小写 (*Case Sensitive*)。

保留字是 Java 语法预先占用的词，例如 `public`、`class`、`static`、`void` 与 `if`。这些词在语言中已经具有固定语法作用，因此不能再作为类名、变量名或方法名使用。

> [!note]
> `$` 在语法上允许出现在标识符中，但普通 Java 代码通常不将其作为命名习惯的首选。实际编程中更稳妥的做法是使用字母、数字与下划线组合命名，以保持代码风格稳定并降低阅读负担。

除硬性规定外，Java 社区约定了以下命名规范：

- 类名 (*Class Name*)：大驼峰式 (*PascalCase*)，每个单词首字母大写，如 `HelloWorld`。
- 变量名与方法名：小驼峰式 (*camelCase*)，首单词小写，后续单词首字母大写，如 `studentName`、`calculateAverage`。
- 常量名 (*Constant Name*)：全大写，单词间以下划线分隔，如 `MAXIMUM`。

> [!example]- 示例：命名规范示例
>
> 这段代码集中展示类名、字段名、方法名、局部变量名与常量名在同一程序中的命名规范。
>
>
> ```java
> /*
> 课件外示例
> */
>
> // 类名：大驼峰命名法（PascalCase）
> public class StudentGradeCalculator {
>
>     // 常量：全大写，单词间用下划线分隔
>     static final int MAX_STUDENTS = 30;
>     static final double PASS_MARK = 60.0;
>
>     // 实例变量：小驼峰命名法（camelCase）
>     private String studentName;
>     private int studentAge;
>     private double[] examScores;
>
>     // 构造器：与类名完全一致
>     public StudentGradeCalculator(String studentName, int studentAge) {
>         this.studentName = studentName;
>         this.studentAge = studentAge;
>         this.examScores = new double[MAX_STUDENTS];
>     }
>
>     // 方法名：小驼峰命名法，动词开头
>     public double calculateAverage() {
>         double totalScore = 0.0;   // 局部变量：小驼峰命名法
>
>         for (double score : examScores) {
>             // 遍历数组并累加每个分数
>             totalScore += score;
>         }
>
>         // 返回平均分
>         return totalScore / examScores.length;
>     }
>
>     // 布尔方法：以 is / has / can 等开头
>     public boolean isPassingGrade(double averageScore) {
>         return averageScore >= PASS_MARK;
>     }
> }
> ```
>
> **类名与构造器**
> ```java
> public class StudentGradeCalculator {
>     public StudentGradeCalculator(String studentName, int studentAge) { ... }
> }
> ```
> - 类名使用大驼峰式，构造器名称必须与类名完全一致。
> - 这种命名方式有助于在阅读代码时快速区分类本身与变量、方法等其他元素。
>
> **常量与字段**
> ```java
> static final int MAX_STUDENTS = 30;
> static final double PASS_MARK = 60.0;
> private String studentName;
> private int studentAge;
> private double[] examScores;
> ```
> - 常量名全部大写，并以下划线分隔单词，用于强调该值在程序运行期间不应被修改。
> - 字段名使用小驼峰式，使其与类名、常量名在视觉上形成稳定区分。
>
> **方法与局部变量**
> ```java
> public double calculateAverage() {
>     double totalScore = 0.0;
>     ...
> }
>
> public boolean isPassingGrade(double averageScore) {
>     return averageScore >= PASS_MARK;
> }
> ```
> - 方法名与局部变量名通常都使用小驼峰式，但方法名更强调行为语义，常以动词或动词短语开头。
> - 布尔方法常以 `is`、`has`、`can` 等开头，使返回值的含义能从名称上直接判断。
>
> 上述示例体现的命名规范可概括如下。
>
> - 类名 (*Class Name*)：大驼峰命名法，每个单词首字母大写，如 `StudentGradeCalculator`。
> - 变量名与方法名：小驼峰命名法，首单词全小写，后续单词首字母大写，如 `studentName`、`calculateAverage`。
> - 常量名：全部大写，单词间以下划线 `_` 分隔，如 `MAX_STUDENTS`。
> - 方法名应以动词开头，清晰表达其行为，如 `calculate`、`is`、`get`、`set`。

---
`Post:` [[ELEC2543 Ch.2 Expression & Java Syntax]]

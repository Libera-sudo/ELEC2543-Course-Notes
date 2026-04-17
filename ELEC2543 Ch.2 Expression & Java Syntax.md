#Y2S2 #ELEC2543
# Chapter 2. 基础语法与表达式

`Pre: ` [[ELEC2543 Ch.1 Introduction to Java]]
`Post:` [[ELEC2543 Ch.3 Introduction to Classes]]

> [!abstract]
> 基础语法与表达式属于程序设计语言入门的核心部分，讨论数据如何被表示、表达式如何被求值，以及控制结构如何组织程序的执行流程。
>
> Java 是强类型语言 (*Strongly Typed Language*)，强调在编译期完成类型检查，并通过明确的控制流 (*Control Flow*) 与作用域 (*Scope*) 规则约束程序结构。与动态类型语言相比，这种设计更强调语义明确性与编译阶段的错误暴露。
>
> 本章具体介绍基本数据类型 (*Primitive Data Types*)、表达式与运算符、条件分支与循环结构、方法与作用域、字符串与控制台输出、数据类型转换，以及标准输入 (*Standard Input*) 的基本使用。
```table-of-contents
maxLevel: 3
```

## 2.1 变量与基本数据类型

Java 提供 **8 种**基础数据类型 (*Primitive Data Types*)，可按用途归纳为四类：

- 整数类型 (*Integer Types*)：`byte` (8 位)、`short` (16 位)、`int` (32 位)、`long` (64 位)。
- 浮点类型 (*Floating-Point Types*)：`float` (单精度，约 6 位有效数字)、`double` (双精度，约 15 位有效数字)。
- 字符类型 (*Character Type*)：`char` (通常 16 位)。
- 布尔类型 (*Boolean Type*)：`boolean`，取值仅为 `true` 或 `false`。

> [!note]
> Java 的 `char` 采用 16 位 Unicode 编码，不仅能表示英文字母，也能表示多种语言的文字符号。

> [!note]
> `boolean` 只有两个合法的值：`true` 和 `false`，不能用 0 或 1 来代替真假。

变量在使用前必须先声明，其基本语法为 `type variableName;`。
```java
int ThisIsANumber;
```

声明与初始化也可在同一语句中完成。
```java
boolean True = true;
```

在声明前加上 `final` 可将变量声明为常量。
```java
// 声明一个常量，通常常量名习惯全部大写，以示区别
final int THIS_IS_A_CONSTANT = 0;
```

## 2.2 表达式与运算符

在 Java 中，一个表达式是由一个或多个运算符 (*Operator*) 和操作数 (*Operand*) 组成的。

- 基础算术运算符：加 `+`、减 `-`、乘 `*`、除 `/`、取余 `%`。

> [!note]
> Java 中，`/` 遵从整数除法：如果两边都是整数，则返回整数。

- 递增和递减运算符：`++` 和 `--`。

> [!note]
> `j = k++;`：先把 k 当前的值赋给 j，然后 k 再加 1。
>
> `j = ++k;`：k 先把自己加 1，然后再把 k 的新值赋给 j。

- 赋值运算符：`=`、`+=`、`-=` 等。

运算符有明确的优先级，这决定了它们被计算的顺序：

- 高优先级：乘法 `*`、除法 `/` 和取余 `%`。
- 低优先级：加法 `+`、减法 `-` 以及字符串拼接。
- 如果优先级相同，则从左到右计算。
- 括号 `()` 拥有最高优先权，可以强行改变计算顺序。

赋值运算符 `=` 的优先级低于算术运算符。

## 2.3 控制流与循环结构

在 Java 中，条件判断 (*Condition*) 必须严格使用布尔表达式 (*Boolean Expression*)。

- 比较运算符：`<`、`>`、`==`、`<=`、`>=`、`!=`。
- 逻辑运算符：非 `!`、与 `&&`、或 `||`。

> [!note]
> Java 中的 `true` 和 `false` 是独立的布尔值，不能像 Python 那样用非零数字（如 `1`）或非空字符串来代表真。
>
> Java 不支持链式比较 (*Chained Comparisons*)。例如，Python 中的 `a < b == c` 在 Java 中是非法的，必须显式写为 `a < b && b == c`。

#### `if-else` 分支 (*If-Else Branch*)

Java 使用 `if`、`else if` 和 `else` 来构建条件分支，并通过大括号 `{}` 定义作用域 (*Scope*)。

```java
if (a > 0) {
    b = -1;
} else if (a < 0) {
    b = 1;
} else {
    b = 0;
}
```

> [!note]
> 如果作用域内只有一条语句，大括号 `{}` 可以省略。但为了代码的清晰与可维护性，通常建议始终保留大括号。

#### `switch-case` 分支 (*Switch-Case Branch*)

当需要对同一个变量进行多次等值判断时，使用 `switch` 语句比多个 `if-else` 更加清晰。

```java
char grade = 'B';
switch(grade) {
    case 'A':
        System.out.println("Excellent!");
        break;
    case 'B':
        System.out.println("Well done");
        break;
    case 'C':
    case 'D': // 匹配 C 或 D
        System.out.println("You passed");
        break;
    case 'F':
        System.out.println("Better try again");
        break;
    default: // 可选的默认分支
        System.out.println("Invalid grade");
}
```

> [!note]
> 每个 `case` 执行完毕后，通常需要使用 `break;` 来跳出 `switch` 结构。如果不写 `break`，程序会继续向下执行后续的 `case` 代码，即发生穿透现象 (*Fall-through*)。

#### 循环结构 (*Loop Structure*)

Java 提供多种循环结构，其中 `while` 与 `for` 最常用。

`while` 循环适用于"先判断、后执行"的重复场景，只要条件为真，循环体就会持续执行。

```java
while (a > b) {
    a--;
}
```

`for` 循环将初始化 (*Initialization*)、布尔表达式 (*Boolean Expression*) 与更新操作 (*Update*) 集中写在一行，适用于循环次数相对明确的场景。

```java
for (int x = 10; x < 20; x++) {
    System.out.println(x);
}
```

## 2.4 方法与作用域

Java 中的函数依附于类存在，因此通常称为方法 (*Method*)。

方法声明的一般形式如下。
`<修饰符> <返回类型> <方法名> (参数列表) { 方法体 }`

```java
// 无返回值、无参数的方法
public void printHello() {
   System.out.println("hello");
}

// 有返回值、带参数的方法
public int addOne(int x) {
   return x++;
}
```

> [!note]
> `void` 关键字表示该方法不返回任何数据。
> 若声明了具体的返回类型（如 `int`），方法体内必须包含 `return` 语句并返回对应类型的值。

- 修饰符 (*Modifier*)：如 `public`，用于控制方法的访问权限与作用域。
- 参数列表 (*Parameter List*)：必须显式声明每个传入参数的数据类型。

## 2.5 字符串与控制台输出

字符串 (*String*) 是由字符序列构成的对象，使用双引号 `""` 包裹。

- 字符串是对象 (*Object*)，由 `String` 类定义，而非基本数据类型。
- 字符串拼接 (*Concatenation*)：使用 `+` 运算符将两个字符串连接在一起。

```java
"Peanut butter " + "and jelly"
```

Java 使用 `System.out` 对象来向控制台（显示器）发送输出信息。

- `println` 方法：打印传入的信息，并在末尾自动换行 (*Advance to the Next Line*)。
- `print` 方法：仅打印信息，不换行。

```java
// System.out 是对象，println 是方法，括号内是参数
System.out.println("Change has come to America.");
```

转义字符 (*Escape Sequences*) 是一系列以反斜杠 `\` 开头的特殊字符组合，用于表示无法直接输入的字符或控制格式。

- `\b`：退格 (*Backspace*)
- `\t`：制表符 (*Tab*)
- `\n`：换行 (*Newline*)
- `\"`：双引号 (*Double quote*)
- `\'`：单引号 (*Single quote*)
- `\\`：反斜杠 (*Backslash*)

```java
// 使用转义字符在字符串中包含双引号
System.out.println("I said \"Hello\" to you.");
```

## 2.6 数据类型转换

数据类型转换 (*Data Conversion*) 允许将一种类型的值转换为另一种类型，但必须关注精度与取值范围是否会发生变化。

> [!note]
> 类型转换不会改变原变量本身的类型或存储的值，仅在计算过程中临时转换该值。

- 扩宽转换 (*Widening Conversion*)：从较小的数据类型转换为较大的数据类型（如 `short` → `int`），通常是安全的。
- 缩窄转换 (*Narrowing Conversion*)：从较大的数据类型转换为较小的数据类型（如 `int` → `short`），可能造成信息丢失。

> [!question]
>
> 以下各赋值语句中，哪些是扩宽转换，哪些是缩窄转换？
>
> （基本类型大小顺序：`byte` < `short` < `int` < `long` < `float` < `double`，`char` 与 `int` 同级）
>
> ```java
> int n1; long n2; float x1; double x2;
> char c; boolean b;
>
> n1 = n2;  // 缩窄：long → int
> n2 = n1;  // 扩宽：int → long
> x2 = n1;  // 扩宽：int → double
> n1 = c;   // 扩宽：char → int
> c = b;    // 非法：boolean 不能转换为任何其他类型
> ```
>

> [!note]
> `boolean` 类型在 Java 中是完全独立的，不能与任何其他数据类型相互转换，这与 C/C++ 不同。

#### 赋值转换 (*Assignment Conversion*)
赋值转换发生在将一种类型的值赋给另一种类型的变量时。

- 仅允许扩宽转换，不允许缩窄转换。

```java
float money;
int dollars = 10;
money = dollars; // 合法：int → float，扩宽转换
```

> [!note]
> `dollars` 本身的类型和值不会发生任何变化，转换仅作用于赋值过程中的那一刻。

#### 自动类型提升 (*Promotion*)
类型提升发生在表达式的运算过程中，由编译器自动完成。

- 当运算符两侧操作数类型不一致时，较小的类型会自动提升为较大的类型，再进行运算。

```java
float sum = 100;
int count = 10;
float result = sum / count; // count 被自动提升为 float，再做除法
```

> [!note]
> `count` 的类型在提升后仍然是 `int`，类型提升仅在该次运算中临时生效。

#### 强制类型转换 (*Casting*)
强制类型转换是最灵活也是最危险的转换方式，扩宽和缩窄转换均可通过它完成。

- 语法：在目标值前以括号包裹目标类型。

```java
// total 和 count 均为 int，直接除法会执行整数除法
result = (float) total / count; // 将 total 强制转换为 float，再与 count 做浮点除法
```

> [!note]
> 此处 `(float) total / count` 与 `(float) (total / count)` 结果不同。前者先转换 `total` 再除，得到浮点结果；后者先做整数除法再转换，小数部分已丢失。

## 2.7 标准输入与应用示例

读取标准输入 (*Standard Input*) 通常借助 `java.util` 包中的 `Scanner` 类实现。

- 键盘输入源由 `System.in` 对象表示。
- `Scanner scan = new Scanner(System.in);` 用于创建一个绑定标准输入流的扫描器对象。
- `nextLine()` 读取整行输入，直到遇到换行符。
- `nextInt()` 解析并读取下一个整数。
- `nextDouble()` 解析并读取下一个双精度浮点数。

> [!note]
> `Scanner` 类并非 Java 核心基础包 (`java.lang`) 的一部分，使用前必须在文件顶部显式导入 (*Import*)：`import java.util.Scanner;`。

> [!example]- 示例：`GasMileage.java`
> 该程序通过终端交互读取用户输入，并计算燃油效率。
>
> ```java
> import java.util.Scanner;
>
> public class GasMileage
> {
>     public static void main(String[] args)
>     {
>         int miles;
>         double gallons, mpg;
>
>         Scanner scan = new Scanner(System.in);
>
>         System.out.print("Enter the number of miles: ");
>         miles = scan.nextInt();
>
>         System.out.print("Enter the gallons of fuel used: ");
>         gallons = scan.nextDouble();
>
>         mpg = miles / gallons;
>
>         System.out.println("Miles Per Gallon: " + mpg);
>     }
> }
> ```
>
> **导入声明**
> ```java
> import java.util.Scanner;
> ```
> - `Scanner` 属于 `java.util` 包，不在自动导入的 `java.lang` 范围内，必须显式声明。
>
> **变量声明与对象创建**
> ```java
> int miles;
> double gallons, mpg;
> Scanner scan = new Scanner(System.in);
> ```
> - `miles` 为整型，`gallons` 与 `mpg` 为双精度浮点型。
> - `new Scanner(System.in)` 创建一个绑定到标准输入流的 `Scanner` 对象，赋值给 `scan`。
>
> **输入读取**
> ```java
> System.out.print("Enter the number of miles: ");
> miles = scan.nextInt();
>
> System.out.print("Enter the gallons of fuel used: ");
> gallons = scan.nextDouble();
> ```
> - `print` 打印提示语但不换行，使光标停在同一行等待用户输入。
> - `nextInt()` 与 `nextDouble()` 阻塞程序执行，直至用户输入并按下回车。
>
> **核心计算**
> ```java
> mpg = miles / gallons;
> ```
> - `miles` 为 `int`，`gallons` 为 `double`；运算时 `miles` 触发自动类型提升 (*Promotion*) 转为 `double`，执行浮点除法而非整数除法，结果保留小数部分。
>
> **输出**
> ```java
> System.out.println("Miles Per Gallon: " + mpg);
> ```
> - `+` 运算符将字符串字面量与 `double` 型的 `mpg` 拼接，`mpg` 自动转换为字符串。
> - `println` 输出拼接结果并换行。
>
> > [!question]
> > 如果输入 `miles` 时没有输入整数会发生什么？
> >
> > 答：`scan.nextInt()` 无法将非整数格式的输入（如字母或小数）解析为 `int`，程序会直接抛出 `InputMismatchException` 异常并崩溃终止。

> [!example]- 示例：`BankAccount.java`
> 下面的代码给出了一个简单的类定义，用于演示实例变量如何描述状态、方法如何定义行为。
>
> ```java
> public class BankAccount {
>
>     private String accountNumber;
>     private double balance;
>
>     public void deposit(double amount) {
>         balance += amount;
>     }
>
>     public void withdraw(double amount) {
>         if (balance >= amount) {
>             balance -= amount;
>         }
>     }
>
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

---
`Pre: ` [[ELEC2543 Ch.1 Introduction to Java]]
`Post:` [[ELEC2543 Ch.3 Introduction to Classes]]

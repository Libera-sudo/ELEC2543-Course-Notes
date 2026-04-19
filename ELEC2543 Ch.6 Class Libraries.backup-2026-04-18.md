#Y2S2 #ELEC2543
# Chapter 6. 类库

`Pre: ` [[ELEC2543 Ch.5 Data Visibility]]
`Post:` [[ELEC2543 Ch.7 Enumerated Type]]

> [!abstract]
> 类库 (*Class Library*) 是一组可供程序开发时复用的类的集合。Java 标准类库 (*Java Standard Class Library*) 是任何 Java 开发环境的组成部分，其中的类并不属于 Java 语言本身的语法定义，但在实际开发中被广泛依赖。
>
>- 标准类库随 JDK 一同提供，无需额外安装
>- 已使用过的 `System`、`Scanner`、`String` 均属于标准类库
>- 除标准类库外，还可使用第三方类库，或自行编写类库
>
>Java 的类库以包 (*Package*) 为单位组织，并通过 `import` 声明引入。与 Python 的模块系统或 C++ 的头文件机制不同，Java 的 `java.lang` 包被自动导入，其余包须显式声明；同时，Java 的格式化输出依赖专用类而非格式化字符串，体现了面向对象的一贯设计风格。
>
>本章涵盖以下核心模块：包的组织结构与 `import` 声明机制、`String` 类的不可变性与常用方法、`Math` 类与 `Random` 类的使用，以及 `NumberFormat` 与 `DecimalFormat` 类的格式化输出。
```table-of-contents
maxLevel: 3
```
## 6.1 包

Java 标准类库中的类以包 (*Package*) 为单位进行组织。每个包内含一组功能相关的类，开发者可按需引入对应的包。

标准类库中的常见包如下：

| 包名                  | 用途                        |
| ------------------- | ------------------------- |
| `java.lang`         | 通用支持（自动导入）                |
| `java.applet`       | 创建网页 Applet               |
| `java.awt`          | 图形与图形用户界面                 |
| `javax.swing`       | 扩展图形功能                    |
| `java.net`          | 网络通信                      |
| `java.util`         | 工具类（如 `Scanner`、`Random`） |
| `javax.xml.parsers` | XML 文档处理                  |

## 6.2 `import` 声明

使用某个包中的类时，有两种引用方式可供选择：
#### 全限定名与 `import` 声明

全限定名 (*Fully Qualified Name*) 是在类名前加上完整的包路径，可以不经任何声明直接使用：

```java
java.util.Scanner scan = new java.util.Scanner(System.in);
```

更常见的做法是在文件顶部使用 `import` 声明，之后直接使用类名：

```java
import java.util.Scanner;
// 导入 java.util 包下的 Scanner 类
Scanner scan = new Scanner(System.in);
```

若需引入某个包下的所有类，可使用通配符 `*`：

```java
import java.util.*;
```

> [!note]
> 通配符 `*` 仅导入该包下的所有类，不会导入其子包中的类。

#### `java.lang` 的自动导入

`java.lang` 包中的所有类在任何程序中均被自动导入，相当于每个程序隐含以下声明：

```java
import java.lang.*;
```

- `System`、`String`、`Math` 等类均属于 `java.lang`，因此无需显式导入
- `Scanner` 属于 `java.util`，必须显式导入才能使用

## 6.3 `String` 类

`String` 类用于表示字符串，是 Java 中最常用的类之一。

`String` 属于 `java.lang` 包，无需显式导入。

`String` 对象可通过两种方式创建：

```java
String name = new String("ELEC 2543"); // 显式使用 new
String name = "ELEC 2543";             // 字符串字面量，更常用
```

两种方式均会创建一个 `String` 对象，并由变量 `name` 持有其引用。

`String` 对象是不可变的 (*Immutable*)：一旦创建，其内容与长度均不能被修改。`String` 类提供的方法不会修改原对象，而是返回一个新的 `String` 对象。

```java
String name = "ELEC 2543";
name = name.replace('E', 'X'); // 原对象不变，name 现在指向新对象 "XLXC 2543"
```

- 原字符串对象 `"ELEC 2543"` 仍然存在于内存中
- `name` 变量被重新赋值，指向新创建的字符串对象 `"XLXC 2543"`

字符串中每个字符对应一个从 0 开始的整数索引，可通过 `charAt(index)` 方法访问：

```java
string.charAt(index);
```

- 字符串 `"Hello"` 中，`'H'` 的索引为 0，`'o'` 的索引为 4
- 索引从 0 开始，最大索引为字符串长度减 1

> [!note]
> 若传入的索引超出范围（小于 0 或大于等于字符串长度），将**抛出** `StringIndexOutOfBoundsException`。
> > [!quote]
>> 在 Java 中，**抛出** (_Throw_) 指程序执行过程中遇到无法继续正常运行的错误条件时，中断当前流程并生成一个**异常对象** (_Exception Object_) 的行为。

> [!example]- 示例：`StringMutation.java`
> 演示 `String` 类的常用方法及字符串的"变异"操作。
>
> ```java
> public class StringMutation
> {
>    public static void main (String[] args)
>    {
> 	   // 原始字符串及四个变异结果变量
>       String phrase = "Change is inevitable";
>       String mutation1, mutation2, mutation3, mutation4;
>
> 	  // 输出原始字符串及其长度
>       System.out.println ("Original string: \"" + phrase + "\"");
>       System.out.println ("Length of string: " + phrase.length());
>
> 	  // 链式字符串操作：拼接 → 转大写 → 替换 → 截取子串
>       mutation1 = phrase.concat (", except from vending machines.");
>       mutation2 = mutation1.toUpperCase();
>       mutation3 = mutation2.replace ('E', 'X');
>       mutation4 = mutation3.substring (3, 30);
>
>       // 输出各阶段变异结果
>       System.out.println ("Mutation #1: " + mutation1);
>       System.out.println ("Mutation #2: " + mutation2);
>       System.out.println ("Mutation #3: " + mutation3);
>       System.out.println ("Mutation #4: " + mutation4);
>
>       System.out.println ("Mutated length: " + mutation4.length());
>    }
> }
> ```
>
> **变量初始化**
> ```java
> String phrase = "Change is inevitable";
> String mutation1, mutation2, mutation3, mutation4;
> ```
> - `phrase` 为原始字符串，后续所有操作均以此为起点
> - `mutation1` 至 `mutation4` 声明为四个字符串变量，用于存放各步骤的结果
>
> **字符串方法链式调用**
> ```java
> mutation1 = phrase.concat(", except from vending machines.");
> mutation2 = mutation1.toUpperCase();
> mutation3 = mutation2.replace('E', 'X');
> mutation4 = mutation3.substring(3, 30);
> ```
> - `concat(String s)`：将参数字符串拼接至原字符串末尾，返回新字符串
> - `toUpperCase()`：将所有字母转换为大写，返回新字符串
> - `replace(char old, char new)`：将所有指定字符替换为新字符，返回新字符串
> - `substring(int begin, int end)`：返回索引 `begin`（含）至 `end`（不含）之间的子字符串
>
> **输出**
> ```java
> System.out.println("Mutation #1: " + mutation1);
> // ...
> System.out.println("Mutated length: " + mutation4.length());
> ```
> - 每步变异结果独立输出，便于观察各方法的效果
> - `length()` 返回字符串的字符数（即长度）
>
> **关于创建了多少个 `String` 对象**
> - `phrase`：1 个
> - `mutation1` 至 `mutation4`：各 1 个，共 4 个
> - 输出语句中的字符串字面量（如 `"Original string: \""` 等）：每个字面量对应 1 个对象
> - 字符串拼接（`+` 运算符）在运行时还会产生额外的临时 `String` 对象
>
> 因此，程序实际创建的 `String` 对象数量远多于显式声明的 5 个。

## 6.4 `Math` 类

`Math` 类属于 `java.lang` 包，提供常用的数学运算方法。`Math` 类中的所有方法均为**静态方法** (*Static Method*)，也称类方法 (*Class Method*)。

静态方法通过类名直接调用，无需创建该类的对象：

```java
value = Math.cos(90) + Math.sqrt(delta); // value 等于 cos90° 加 sqrt(delta)
```

- `Math.cos(double a)`：返回角度 $a$（单位为**弧度**）的余弦值
- `Math.sqrt(double a)`：返回 $a$ 的平方根
- `Math.abs(double a)`：返回 $a$ 的绝对值
- `Math.pow(double a, double b)`：返回 $a^b$
- `Math.PI`：常量 $\pi \approx 3.141592653589793$
- `Math.random()`：返回 $[0.0, 1.0)$ 范围内的随机 `double` 值

> [!note]
> 静态方法的详细机制将在后续章节进一步讨论。此处只需掌握：调用 `Math` 类的方法时，使用类名而非对象名。

## 6.5 `Random` 类

`Random` 类属于 `java.util` 包，提供生成各类随机数的方法。与 `Math` 类不同，`Random` 类需要先**创建对象**，再通过对象调用方法：

```java
Random generator = new Random();
```

常用方法如下：

```java
// 返回 int 范围内的任意随机整数
num1 = generator.nextInt();

// 返回 [0, 10) 范围内的随机整数，即 0 到 9
num1 = generator.nextInt(10);

// 返回 [0.0, 1.0) 范围内的随机浮点数
num2 = generator.nextFloat();
```

- `nextInt(int bound)` 的返回值范围为 $[0, \text{bound})$，**不包含** `bound` 本身
- `nextFloat()` 返回类型为 `float`，若需要 `double` 类型可使用 `nextDouble()`
- 使用前须在文件顶部添加 `import java.util.Random;`

> [!note]
> 若需生成 $[a, b)$ 范围内的随机整数，可使用：
> ```java
> num = generator.nextInt(b - a) + a;
> ```

## 6.6 格式化输出

Java 提供 `NumberFormat` 与 `DecimalFormat` 两个类用于格式化数值输出。二者均定义于 `java.text` 包，需显式导入。

#### `NumberFormat` 类

`NumberFormat` 类通过静态工厂方法（而非 `new`）创建实例：

```java
NumberFormat fmt1 = NumberFormat.getCurrencyInstance(); // 货币格式
NumberFormat fmt2 = NumberFormat.getPercentInstance();  // 百分比格式
```

- `getCurrencyInstance()`：按本地货币格式输出，如 `HKD12.00`
- `getPercentInstance()`：按百分比格式输出，如 `6%`
- `format(double number)`：将数值按格式对象的模式转换为字符串并返回

> [!example]- 示例：`Purchase.java`
> 演示 `NumberFormat` 类的货币与百分比格式化输出。
>
> ```java
> import java.util.Scanner;
> import java.text.NumberFormat;
>
> public class Purchase
> {
>     public static void main(String[] args) {
>
>         final double TAX_RATE = 0.06;
>
>         int quantity;
>         double subtotal, tax, totalCost, unitPrice;
>
>         Scanner scan = new Scanner(System.in);
>
>         NumberFormat fmt1 = NumberFormat.getCurrencyInstance();
>         NumberFormat fmt2 = NumberFormat.getPercentInstance();
>
>         System.out.print("Enter the quantity: ");
>         quantity = scan.nextInt();
>
>         System.out.print("Enter the unit price: ");
>         unitPrice = scan.nextDouble();
>
>         subtotal = quantity * unitPrice;
>         tax = subtotal * TAX_RATE;
>         totalCost = subtotal + tax;
>
>         System.out.println("Subtotal: " + fmt1.format(subtotal));
>         System.out.println("Tax: " + fmt1.format(tax) + " at " + fmt2.format(TAX_RATE));
>         System.out.println("Total: " + fmt1.format(totalCost));
>     }
> }
> ```
>
> 输出（以输入 quantity=3、unitPrice=4 为例）：
> ```
> Enter the quantity: 3
> Enter the unit price: 4
> Subtotal: HKD12.00
> Tax: HKD0.72 at 6%
> Total: HKD12.72
> ```
>
> **导入声明与变量初始化**
> ```java
> import java.util.Scanner;
> import java.text.NumberFormat;
>
> final double TAX_RATE = 0.06;
> int quantity;
> double subtotal, tax, totalCost, unitPrice;
> ```
> - `TAX_RATE` 声明为 `final`，表示税率为不可修改的常量
> - `subtotal`、`tax`、`totalCost` 用于存放计算的中间结果与最终结果
>
> **创建格式对象**
> ```java
> NumberFormat fmt1 = NumberFormat.getCurrencyInstance();
> NumberFormat fmt2 = NumberFormat.getPercentInstance();
> ```
> - `fmt1` 用于货币格式，`fmt2` 用于百分比格式
> - 两者均通过静态工厂方法创建，不使用 `new`
>
> **核心计算逻辑**
> ```java
> subtotal = quantity * unitPrice;
> tax = subtotal * TAX_RATE;
> totalCost = subtotal + tax;
> ```
> - 依次计算小计、税额与总价，逻辑清晰，各步结果存入独立变量
>
> **格式化输出**
> ```java
> System.out.println("Subtotal: " + fmt1.format(subtotal));
> System.out.println("Tax: " + fmt1.format(tax) + " at " + fmt2.format(TAX_RATE));
> System.out.println("Total: " + fmt1.format(totalCost));
> ```
> - `fmt1.format()` 将 `double` 值转换为货币格式字符串
> - `fmt2.format(TAX_RATE)` 将 `0.06` 转换为 `6%`
#### `DecimalFormat` 类

`DecimalFormat` 类通过构造函数接收一个格式模式字符串，必须使用 `new` 创建实例：

```java
DecimalFormat fmt = new DecimalFormat("0.###");
System.out.println("The circle's area: " + fmt.format(area));
```

格式模式中的常用符号：
- `0`：该位置若无数字则补 `0`
- `#`：该位置若无数字则省略（不补 `0`）
- `.`：小数点位置

> [!note]
> `"0.###"` 表示整数部分至少显示一位，小数部分最多显示三位，末尾的零不保留。例如 `3.1400` 将输出为 `3.14`。

> [!example]- 例题：输出 PI 的格式化值
> 编写代码片段，使用 `DecimalFormat` 输出 `"The value of PI is 3.1416."`，其中 `3.1416` 由 `Math.PI` 经格式化方法得到。
>
> ```java
> import java.text.DecimalFormat;
>
> DecimalFormat fmt = new DecimalFormat("0.####");
> System.out.println("The value of PI is " + fmt.format(Math.PI) + ".");
> ```
>
> **格式模式分析**
> ```java
> new DecimalFormat("0.####")
> ```
> - 整数部分 `0`：至少保留一位，`Math.PI` 整数部分为 `3`，正常显示
> - 小数部分 `####`：最多保留四位，末尾零省略
> - `Math.PI ≈ 3.141592653...`，保留四位小数后四舍五入得 `3.1416`
>
> **调用 `format` 方法**
> ```java
> fmt.format(Math.PI)
> ```
> - `Math.PI` 为 `java.lang` 包中 `Math` 类的静态常量，无需导入即可使用
> - `format()` 返回 `String` 类型，可直接与其他字符串拼接

---
`Pre: ` [[ELEC2543 Ch.5 Data Visibility]]
`Post:` [[ELEC2543 Ch.7 Enumerated Type]]

#Y2S2 #ELEC2543
# Chapter 6. 类库

`Pre: ` [[ELEC2543 Ch.5 Data Visibility]]
`Post:` [[ELEC2543 Ch.7 Enumerated Type]]

> [!abstract]
> 类库 (*Class Library*) 是一组可供程序复用的预定义类集合，是 Java 程序从语言语法走向实际开发环境的关键桥梁。
>
> Java 将大量常用能力组织为标准类库 (*Java Standard Class Library*) 并随 JDK 一同提供。与将许多常用能力视为语言内建或依赖格式化字符串的语言不同，Java 更倾向于通过包 (*Package*)、类与对象方法组织输入输出、随机数、数学运算与格式化功能，其中 `java.lang` 自动导入，其余包则通过 `import` 显式引入。
>
> 本章围绕类库的基本使用展开，依次介绍包与 `import` 声明、`String` 类的不可变性与常用操作、`Math` 与 `Random` 类的调用方式，以及 `NumberFormat` 和 `DecimalFormat` 的格式化输出机制。
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

`System`、`String`、`Math` 等类均属于 `java.lang`，因此无需显式导入；`Scanner` 属于 `java.util`，必须显式导入才能使用。

## 6.3 `String` 类

`String` 类用于表示字符串，是 Java 中最常用的类之一。它属于 `java.lang` 包，因此无需显式导入。

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

原字符串对象 `"ELEC 2543"` 仍然存在于内存中；被重新赋值的是变量 `name`，它改为指向新创建的 `"XLXC 2543"` 对象。

字符串中每个字符对应一个从 0 开始的整数索引，其访问形式为：

```java
<字符串>.charAt(<索引>)
```

例如，在 `"Hello"` 中，`'H'` 的索引为 0，`'o'` 的索引为 4；最大合法索引等于字符串长度减 1。

> [!note]
> 若传入的索引超出范围（小于 0 或大于等于字符串长度），将**抛出** `StringIndexOutOfBoundsException`。
>
> > [!quote]
> > 在 Java 中，**抛出** (*Throw*) 指程序执行过程中遇到无法继续正常运行的错误条件时，中断当前流程并生成一个**异常对象** (*Exception Object*) 的行为。

> [!example]- 示例：`StringMutation.java`
> 观察同一字符串在拼接、转大写、字符替换与子串截取后的结果变化。
>
> ```java
> public class StringMutation
> {
>     public static void main(String[] args)
>     {
>         // 原始字符串与各阶段结果
>         String phrase = "Change is inevitable";
>         String mutation1, mutation2, mutation3, mutation4;
>
>         // 输出原始字符串与长度
>         System.out.println("Original string: \"" + phrase + "\"");
>         System.out.println("Length of string: " + phrase.length());
>
>         // 依次执行拼接、转大写、字符替换与子串截取
>         mutation1 = phrase.concat(", except from vending machines.");
>         mutation2 = mutation1.toUpperCase();
>         mutation3 = mutation2.replace('E', 'X');
>         mutation4 = mutation3.substring(3, 30);
>
>         // 输出每一步变换结果
>         System.out.println("Mutation #1: " + mutation1);
>         System.out.println("Mutation #2: " + mutation2);
>         System.out.println("Mutation #3: " + mutation3);
>         System.out.println("Mutation #4: " + mutation4);
>
>         System.out.println("Mutated length: " + mutation4.length());
>     }
> }
> ```
>
> 输出：
>
> ```
> Original string: "Change is inevitable"
> Length of string: 20
> Mutation #1: Change is inevitable, except from vending machines.
> Mutation #2: CHANGE IS INEVITABLE, EXCEPT FROM VENDING MACHINES.
> Mutation #3: CHANGX IS INXVITABLX, XXCXPT FROM VXNDING MACHINXS.
> Mutation #4: NGX IS INXVITABLX, XXCXPT F
> Mutated length: 27
> ```
>
> **变量初始化**
>
> ```java
> String phrase = "Change is inevitable";
> String mutation1, mutation2, mutation3, mutation4;
> ```
> - `phrase` 为原始字符串，后续所有操作均以此为起点。
> - `mutation1` 至 `mutation4` 声明为四个字符串变量，用于存放各步骤的结果。
>
> **字符串变换过程**
>
> ```java
> mutation1 = phrase.concat(", except from vending machines.");
> mutation2 = mutation1.toUpperCase();
> mutation3 = mutation2.replace('E', 'X');
> mutation4 = mutation3.substring(3, 30);
> ```
> - `concat(String s)` 将参数字符串拼接至原字符串末尾，并返回新字符串。
> - `toUpperCase()` 将所有字母转换为大写，`replace(char old, char new)` 将指定字符替换为新字符。
> - `substring(int begin, int end)` 返回索引 `begin`（含）至 `end`（不含）之间的子字符串。
>
> **输出与长度观察**
>
> ```java
> System.out.println("Original string: \"" + phrase + "\"");
> System.out.println("Length of string: " + phrase.length());
> System.out.println("Mutation #1: " + mutation1);
> System.out.println("Mutation #2: " + mutation2);
> System.out.println("Mutation #3: " + mutation3);
> System.out.println("Mutation #4: " + mutation4);
> System.out.println("Mutated length: " + mutation4.length());
> ```
> - 原字符串长度为 20，而截取后的 `mutation4` 长度为 27，说明各步操作产生的是新的字符串结果。
> - 各阶段结果分别输出，便于对照每个方法对字符串内容产生的具体影响。
>
> **对象创建数量**
>
> - `phrase`：1 个。
> - `mutation1` 至 `mutation4`：各 1 个，共 4 个。
> - 输出语句中的字符串字面量（如 `"Original string: \""` 等）：每个字面量对应 1 个对象。
> - 字符串拼接（`+` 运算符）在运行时还会产生额外的临时 `String` 对象。
>
> 因此，程序实际创建的 `String` 对象数量远多于显式声明的 5 个。

## 6.4 `Math` 类

`Math` 类属于 `java.lang` 包，提供常用的数学运算方法。`Math` 类中的所有方法均为**静态方法** (*Static Method*)，也称类方法 (*Class Method*)。

静态方法通过类名直接调用，无需创建该类的对象：

```java
value = Math.cos(angle) + Math.sqrt(delta); // 计算 cos(angle) 与 sqrt(delta) 的和
```

- `Math.cos(double a)`：返回参数 `a`（单位为**弧度**）的余弦值
- `Math.sqrt(double a)`：返回 $a$ 的平方根
- `Math.abs(double a)`：返回 $a$ 的绝对值
- `Math.pow(double a, double b)`：返回 $a^b$
- `Math.PI`：常量 $\pi \approx 3.141592653589793$
- `Math.random()`：返回 $[0.0, 1.0)$ 范围内的随机 `double` 值

> [!note]
> 静态方法的详细机制将在后续章节进一步讨论。此处只需掌握：调用 `Math` 类的方法时，使用类名而非对象名。

## 6.5 `Random` 类

`Random` 类属于 `java.util` 包，提供生成各类随机数的方法。与 `Math` 直接提供静态方法不同，`Random` 通过对象维护随机数序列状态，因此使用前需先**创建对象**：

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

- `getCurrencyInstance()`：按本地货币格式输出，如 `HK$12.00`
- `getPercentInstance()`：按百分比格式输出，如 `6%`
- `format(double number)`：将数值按格式对象的模式转换为字符串并返回

> [!note]
> `getCurrencyInstance()` 与 `getPercentInstance()` 的具体输出形式依默认区域设置 (*Locale*) 而定，货币符号、分组方式与小数显示可能随运行环境不同而变化。

> [!example]- 示例：`Purchase.java`
> 观察同一笔购买在货币格式与百分比格式下的输出结果。
>
> ```java
> import java.util.Scanner;
> import java.text.NumberFormat;
>
> public class Purchase
> {
>     public static void main(String[] args) {
>         // 税率常量
>         final double TAX_RATE = 0.06;
>
>         // 购买数量、单价、小计、税额与总价
>         int quantity;
>         double subtotal, tax, totalCost, unitPrice;
>
>         // 读取用户输入
>         Scanner scan = new Scanner(System.in);
>
>         // 创建货币与百分比格式对象
>         NumberFormat fmt1 = NumberFormat.getCurrencyInstance();
>         NumberFormat fmt2 = NumberFormat.getPercentInstance();
>
>         // 读取购买数量
>         System.out.print("Enter the quantity: ");
>         quantity = scan.nextInt();
>
>         // 读取商品单价
>         System.out.print("Enter the unit price: ");
>         unitPrice = scan.nextDouble();
>
>         // 计算小计、税额与总价
>         subtotal = quantity * unitPrice;
>         tax = subtotal * TAX_RATE;
>         totalCost = subtotal + tax;
>
>         // 按指定格式输出结果
>         System.out.println("Subtotal: " + fmt1.format(subtotal));
>         System.out.println("Tax: " + fmt1.format(tax) + " at " + fmt2.format(TAX_RATE));
>         System.out.println("Total: " + fmt1.format(totalCost));
>     }
> }
> ```
>
> 输出（以 Hong Kong 区域设置、输入 `quantity = 3`、`unitPrice = 4` 为例）：
> ```
> Enter the quantity: 3
> Enter the unit price: 4
> Subtotal: HK$12.00
> Tax: HK$0.72 at 6%
> Total: HK$12.72
> ```
>
> **导入声明与变量初始化**
>
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
> **读取输入**
>
> ```java
> Scanner scan = new Scanner(System.in);
>
> System.out.print("Enter the quantity: ");
> quantity = scan.nextInt();
>
> System.out.print("Enter the unit price: ");
> unitPrice = scan.nextDouble();
> ```
> - `Scanner` 从标准输入读取购买数量与单价，分别存入 `quantity` 与 `unitPrice`。
> - `print()` 先输出提示词，再等待用户输入，因此控制台中的提示与输入会连续出现。
>
> **创建格式对象**
>
> ```java
> NumberFormat fmt1 = NumberFormat.getCurrencyInstance();
> NumberFormat fmt2 = NumberFormat.getPercentInstance();
> ```
> - `fmt1` 用于货币格式，`fmt2` 用于百分比格式
> - 两者均通过静态工厂方法创建，不使用 `new`
>
> **核心计算逻辑**
>
> ```java
> subtotal = quantity * unitPrice;
> tax = subtotal * TAX_RATE;
> totalCost = subtotal + tax;
> ```
> - 依次计算小计、税额与总价，逻辑清晰，各步结果存入独立变量
>
> **格式化输出**
>
> ```java
> System.out.println("Subtotal: " + fmt1.format(subtotal));
> System.out.println("Tax: " + fmt1.format(tax) + " at " + fmt2.format(TAX_RATE));
> System.out.println("Total: " + fmt1.format(totalCost));
> ```
> - `fmt1.format()` 将 `double` 值转换为货币格式字符串
> - `fmt2.format(TAX_RATE)` 将 `0.06` 转换为 `6%`

#### `DecimalFormat` 类

`DecimalFormat` 类通过构造函数接收一个格式模式字符串。其创建形式为：

```java
new DecimalFormat("<格式模式>")
```

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

> [!question] 习题：输出 PI 的格式化值
> 编写代码片段，使用 `DecimalFormat` 输出 `The value of PI is 3.1416.`，其中 `3.1416` 由 `Math.PI` 经格式化得到。
>
> > [!check]-
> > 可先创建格式对象，再对 `Math.PI` 调用 `format()` 并与说明文字拼接输出。
> >
> > ```java
> > import java.text.DecimalFormat;
> >
> > DecimalFormat fmt = new DecimalFormat("0.####"); // 最多保留四位小数
> > System.out.println("The value of PI is " + fmt.format(Math.PI) + ".");
> > ```
> >
> > 输出：
> > ```
> > The value of PI is 3.1416.
> > ```
> >
> > **格式模式**
> >
> > ```java
> > new DecimalFormat("0.####")
> > ```
> > - 整数部分 `0` 保证至少显示一位。
> > - 小数部分 `####` 表示最多保留四位，末尾零会被省略。
> > - `Math.PI ≈ 3.141592653...`，按该模式格式化后得到 `3.1416`。
> >
> > **调用 `format()`**
> >
> > ```java
> > fmt.format(Math.PI)
> > ```
> > - `Math.PI` 是 `Math` 类的静态常量，可直接通过类名访问。
> > - `format()` 返回 `String`，因此可直接用于字符串拼接输出。

---
`Pre: ` [[ELEC2543 Ch.5 Data Visibility]]
`Post:` [[ELEC2543 Ch.7 Enumerated Type]]

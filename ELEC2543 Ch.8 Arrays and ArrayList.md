 #Y2S2 #ELEC2543
# Chapter 8. 数组与动态列表

`Pre: ` [[ELEC2543 Ch.7 Enumerated Type]]
`Post:` [[ELEC2543 Ch.9 Method]]

> [!abstract]
> 数组 (*Array*) 与动态列表 (*ArrayList*) 是 Java 中最基础的线性数据结构，也是后续讨论数据组织、对象集合与容器类的起点。
>
> Java 将数组设计为对象，必须通过 `new` 创建且长度固定；`ArrayList` 则提供可变长度与更丰富的增删查操作，用于弥补原生数组在插入、删除与容量管理上的局限。这一区分体现了 Java 在固定结构与动态容器之间的分层设计。
>
> 本章围绕数组的声明与初始化、索引访问与边界检查、对象数组、二维数组，以及 `ArrayList` 的泛型声明与常用操作展开，重点比较固定长度数组与动态列表在表示方式与使用场景上的差异。
```table-of-contents
maxLevel: 3
```
## 8.1 数组基础

数组 (*Array*) 是一个有序的值列表，整个数组共享同一名称，每个值通过数字索引访问。大小为 $N$ 的数组，其索引范围为 $0$ 到 $N-1$。

以数组 `scores` 为例：

```
        index:   [0]   [1]   [2]   [3]   [4]   [5]   [6]   [7]
               +-----+-----+-----+-----+-----+-----+-----+-----+
scores ------> |  79 |  87 |  94 |  82 |  67 |  98 |  87 |  81 |
               +-----+-----+-----+-----+-----+-----+-----+-----+

该数组包含 8 个整数值，索引范围为 0 到 7
```

数组中存储的每个值称为数组元素 (*Array Element*)，所有元素必须具有同一元素类型 (*Element Type*)。元素类型既可以是基本类型，如 `int` 、`double` ，也可以是对象引用，如 `String` 或自定义类；数组本身则始终是对象，因此必须通过 `new` 实例化。

数组的声明语法有两种等价形式：`int[] scores` 或 `int scores[]` 。课程笔记统一采用前一种写法，因为类型与变量名的归属更清晰。

```java
int[] scores;   // 推荐写法
int scores[];   // 合法但不推荐
```

数组的实例化写法为：

```java
scores = new int[10];
```

```java
scores = new int[10];  // 创建容量为 10 的整型数组
```

声明与实例化也可合并为一行：

```java
float[] prices = new float[500];
char[]  codes  = new char[1750];
```

> [!note]
> `int scores[10];` 在 Java 中是非法写法。数组大小只能在 `new` 表达式中指定，不能写在声明的类型或变量名后。这与 C/C++ 的语法不同。

若初始值已知，可使用初始化列表 (*Initializer List*) 直接赋值，此时无需 `new` ，数组大小由列表元素数量自动确定：

```java
char[] vowels = {'A', 'E', 'I', 'O', 'U'};  // 大小自动为 5
```

访问某个元素时，使用数组名加方括号内的索引：

```java
scores[2]   // 引用第 3 个元素（索引从 0 开始）
```

该表达式代表数组中的一个具体存储位置，因此可以出现在赋值、运算与传参等一切需要该元素类型值的位置。

遍历数组有两种方式。`for-each` 循环语法更简洁，适用于从头到尾顺序处理所有元素的场景；传统 `for` 循环则在需要访问索引时使用：

```java
// for-each 循环（无法访问索引）
for (int score : scores)
    System.out.println(score);

// 传统 for 循环（可访问索引）
for (int i = 0; i < 10; i++)
    System.out.println(scores[i]);
```

> [!note]
> `for-each` 循环又称增强型 `for` 循环 (*Enhanced for Loop*)，仅适用于从最低索引到最高索引顺序遍历全部元素的场景。
>
> 若需要逆序遍历、跳步访问或修改特定位置的元素，必须使用传统 `for` 循环。

> [!example]- 示例：`BasicArray.java`
>
> ^basicarray
>
> 演示数组的声明、初始化、元素修改与遍历输出。
> ```java
> public class BasicArray {
>     public static void main(String[] args) {
>         final int LIMIT = 15, MULTIPLE = 10;
>
>         int[] list = new int[LIMIT]; // 声明并实例化大小为 15 的整型数组
>
>         // 用循环初始化数组元素
>         for (int index = 0; index < LIMIT; index++)
>             list[index] = index * MULTIPLE;
>
>         list[5] = 999; // 修改索引 5 处的元素
>
>         // 用 for-each 循环打印所有元素
>         for (int value : list)
>             System.out.print(value + " ");
>     }
> }
> ```
> 输出：
> ```
> 0 10 20 30 40 999 60 70 80 90 100 110 120 130 140
> ```
> **变量初始化**
> ```java
> final int LIMIT = 15, MULTIPLE = 10;
> int[] list = new int[LIMIT];
> ```
> - `LIMIT` 与 `MULTIPLE` 声明为 `final`，作为常量控制数组大小与步长，避免魔法数字。
> - `new int[LIMIT]` 创建大小为 15 的数组，索引范围为 0 到 14，所有元素默认初始化为 0。
>
> **初始化循环**
> ```java
> for (int index = 0; index < LIMIT; index++)
>     list[index] = index * MULTIPLE;
> ```
> - 依次将 `list[0]` 到 `list[14]` 赋值为 0、10、20、…、140。
> - 循环条件为 `index < LIMIT`（即 `< 15`），确保不越界访问。
>
> **元素修改**
> ```java
> list[5] = 999;
> ```
> - 直接通过索引覆盖 `list[5]` 原有值 50，替换为 999。
>
> **输出**
> ```java
> for (int value : list)
>     System.out.print(value + " ");
> ```
> - 使用 `for-each` 循环顺序打印全部 15 个元素，元素间以空格分隔。

> [!question]
> 声明一个用于表示 $100$ 名儿童年龄的数组，并编写代码打印整数数组 `values` 的每个元素。
>
> > [!check]-
> > 可直接写作：
> >
> > ```java
> > // 声明并实例化可存放 100 个年龄值的整型数组
> > int[] ages = new int[100];
> >
> > // 顺序打印数组 values 的每个元素
> > for (int value : values)
> >     System.out.println(value);
> > ```
> >
> > `ages` 的元素类型为 `int`，大小为 $100$，索引范围为 $0$ 到 $99$。打印部分使用增强型 `for` 循环顺序访问 `values` 中的每个元素，因此不需要显式书写索引变量。

## 8.2 边界检查与 `length`

数组一旦创建，其大小便固定不变。Java 运行时会对每次数组访问自动执行边界检查 (*Bounds Checking*)：若使用的索引超出有效范围，解释器将抛出 `ArrayIndexOutOfBoundsException`。

每个数组对象都有一个公开常量 `length`，存储该数组的元素个数，通过数组名直接访问：

```java
scores.length  // 返回数组 scores 的元素个数
```

> [!note]
> `length` 存储的是元素总数，而非最大合法索引。大小为 $N$ 的数组，`length` 值为 $N$，最大合法索引为 $N-1$。
>
> 因此传统 `for` 循环的条件应写为 `index < scores.length`，而非 `index <= scores.length`。

> [!example]- 示例：`ReverseOrder.java`
> 演示使用 `length` 控制循环边界，并实现数组的逆序输出。
> ```java
> import java.util.Scanner;
>
> public class ReverseOrder {
> 	// 从用户处读取数字列表，将它们存储在数组中，然后以相反的顺序打印它们。
>     public static void main(String[] args) {
>         Scanner scan = new Scanner(System.in);
>
>         double[] numbers = new double[10]; // 声明大小为 10 的 double 数组
>
>         System.out.println("The size of the array: " + numbers.length);
>
>         // 依次读入 10 个数字
>         for (int index = 0; index < numbers.length; index++) {
>             System.out.print("Enter number " + (index + 1) + ": ");
>             numbers[index] = scan.nextDouble();
>         }
>
>         // 从最高索引向最低索引逆序打印
>         System.out.println("The numbers in reverse order:");
>         for (int index = numbers.length - 1; index >= 0; index--)
>             System.out.print(numbers[index] + " ");
>     }
> }
> ```
> 输出（示例输入）：
> ```
> The size of the array: 10
> Enter number 1: 18.36
> ...
> Enter number 10: 99.18
> The numbers in reverse order:
> 99.18 69.0 45.55 63.41 34.8 72.404 29.06 53.5 48.9 18.36
> ```
> **变量初始化**
> ```java
> double[] numbers = new double[10];
> ```
> - 创建大小为 $10$ 的 `double` 数组，索引范围为 $0$ 到 $9$，元素默认初始化为 $0.0$。
>
> **正序读入**
> ```java
> for (int index = 0; index < numbers.length; index++) {
>     numbers[index] = scan.nextDouble();
> }
> ```
> - 使用 `numbers.length` 而非硬编码的 $10$ 作为上界，便于日后修改数组大小时只需改一处。
>
> **逆序输出**
> ```java
> for (int index = numbers.length - 1; index >= 0; index--)
>     System.out.print(numbers[index] + " ");
> ```
> - 循环从索引 $9$（即 `numbers.length - 1`）开始，每次递减，直到索引 $0$。
> - 此场景需要访问具体索引，因此不能使用 `for-each` 循环。

> [!example]- 示例：`LetterCount.java`
> 演示数组下标与字符 ASCII 值的算术关系，统计字符串中每个字母的大小写出现次数。
> ```java
> import java.util.Scanner;
>
> public class LetterCount {
> 	// 读取用户的句子并计算其中包含的大写和小写字母的数量。
>     public static void main(String[] args) {
>         final int NUMCHARS = 26;
>         Scanner scan = new Scanner(System.in);
>
>         int[] upper = new int[NUMCHARS]; // 统计 A-Z 各字母出现次数
>         int[] lower = new int[NUMCHARS]; // 统计 a-z 各字母出现次数
>
>         char current;  // 当前处理的字符
>         int other = 0; // 非字母字符计数
>
>         System.out.println("Enter a sentence:");
>         String line = scan.nextLine();
>
>         // 遍历字符串中每个字符并分类计数
>         for (int ch = 0; ch < line.length(); ch++) {
>             current = line.charAt(ch);
>             if (current >= 'A' && current <= 'Z')
>                 upper[current - 'A']++;       // 大写字母映射到索引 0-25
>             else if (current >= 'a' && current <= 'z')
>                 lower[current - 'a']++;       // 小写字母映射到索引 0-25
>             else
>                 other++;
>         }
>
>         // 打印每个字母的大小写计数
>         for (int letter = 0; letter < upper.length; letter++) {
>             System.out.print((char)(letter + 'A'));       // 还原大写字母
>             System.out.print(": " + upper[letter]);
>             System.out.print("\t\t" + (char)(letter + 'a')); // 还原小写字母
>             System.out.println(": " + lower[letter]);
>         }
>
>         System.out.println();
>         System.out.println("Non-alphabetic characters: " + other);
>     }
> }
> ```
> 输出（输入：`In Casablanca, Humphrey Bogart never says "Play it again, Sam."`）：
> ```
> A: 0    a: 10
> B: 1    b: 1
> ...
> Non-alphabetic characters: 14
> ```
> **变量初始化**
> ```java
> int[] upper = new int[NUMCHARS];
> int[] lower = new int[NUMCHARS];
> ```
> - 两个大小为 $26$ 的整型数组，元素默认为 $0$，分别对应字母表中 $26$ 个大写与小写字母的计数。
>
> **字符到索引的映射**
> ```java
> upper[current - 'A']++;
> lower[current - 'a']++;
> ```
> - 字符在 Java 中可参与算术运算，其值为对应的 Unicode（ASCII）码。
> - `'A'` 的值为 $65$，`'Z'` 为 $90$；`current - 'A'` 将大写字母映射到 $0$ – $25$，恰好对应数组索引。
> - 例如 `'C' - 'A'` $= 67 - 65 = 2$，即 `upper[2]` 统计字母 C 的出现次数。
>
> **索引还原为字符**
> ```java
> (char)(letter + 'A')
> (char)(letter + 'a')
> ```
> - 将整型索引 $0$ – $25$ 加上 `'A'` 或 `'a'` 的 ASCII 值，再强制转换为 `char`，还原出对应字母。

## 8.3 对象数组

对象数组 (*Array of Objects*) 的元素存储的是对象引用，而非对象本身。声明并实例化一个对象数组时，只分配了存储引用的空间，数组中每个位置初始值为 `null`，各对象需单独实例化后才能存入数组。

```java
String[] words = new String[5]; // 仅创建 5 个 null 引用的槽位，未创建任何 String 对象
```

> [!note]
> 此时访问 `words[0]` 不会抛出 `ArrayIndexOutOfBoundsException`（索引 $0$ 在合法范围内），但对 `null` 引用调用方法（如 `words[0].length()`）会抛出 `NullPointerException`。直接打印 `words[0]` 则输出 `null`。

> [!note]
> 基本类型与对象引用在数组中的默认值如下：
>
> | 元素类型                        | 默认值             |
> | --------------------------- | --------------- |
> | `byte`、`short`、`int`、`long` | $0$             |
> | `float`、`double`            | $0.0$           |
> | `char`                      | `'\u0000'`（空字符） |
> | `boolean`                   | `false`         |
> | 对象引用（如 `String`、自定义类）       | `null`          |
>

各对象需通过 `new` 单独实例化并赋值给对应索引：

```java
words[0] = new String("friendship");
words[1] = new String("loyalty");
words[2] = new String("honor");
// words[3] 与 words[4] 仍为 null
```

> [!example]- 示例：`DVD.java` / `DVDCollection.java` / `Movies.java`
> 这一组程序共同演示对象数组如何保存对象引用、如何在集合类中逐元素创建对象，以及如何在容量不足时扩展底层数组。
>
> > [!info]- UML 框图
> >
> > ```
> > Movies
> >   |
> >   v
> > DVDCollection
> >   |
> >   | contains
> >   v
> > DVD[]
> >   |
> >   v
> > DVD
> > ```
>
> `DVD.java` 负责定义单个 DVD 对象；`DVDCollection.java` 负责管理 `DVD` 对象数组；`Movies.java` 负责创建集合并触发输出。
>
> **`DVD.java`**
>
> ```java
> import java.text.NumberFormat;
>
> // 表示一张 DVD 影碟
> public class DVD {
>     private String title, director;
>     private int year;
>     private double cost;
>     private boolean bluRay;
>
>     // 初始化一张 DVD 的基本信息
>     public DVD(String title, String director, int year,
>                double cost, boolean bluRay) {
>         this.title = title;
>         this.director = director;
>         this.year = year;
>         this.cost = cost;
>         this.bluRay = bluRay;
>     }
>
>     // 返回当前 DVD 的格式化描述
>     public String toString() {
>         NumberFormat fmt = NumberFormat.getCurrencyInstance();
>         String description = fmt.format(cost) + "\t" + year + "\t";
>         description += title + "\t" + director;
>
>         if (bluRay)
>             description += "\tBlu-Ray";
>
>         return description;
>     }
> }
> ```
>
> **`DVDCollection.java`**
>
> ```java
> import java.text.NumberFormat;
>
> // 表示一个 DVD 收藏集合
> public class DVDCollection {
>     private DVD[] collection;  // 存储 DVD 对象引用的数组
>     private int count;         // 当前已存入的 DVD 数量
>     private double totalCost;  // 当前集合中 DVD 的总价
>
>     // 初始化容量为 100 的空集合
>     public DVDCollection() {
>         collection = new DVD[100];
>         count = 0;
>         totalCost = 0.0;
>     }
>
>     // 添加一张 DVD，必要时先扩容
>     public void addDVD(String title, String director, int year,
>                        double cost, boolean bluRay) {
>         if (count == collection.length)
>             increaseSize();
>
>         collection[count] = new DVD(title, director, year, cost, bluRay);
>         totalCost += cost;
>         count++;
>     }
>
>     // 返回集合的统计信息与所有 DVD 条目
>     public String toString() {
>         NumberFormat fmt = NumberFormat.getCurrencyInstance();
>         String report = "~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\n";
>         report += "My DVD Collection\n\n";
>         report += "Number of DVDs: " + count + "\n";
>         report += "Total cost: " + fmt.format(totalCost) + "\n";
>         report += "Average cost: " + fmt.format(totalCost / count);
>         report += "\n\nDVD List:\n\n";
>
>         for (int dvd = 0; dvd < count; dvd++)
>             report += collection[dvd].toString() + "\n";
>
>         return report;
>     }
>
>     // 将底层数组容量扩大为原来的两倍
>     private void increaseSize() {
>         DVD[] temp = new DVD[collection.length * 2];
>
>         for (int dvd = 0; dvd < collection.length; dvd++)
>             temp[dvd] = collection[dvd];
>
>         collection = temp;
>     }
> }
> ```
>
> **`Movies.java`**
>
> ```java
> // 创建集合并添加 DVD
> public class Movies {
>     public static void main(String[] args) {
>         DVDCollection movies = new DVDCollection();
>
>         movies.addDVD("The Godfather",  "Francis Ford Coppola", 1972, 24.95, true);
>         movies.addDVD("District 9",     "Neill Blomkamp",       2009, 19.95, false);
>         movies.addDVD("Iron Man",       "Jon Favreau",          2008, 15.95, false);
>         movies.addDVD("All About Eve",  "Joseph Mankiewicz",    1950, 17.50, false);
>         movies.addDVD("The Matrix",     "Andy & Lana Wachowski",1999, 19.95, true);
>         System.out.println(movies);
>
>         movies.addDVD("Iron Man 2",  "Jon Favreau",    2010, 22.99, false);
>         movies.addDVD("Casablanca",  "Michael Curtiz", 1942, 19.95, false);
>         System.out.println(movies);
>     }
> }
> ```
> 输出（第一次打印）：
> ```
> My DVD Collection
>
> Number of DVDs: 5
> Total cost: $98.30
> Average cost: $19.66
>
> DVD List:
>
> $24.95  1972  The Godfather   Francis Ford Coppola  Blu-Ray
> $19.95  2009  District 9      Neill Blomkamp
> $15.95  2008  Iron Man        Jon Favreau
> $17.50  1950  All About Eve   Joseph Mankiewicz
> $19.95  1999  The Matrix      Andy & Lana Wachowski  Blu-Ray
> ```
>
> **整体关系**
>
> ```java
> DVDCollection movies = new DVDCollection();
> movies.addDVD(...);
> System.out.println(movies);
> ```
>
> - `Movies` 负责驱动程序流程，`DVDCollection` 负责管理集合状态，`DVD` 负责表示单个元素。
> - 这一结构说明对象数组通常不会直接暴露给主程序，而是封装在更高一层的管理类中。
>
> **`DVD.java` 的作用**
> ```java
> private String title, director;
> private int year;
> private double cost;
> private boolean bluRay;
> ```
> - 所有字段声明为 `private`，符合封装原则，外部只能通过 `toString()` 获取格式化信息。
> - 构造器使用 `this.` 区分同名的参数与实例变量。
>
> **`DVDCollection.java` 的对象数组**
> ```java
> collection = new DVD[100];
> ```
> - 此时数组中 $100$ 个槽位均为 `null`，尚未创建任何 `DVD` 对象。
> - `count` 追踪已实际存入的元素数量，`toString()` 中的循环上界为 `count` 而非 `collection.length`，避免打印 `null` 元素。
>
> **动态扩容**
> ```java
> private void increaseSize() {
>     DVD[] temp = new DVD[collection.length * 2];
>     for (int dvd = 0; dvd < collection.length; dvd++)
>         temp[dvd] = collection[dvd];
>     collection = temp;
> }
> ```
> - 创建容量为原数组 $2$ 倍的新数组 `temp`，将原数组中的引用逐一复制过去。
> - 最后将 `collection` 指向新数组，原数组由垃圾回收器 (*Garbage Collector*) 回收。
> - 复制的是引用而非对象本身，`DVD` 对象在内存中的位置不变。
>
> **逐元素实例化**
> ```java
> collection[count] = new DVD(title, director, year, cost, bluRay);
> count++;
> ```
> - 每次调用 `addDVD` 才创建一个新的 `DVD` 对象并存入数组，体现了对象数组需逐元素实例化的特性。

## 8.4 二维数组

在 Java 中，二维数组 (*Two-Dimensional Array*) 本质上是数组的数组。外层数组的每个元素本身是一个一维数组。其声明与实例化需分别指定两个维度的大小：

```java
int[][] scores = new int[12][50];
// 外层数组有 12 个元素，每个元素是一个大小为 50 的 int 数组
```

访问某个元素时使用两个索引，分别指定行与列：

```java
value = scores[3][6]; // 第 4 行（索引 3）、第 7 列（索引 6）的元素
```

外层的单个索引引用整行数组：

```java
scores[0] // 第 1 行，本身是一个大小为 50 的 int 数组
```

> [!note]
> 由于二维数组是数组的数组，`scores.length` 返回行数（外层数组大小，此处为 $12$），而 `scores[0].length` 返回第 $0$ 行的列数（内层数组大小，此处为 $50$）。遍历二维数组时通常使用嵌套 `for` 循环：
> ```java
> for (int row = 0; row < scores.length; row++)
>     for (int col = 0; col < scores[row].length; col++)
>         System.out.print(scores[row][col] + " ");
> ```

Java 的二维数组还支持锯齿数组 (*Jagged Array*)，即每行的列数可以不同：

```java
int[][] jagged = new int[3][];  // 仅指定行数，列数待定
jagged[0] = new int[5];         // 第 0 行有 5 列
jagged[1] = new int[3];         // 第 1 行有 3 列
jagged[2] = new int[7];         // 第 2 行有 7 列
```

> [!note]
> 锯齿数组不属于课内要求，但理解其存在有助于理解 Java 二维数组“数组的数组”这一结构本质。
>
> 在 C/C++ 中，二维数组在内存中是连续存储的矩形块；Java 的二维数组则是通过引用链接的独立一维数组，两者内存布局不同。
>
> Java 支持任意维度的多维锯齿数组，但在实际开发中三维及以上的数组较少直接使用，通常以对象封装或集合类替代，以提高可读性与可维护性。

> [!note]
> 数组在声明时已确定元素类型，所有维度的最终元素必须属于同一类型。

## 8.5 `ArrayList` 类

`ArrayList` 是 `java.util` 包中提供的动态数组容器类。它与原生数组一样使用数字索引访问元素，但长度可随元素增删自动调整，因此更适合元素个数无法预先确定的场景。

插入元素时，指定位置及之后的元素会整体后移；删除元素时，后续元素会前移以填补空缺。`ArrayList` 只能存储对象引用，不能直接存储 `int`、`double` 等基本类型；若需保存这些数据，必须使用对应的包装类 (*Wrapper Class*)，如 `Integer`、`Double`。

声明与实例化 `ArrayList` 时，通常使用泛型 (*Generic*) 语法指定允许存储的对象类型，其一般形式为：

```java
ArrayList<对象类型> 列表名 = new ArrayList<对象类型>();
```

```java
ArrayList<Integer> intList = new ArrayList<Integer>();
ArrayList<String> strList = new ArrayList<String>();
```

> [!note]
> 泛型 `<Integer>` 确保列表只能存入整型包装对象，编译器会在编译阶段进行类型检查。
>
> 虽然也可声明不带泛型的裸类型 `ArrayList objList;`，但这种写法会失去类型安全性，在现代 Java 中应避免使用。
>
> > [!quote]
> > 关于泛型与包装类的进一步说明，将在 [[ELEC2543 Ch.12 Wrapper Class]] 中展开。

`ArrayList` 提供了丰富的内置方法来操作元素：

- `add(element)`：将元素追加到列表末尾。
- `add(position, element)`：将元素插入到指定索引 `position` 处，原有元素依次后移；`position` 的合法范围为 $0$ 到 `size()`。
- `remove(pos)`：移除并返回指定索引处的元素。
- `remove(Object o)`：移除列表中首次出现的指定对象（基于 `equals` 方法比较），若移除成功则返回 `true`。
- `get(pos)`：返回指定索引处的元素。
- `size()`：返回列表中当前的元素总数（等价于数组的 `length` 属性）。
- `indexOf(Object o)`：返回指定元素首次出现的索引，若列表中不包含该元素则返回 $-1$。

> [!example]- 示例：`IntArrayList.java`
> 演示整型 `ArrayList` 的基本增删查操作及索引变化。
> ```java
> import java.util.ArrayList;
>
> public class IntArrayList {
>     public static void main(String[] args) {
>         // 创建仅保存 Integer 的动态列表
>         ArrayList<Integer> intList = new ArrayList<Integer>();
>
>         // 依次加入 0、5、10、15、20
>         for (int i = 0; i < 5; i++) {
>             intList.add(i * 5);
>         }
>
>         // 输出当前大小与全部元素
>         System.out.println("size of intList: " + intList.size());
>         System.out.println(intList);
>
>         // 删除索引 2 处的元素
>         intList.remove(2);
>         System.out.println("size of intList: " + intList.size());
>         System.out.println(intList);
>
>         // 查找元素 15 当前所在的位置
>         int location = intList.indexOf(15);
>         System.out.println("The index of 15 is " + location);
>
>         // 读取索引 0 处的元素
>         System.out.println("The first element is : " + intList.get(0));
>
>         // 在索引 location 处插入新元素
>         intList.add(location, 100);
>         System.out.println(intList);
>     }
> }
> ```
> 输出：
> ```
> size of intList: 5
> [0, 5, 10, 15, 20]
> size of intList: 4
> [0, 5, 15, 20]
> The index of 15 is 2
> The first element is : 0
> [0, 5, 100, 15, 20]
> ```
> **导入与实例化**
> ```java
> import java.util.ArrayList;
> ArrayList<Integer> intList = new ArrayList<Integer>();
> ```
> - 必须导入 `java.util.ArrayList`。
> - 使用包装类 `Integer` 作为泛型参数。
>
> **追加与打印**
> ```java
> intList.add(i * 5);
> System.out.println(intList);
> ```
> - `add` 方法默认将 $0, 5, 10, 15, 20$ 追加至末尾。
> - 直接打印 `ArrayList` 对象会隐式调用其 `toString()` 方法，输出格式为 `[元素1, 元素2, ...]`。
>
> **按索引删除**
> ```java
> intList.remove(2);
> ```
> - 移除索引 $2$ 处的元素（即 $10$）。
> - 移除后，后续元素 $15$ 和 $20$ 自动向前移动，填补索引 $2$ 的位置，列表大小变为 $4$。
>
> **查找与按索引插入**
> ```java
> int location = intList.indexOf(15);
> intList.add(location, 100);
> ```
> - `indexOf(15)` 返回 $15$ 当前的索引，由于前一步的删除，此时 $15$ 的索引为 $2$。
> - `add(2, 100)` 将 $100$ 插入到索引 $2$ 处，原索引 $2$ 及之后的元素（$15$ 和 $20$）自动向后移动。

> [!example]- 示例：`JavaExample.java`
> 演示字符串 `ArrayList` 的按对象删除与 `for-each` 遍历。
> ```java
> import java.util.ArrayList;
>
> public class JavaExample {
>     public static void main(String[] args) {
>         // 创建仅保存 String 的动态列表
>         ArrayList<String> obj = new ArrayList<String>();
>
>         // 先加入 5 个初始元素
>         obj.add("Ajeet");
>         obj.add("Harry");
>         obj.add("Chaitanya");
>         obj.add("Steve");
>         obj.add("Anuj");
>
>         // 输出原始列表
>         System.out.println("Original ArrayList:");
>         for (String str : obj)
>             System.out.println(str);
>
>         // 在指定索引处插入两个新元素
>         obj.add(0, "Rahul");  // 插入到首位
>         obj.add(1, "Justin"); // 插入到第二位
>
>         System.out.println("ArrayList after add operation:");
>         for (String str : obj)
>             System.out.println(str);
>
>         // 按对象内容删除元素
>         obj.remove("Chaitanya");
>         obj.remove("Harry");
>
>         System.out.println("ArrayList after remove operation:");
>         for (String str : obj)
>             System.out.println(str);
>
>         // 再按索引删除元素
>         obj.remove(1); // 移除当前索引 1 的元素
>
>         System.out.println("Final ArrayList:");
>         for (String str : obj)
>             System.out.println(str);
>     }
> }
> ```
> 输出：
> ```
> Original ArrayList:
> Ajeet
> Harry
> Chaitanya
> Steve
> Anuj
> ArrayList after add operation:
> Rahul
> Justin
> Ajeet
> Harry
> Chaitanya
> Steve
> Anuj
> ArrayList after remove operation:
> Rahul
> Justin
> Ajeet
> Steve
> Anuj
> Final ArrayList:
> Rahul
> Ajeet
> Steve
> Anuj
> ```
> **指定位置插入**
> ```java
> obj.add(0, "Rahul");
> obj.add(1, "Justin");
> ```
> - 第一次插入将 `"Rahul"` 放在首位，原所有元素后移。
> - 第二次插入将 `"Justin"` 放在索引 $1$ 处，原索引 $1$ 及之后的元素再次后移。
>
> **按对象内容删除**
> ```java
> obj.remove("Chaitanya");
> obj.remove("Harry");
> ```
> - 这里传入的是字符串对象而非数字索引，因此 `ArrayList` 会按内容查找第一个相等元素并将其移除。
> - 这种写法适用于已知目标值、但不想先手动查找索引的位置。
>
> **遍历**
> ```java
> for (String str : obj)
> ```
> - `ArrayList` 完全支持 `for-each` 循环语法，底层通过迭代器 (*Iterator*) 实现顺序访问。

---
`Pre: ` [[ELEC2543 Ch.7 Enumerated Type]]
`Post:` [[ELEC2543 Ch.9 Method]]

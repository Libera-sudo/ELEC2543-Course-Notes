#Y2S2 #ELEC2543
# Chapter 7. 枚举类型

`Pre: ` [[ELEC2543 Ch.6 Class Libraries]]
`Post:` [[ELEC2543 Ch.8 Arrays and ArrayList]]

> [!abstract]
> 枚举类型 (_Enumerated Type_) 是一种用于定义有限取值集合的自定义类型，属于类型安全的常量管理机制，广泛应用于表示状态、方向、季节等离散值的场景。
>
> Java 的枚举类型以 `enum` 关键字定义，本质上是一种特殊的类 (_Class_)。与 C/C++ 中枚举仅为整数别名不同，Java 的 `enum` 是完整的类结构，支持构造器、实例变量与方法定义，同时内置序数值管理与名称查询机制，兼顾了类型安全与面向对象的扩展性。
>
> 本章涵盖以下核心模块：枚举类型的基本声明与使用、序数值与内置方法（`ordinal()`、`name()`、`values()`），以及将枚举类型扩展为含构造器与实例变量的完整类结构。
```table-of-contents
maxLevel: 3
```
## 7.1 枚举类型的基本使用

**枚举类型** (*Enumerated Type*) 允许程序员定义一组具名的固定**枚举常量**集合，作为一种新的数据类型使用。当某个变量的取值只能是若干预定义值之一时（如季节、方向、口味），使用枚举类型可以使代码更具可读性，并由编译器保证类型安全。

其声明语法为：

```java
enum TypeName {VALUE1, VALUE2, VALUE3, ...}

enum Season {WINTER, SPRING, SUMMER, FALL}
```

- 枚举类型可以声明在类的内部 ，也可以作为独立文件声明（此时需加 `public` 修饰符）。
- 枚举常量名按惯例全部大写。

枚举变量的声明与赋值语法与普通变量一致，但赋值时须通过类型名访问常量：

```java
Season time;            // 声明一个类型为 Season 的变量 time
time = Season.SPRING;   // 将枚举常量 Season.SPRING 赋值给变量 time
```

> [!note]
> 不能直接写 `time = SPRING;`，必须通过 `Season.SPRING` 访问。这与 C 的枚举不同，Java 枚举常量不在当前作用域内自动可见。
>
> 同时，`Season time` 的类型是 `Season`，而非 `int` 或 `String`。编译器会拒绝如下赋值：
> > [!bug]
> > ```java
> > time = "SPRING";   // 编译错误：类型不匹配
> > time = 1;          // 编译错误：类型不匹配
> > ```

每个枚举常量在内部以整数存储，称为序数值 (*Ordinal Value*)，从 0 开始依次递增。

每个枚举变量提供以下两个常用内置方法：

- `ordinal()`：返回该枚举常量的序数值（`int` 类型），即其在声明列表中的位置，从 0 起计。
- `name()`：返回该枚举常量的名称（`String` 类型），与声明时的标识符完全一致。

将枚举变量直接用于字符串拼接时，效果与调用 `name()` 相同，均输出常量名称。

> [!note]
> 枚举变量的优势：
> - 类型安全：
> 	  编译器拒绝非法赋值。直接使用字符串时，`sunRise = "EASTT"` 这样的拼写错误在编译期不会被发现，运行时才会出错；使用枚举时，`Direction.EASTT` 在编译期即报错。
>
> - 可读性：
> 	  `Direction.EAST` 代码意图一目了然。
>
> - 约束取值范围：
> 	  `Direction` 类型的变量只能持有 `NORTH`、`EAST`、`SOUTH`、`WEST` 四个值之一，不可能出现 `5` 或 `"northwest"` 这样的非法值。
>
> - IDE 支持：
> 	  使用枚举时，IDE 可以自动补全所有合法常量，减少手动输入错误。

> [!example]- 示例：`Direction.java` + `Sun.java`
> 演示枚举类型在独立文件中声明，并在另一个类中使用枚举变量与字符串拼接输出。
>
> ```java
> // Direction.java
> enum Direction {NORTH, EAST, SOUTH, WEST}   // Sun.java 和 Direction.java 位于同一包内，无需加 public
> ```
> ```java
> // Sun.java
> public class Sun {
>     public static void main(String[] args) {
>         Direction sunRise = Direction.EAST;   // 序数值为 1
>         Direction sunSet = Direction.WEST;    // 序数值为 3
>
>         System.out.println("The sun rises in the " + sunRise +
>                            " and sets in the " + sunSet + ".");
>     }
> }
> ```
>
> 输出：
> ```
> The sun rises in the EAST and sets in the WEST.
> ```
>
> **枚举声明**
> ```java
> enum Direction {NORTH, EAST, SOUTH, WEST}
> ```
> - 声明为独立文件，不加 `public`，因此与 `Sun.java` 位于同一包内即可访问。
> - 四个常量的序数值依次为 0、1、2、3。
>
> **枚举变量赋值与字符串拼接**
> ```java
> Direction sunRise = Direction.EAST;
> System.out.println("... " + sunRise + " ...");
> ```
> - 枚举变量在字符串拼接中自动调用 `toString()`，输出结果与 `name()` 相同，即常量名称字符串。

> [!example]- 示例：`IceCream.java`
> 演示枚举类型声明在类内部，以及 `ordinal()`、`name()` 方法与枚举赋值的使用。
>
> ```java
> public class IceCream {
>     // 在类内部声明枚举类型
>     enum Flavor {COFFEE, VANILLA, STRAWBERRY, CHOCOLATE, MINT}
>
>     public static void main(String[] args) {
>         Flavor cone1, cone2, cone3;
>
>         cone1 = Flavor.MINT;    // 序数值为 4
>         cone2 = Flavor.COFFEE;  // 序数值为 0
>
>         System.out.println("cone1 value: " + cone1);
>         System.out.println("cone1 ordinal: " + cone1.ordinal());
>         System.out.println("cone1 name: " + cone1.name());
>
>         System.out.println();
>         System.out.println("cone2 value: " + cone2);
>         System.out.println("cone2 ordinal: " + cone2.ordinal());
>         System.out.println("cone2 name: " + cone2.name());
>
>         cone3 = cone1;  // 枚举变量赋值，cone3 与 cone1 指向同一常量
>
>         System.out.println();
>         System.out.println("cone3 value: " + cone3);
>         System.out.println("cone3 ordinal: " + cone3.ordinal());
>         System.out.println("cone3 name: " + cone3.name());
>     }
> }
> ```
>
> 输出：
> ```
> cone1 value: MINT
> cone1 ordinal: 4
> cone1 name: MINT
>
> cone2 value: COFFEE
> cone2 ordinal: 0
> cone2 name: COFFEE
>
> cone3 value: MINT
> cone3 ordinal: 4
> cone3 name: MINT
> ```
>
> **枚举类型声明**
> ```java
> enum Flavor {COFFEE, VANILLA, STRAWBERRY, CHOCOLATE, MINT}
> ```
> - 声明在 `IceCream` 类内部，序数值从 0 起：`COFFEE=0`，`VANILLA=1`，`STRAWBERRY=2`，`CHOCOLATE=3`，`MINT=4`。
>
> ** `ordinal()` 与 `name()` 方法**
> ```java
> cone1 = Flavor.MINT;
> System.out.println("cone1 value: " + cone1);       // 输出 MINT
> System.out.println("cone1 ordinal: " + cone1.ordinal()); // 输出 4
> System.out.println("cone1 name: " + cone1.name());  // 输出 MINT
> ```
> - `cone1` 直接参与字符串拼接时输出常量名，与 `name()` 结果一致。
> - `ordinal()` 返回 `MINT` 在声明列表中的位置索引 4。
>
> **枚举变量赋值**
> ```java
> cone3 = cone1;
> ```
> - 枚举变量赋值使 `cone3` 与 `cone1` 指向同一枚举常量，因此两者的所有方法返回值完全相同。

## 7.2 枚举类型作为类

枚举类型在 Java 中是一种特殊的类 (*Class*)。除了定义一组固定常量外，枚举还可以包含实例变量、构造器和方法，从而为每个枚举常量附加额外的数据与行为。

枚举作为类时，其结构如下：

```java
public enum TypeName {
    VALUE1 (arg1),      // 枚举常量名称(传参)
    VALUE2 (arg2);      // 常量列表以分号结束

    private Type instanceVar;   // 实例变量

    TypeName (Type arg) {     // 隐式构造器，使用枚举常量的参数
        instanceVar = arg;
    }

    public Type getInstanceVar() {  // 方法
        return instanceVar;
    }
}
```

- 枚举常量列表必须位于类体的**最前面**，且以分号 `;` 结束（当类体中存在其他成员时）。
- 枚举的构造器隐式为 `private`（不加访问修饰符），不能在枚举外部手动实例化枚举对象。
- 每个枚举常量在声明时传入的参数会调用构造器，为该常量的实例变量赋值。

> [!note]
> 枚举常量本质上是该枚举类型的唯一实例，由 JVM 在类加载时创建，且每个常量只存在一个实例。因此枚举天然适合用于单例模式。

`values()` 方法是每个枚举类型自动提供的静态方法。

- 返回包含所有枚举常量的数组。
- 数组元素按声明顺序排列，类型为该枚举类型。

其常见用法为配合增强型 `for` 循环 (_Enhanced for Loop_) 遍历所有常量：

```java
for (Season time : Season.values())    // .value() 返回包含所有常量的数组；变量 time 依次指向数组中的下一个枚举常量
    // : 表示“来自于”
    System.out.println(time + "\t" + time.getSpan());
```

> [!example]- 示例：`Season.java` + `SeasonTester.java`
> 演示枚举类型作为类，定义实例变量、构造器与方法，并使用 `values()` 遍历所有枚举常量。
>
> ```java
> // Season.java
> public enum Season {
>     WINTER ("December through February"),   // 调用构造器，传入月份描述
>     SPRING ("March through May"),
>     SUMMER ("June through August"),
>     FALL   ("September through November");  // 常量列表以分号结束
>
>     private String span;    // 实例变量，存储月份描述
>
>     // 构造器，为每个枚举常量的 span 赋值
>     Season (String months) {
>         span = months;
>     }
>
>     // 访问器方法
>     public String getSpan() {
>         return span;
>     }
> }
> ```
>
> ```java
> // SeasonTester.java
> public class SeasonTester {
>     public static void main(String[] args) {
>         // 使用 values() 获取所有枚举常量并遍历
>         for (Season time : Season.values())
>             System.out.println(time + "\t" + time.getSpan());
>     }
> }
> ```
>
> 输出：
> ```
> WINTER    December through February
> SPRING    March through May
> SUMMER    June through August
> FALL      September through November
> ```
>
> **枚举常量声明与构造器调用**
> ```java
> WINTER ("December through February"),
> ```
> - 每个常量在声明时传入字符串参数，触发 `Season(String months)` 构造器，将参数赋值给实例变量 `span`。
> - 四个常量的序数值依次为 0（`WINTER`）至 3（`FALL`）。
>
> **实例变量与访问器**
> ```java
> private String span;
> public String getSpan() { return span; }
> ```
> - `span` 是每个枚举常量独立持有的实例变量，不同常量的 `span` 值各不相同。
> - `getSpan()` 是普通的实例方法，通过枚举常量直接调用。
>
> ** `values()` 方法与增强型 `for` 循环**
> ```java
> for (Season time : Season.values())
>     System.out.println(time + "\t" + time.getSpan());
> ```
> - `Season.values()` 返回 `Season[]` 数组，元素按声明顺序排列：`{WINTER, SPRING, SUMMER, FALL}`。
> - 每次迭代中，`time` 依次指向各枚举常量，`time.getSpan()` 返回对应的月份描述字符串。

---
`Pre: ` [[ELEC2543 Ch.6 Class Libraries]]
`Post:` [[ELEC2543 Ch.8 Arrays and ArrayList]]


#Y2S2 #ELEC2543
# Chapter 7. 枚举类型

`Pre: ` [[ELEC2543 Ch.6 Class Libraries]]
`Post:` [[ELEC2543 Ch.8 Arrays and ArrayList]]

> [!abstract]
> 枚举类型 (*Enumerated Type*) 是用于表示有限离散取值集合的自定义类型，适合描述方向、季节、状态等只能取若干固定值之一的场景。
>
> Java 使用 `enum` 关键字定义枚举，并将其实现为一种特殊的类 (*Class*)。与仅把枚举视为整数别名的语言不同，Java 枚举同时提供类型安全、常量命名、内置方法以及实例变量、构造器与方法扩展能力，使“有限取值集合”既可作为常量系统，也可作为完整对象结构。
>
> 本章依次介绍枚举类型的基本声明与赋值方式、`ordinal()` / `name()` / `values()` 等内置方法，以及将枚举扩展为带实例变量、构造器和访问器的类结构。
```table-of-contents
maxLevel: 3
```

## 7.1 枚举类型的基本使用

枚举类型 (*Enumerated Type*) 允许程序员定义一组具名且固定的枚举常量 (*Enumerated Constants*)，并把它们作为一种新的类型使用。当某个变量的取值范围天然有限时，枚举能同时提供更清晰的语义和更严格的类型约束。

其声明语法为：

```java
enum <类型名> {<常量 1>, <常量 2>, <常量 3>, ...}
```

```java
enum Season {WINTER, SPRING, SUMMER, FALL}
```

- 枚举类型可以声明在类内部，也可以作为独立文件声明。
- 若枚举作为独立文件且需要对外公开，应使用 `public` 修饰。
- 枚举常量名按惯例全部大写。

枚举变量的声明与赋值方式和普通变量一致，但赋值时必须通过 `<类型名>.<常量名>` 的形式访问目标常量。

```java
Season time;
time = Season.SPRING;
```

> [!note]
> 不能直接写 `time = SPRING;`，必须写成 `Season.SPRING`。Java 不会像某些语言那样把枚举常量自动暴露到当前作用域。
>
> 同时，`time` 的类型是 `Season`，而不是 `int` 或 `String`。
>
> > [!bug]
> > ```java
> > time = "SPRING";   // 编译错误：类型不匹配
> > time = 1;          // 编译错误：类型不匹配
> > ```

每个枚举常量在内部都对应一个从 0 开始的序数值 (*Ordinal Value*)，按其在声明列表中的顺序依次递增。

枚举变量最常见的两个内置方法为：

- `ordinal()`：返回该常量的序数值，即其在声明列表中的位置索引。
- `name()`：返回该常量的名称字符串，与声明时的标识符完全一致。

将枚举变量直接用于字符串拼接时，输出效果与调用 `name()` 一致，都会得到常量名称。

> [!note]
> 枚举变量的优势主要体现在类型安全、可读性与取值范围约束上。
>
> - 编译器会拒绝非法赋值，拼写错误也能在编译期被发现。
> - `Direction.EAST` 这类写法比整数或裸字符串更能直接表达程序意图。
> - `Direction` 类型的变量只能持有声明中给出的合法常量，不能落入任意整数或任意字符串。

> [!example]- 示例：`Direction.java` / `Sun.java`
> 观察枚举在独立文件中声明后，如何被另一个类引用并参与字符串输出。
>
> > [!info]- UML 框图
> >
> > ```
> > Direction.java
> >   └─ 定义枚举类型 Direction
> > 
> > Sun.java
> >   └─ 在 main() 中使用 Direction.EAST / Direction.WEST
> > ```
>
> ```java
> // Direction.java
> enum Direction {NORTH, EAST, SOUTH, WEST}
> ```
>
> ```java
> // Sun.java
> public class Sun {
>     public static void main(String[] args) {
>         Direction sunRise = Direction.EAST;
>         Direction sunSet = Direction.WEST;
>
>         System.out.println("The sun rises in the " + sunRise
>                            + " and sets in the " + sunSet + ".");
>     }
> }
> ```
>
> 输出：
>
> ```
> The sun rises in the EAST and sets in the WEST.
> ```
>
> **整体关系**
>
> - `Direction.java` 负责定义枚举类型本身，`Sun.java` 负责创建枚举变量并观察其输出行为。
> - 该组程序说明：枚举既可以独立成文件，也可以像普通类型一样被其他类直接使用。
>
> **`Direction.java` 的作用**
>
> ```java
> enum Direction {NORTH, EAST, SOUTH, WEST}
> ```
> - 四个常量按声明顺序形成一个新的离散类型。
> - 若不加 `public`，则该枚举可被同一包中的其他类访问。
>
> **`Sun.java` 的作用**
>
> ```java
> Direction sunRise = Direction.EAST;
> Direction sunSet = Direction.WEST;
> ```
> - 两个变量的类型都是 `Direction`，因此只能接收 `Direction` 中已声明的常量。
> - `Direction.EAST` 与 `Direction.WEST` 分别对应序数值 1 和 3。
>
> **字符串拼接输出**
>
> ```java
> System.out.println("The sun rises in the " + sunRise
>                    + " and sets in the " + sunSet + ".");
> ```
> - 枚举变量参与字符串拼接时会输出常量名称。
> - 因此运行结果中的 `EAST` 与 `WEST` 本质上对应的是各自的 `name()` 返回值。

> [!example]- 示例：`IceCream.java`
> 观察枚举声明在类内部时，`ordinal()`、`name()` 与枚举变量赋值如何配合使用。
>
> ```java
> public class IceCream
> {
>     // 在类内部声明枚举类型
>     enum Flavor {COFFEE, VANILLA, STRAWBERRY, CHOCOLATE, MINT}
>
>     public static void main(String[] args) {
>         Flavor cone1, cone2, cone3;
>
>         cone1 = Flavor.MINT;
>         cone2 = Flavor.COFFEE;
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
>         cone3 = cone1;
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
>
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
> **类内枚举声明**
>
> ```java
> enum Flavor {COFFEE, VANILLA, STRAWBERRY, CHOCOLATE, MINT}
> ```
> - `Flavor` 声明在 `IceCream` 类内部，因此它的作用域局限在该类中。
> - 常量的序数值按声明顺序从 0 递增，`MINT` 的序数值为 4。
>
> **`ordinal()` 与 `name()`**
>
> ```java
> System.out.println("cone1 ordinal: " + cone1.ordinal());
> System.out.println("cone1 name: " + cone1.name());
> ```
> - `ordinal()` 返回常量在声明列表中的位置索引。
> - `name()` 返回常量标识符本身，因此 `cone1` 的名称为 `MINT`。
>
> **枚举变量赋值**
>
> ```java
> cone3 = cone1;
> ```
> - 赋值后，`cone3` 与 `cone1` 指向同一个枚举常量。
> - 因此二者在输出值、序数值和名称上完全一致。

## 7.2 枚举类型作为类

在 Java 中，枚举不是“带名字的整数集合”，而是一种特殊的类 (*Class*)。因此它除了定义固定常量外，还可以继续声明实例变量、构造器和方法，为每个枚举常量附加额外的数据与行为。

其一般结构可写为：

```java
public enum TypeName {
    VALUE1(arg1),
    VALUE2(arg2);

    private Type instanceVar;

    TypeName(Type arg) {
        instanceVar = arg;
    }

    public Type getInstanceVar() {
        return instanceVar;
    }
}
```

- 枚举常量列表必须位于类体最前面。
- 当枚举中还定义了实例变量、构造器或方法时，常量列表末尾必须加分号 `;`。
- 枚举构造器不能由外部手动调用，其实例创建由枚举常量声明过程隐式完成。

> [!note]
> 枚举构造器虽然通常不写访问修饰符，但其可见性等效于 `private`。这意味着程序不能在枚举外部写出 `new Season(...)` 之类的代码。

`values()` 是每个枚举类型自动提供的静态方法，用于返回按声明顺序排列的“全部枚举常量数组”。

其常见用法为：`for (<枚举类型> <变量> : <枚举类型>.values())`

```java
for (Season time : Season.values())
    System.out.println(time + "\t" + time.getSpan());
```

> [!note]
> 冒号 `:` 在增强型 `for` 循环中可理解为“来自于”。上式表示：变量 `time` 依次取自 `Season.values()` 返回数组中的每个枚举常量。

> [!example]- 示例：`Season.java` / `SeasonTester.java`
> 观察枚举如何扩展为带实例变量、构造器和访问器的方法结构，并通过 `values()` 遍历全部常量。
>
> > [!info]- UML 框图
> >
> > ```
> > Season.java
> >   └─ 定义枚举 Season
> >      ├─ 实例变量 span
> >      ├─ 构造器 Season(String months)
> >      └─ 方法 getSpan()
> > 
> > SeasonTester.java
> >   └─ 遍历 Season.values() 并输出常量及其月份描述
> > ```
>
> ```java
> // Season.java
> public enum Season {
>     WINTER("December through February"),
>     SPRING("March through May"),
>     SUMMER("June through August"),
>     FALL("September through November");
>
>     private String span;
>
>     // 构造器：为每个枚举常量绑定月份描述
>     Season(String months) {
>         span = months;
>     }
>
>     // 访问器：返回当前常量对应的月份描述
>     public String getSpan() {
>         return span;
>     }
> }
> ```
>
> ```java
> // SeasonTester.java
> import java.util.*;
>
> public class SeasonTester {
>     public static void main(String[] args) {
>         // 遍历所有枚举常量并输出其描述
>         for (Season time : Season.values())
>             System.out.println(time + "\t" + time.getSpan());
>     }
> }
> ```
>
> 输出：
>
> ```
> WINTER	December through February
> SPRING	March through May
> SUMMER	June through August
> FALL	September through November
> ```
>
> **整体关系**
>
> - `Season.java` 负责定义枚举常量与附属数据，`SeasonTester.java` 负责遍历并展示这些数据。
> - 该组程序说明：枚举常量不仅有名称和序数值，还能像对象一样携带实例变量和方法。
>
> **常量声明与构造器参数**
>
> ```java
> WINTER("December through February"),
> SPRING("March through May"),
> SUMMER("June through August"),
> FALL("September through November");
> ```
> - 每个常量在声明时都传入一个字符串参数，该参数随后交给构造器处理。
> - 这些参数使不同常量拥有不同的 `span` 值，而不只是不同的名称。
>
> **实例变量与访问器**
>
> ```java
> private String span;
>
> public String getSpan() {
>     return span;
> }
> ```
> - `span` 是每个枚举常量各自持有的实例变量。
> - `getSpan()` 通过当前常量对象返回对应的月份描述。
>
> **构造器**
>
> ```java
> Season(String months) {
>     span = months;
> }
> ```
> - 构造器在每个枚举常量初始化时自动执行。
> - `months` 的值来自该常量声明时括号中的参数。
>
> **`values()` 与增强型 `for`**
>
> ```java
> for (Season time : Season.values())
>     System.out.println(time + "\t" + time.getSpan());
> ```
> - `Season.values()` 返回 `Season[]` 数组，元素顺序与声明顺序一致。
> - 每次循环中，`time` 依次指向一个枚举常量，再调用 `getSpan()` 输出其月份描述。

---
`Pre: ` [[ELEC2543 Ch.6 Class Libraries]]
`Post:` [[ELEC2543 Ch.8 Arrays and ArrayList]]

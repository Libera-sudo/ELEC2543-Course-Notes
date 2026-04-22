#Y2S2 #ELEC2543
# Chapter 9. 方法

`Pre: ` [[ELEC2543 Ch.8 Arrays and ArrayList]]
`Post:` [[ELEC2543 Ch.10 Static Modifier]]

> [!abstract]
> 方法 (*Method*) 是结构化程序设计与面向对象编程的基本构建单元，属于模块化设计领域的核心机制。通过将逻辑封装为具名的可复用代码块，程序得以降低冗余、提升可维护性。
>
> Java 中的方法必须定义在类 (*Class*) 内部，这与 C/C++ 中的独立函数不同。Java 在参数传递上统一采用值传递 (*Pass by Value*)：对基本类型 (*Primitive Type*) 复制值本身，对对象 (*Object*) 复制引用副本。这一设计直接影响方法对外部状态的修改能力。
>
> 本章涵盖方法头与方法体的结构、`return` 语句、方法重载 (*Method Overloading*)、参数传递机制、数组作为参数，以及可变长参数列表 (*Variable Length Parameter Lists*)。

```table-of-contents
maxLevel: 3
```

## 9.1 方法结构

方法是类中封装特定功能的具名代码块，由方法头与方法体两部分构成。

#### 方法头

方法头 (*Method Header*) 声明方法的返回类型、名称与形式参数列表，其语法为：

```java
<返回类型> <方法名>(<参数类型> <参数名>, ...)

char calc(int num1, int num2, String message)
```

- `char` 是返回类型 (*Return Type*)，表示该方法执行完毕后向调用处返回一个 `char` 值。
- `calc` 是方法名。
- `int num1, int num2, String message` 是形式参数列表 (*Formal Parameter List*)，每个参数须单独声明类型，不可合并写为 `int num1, num2`。
- 列表中每个参数的名称称为形式参数 (*Formal Parameter*)。

#### 方法体

方法体 (*Method Body*) 是紧跟方法头的花括号块，包含具体执行逻辑：

```java
char calc(int num1, int num2, String message) {
    int sum = num1 + num2;              // 局部变量
    char result = message.charAt(sum);  // 局部变量
    return result;                      // 返回语句
}
```

- `sum` 与 `result` 是局部变量 (*Local Variable*)，仅在该方法的作用域 (*Scope*) 内存在。
- 局部变量在每次方法被调用时创建，方法执行结束后销毁，不保留上次调用的值。
- `return result;` 将 `result` 的值返回给调用处，执行后方法立即终止。

#### `return` 语句

`return` 语句的语法为：

```java
return <表达式>;
```

- `expression` 的类型必须与方法头声明的返回类型一致，或可自动转换为该类型。
- 若方法返回类型为 `void`，则不返回任何值；此时可省略 `return` 语句，或单独写 `return;` 以提前退出方法。

> [!note]
> `void` 方法中使用 `return expression;` 会导致编译错误。非 `void` 方法中若存在某条执行路径未到达任何 `return` 语句，同样会导致编译错误：编译器要求所有可能的执行路径都必须有明确的返回值。

## 9.2 方法重载

方法重载是指在同一个类中为同一个方法名定义多个版本，每个版本处理不同类型或数量的参数。编译器在编译期根据调用时传入的实际参数自动分派对应版本，无需程序员手动区分。

编译器挑选的依据是方法签名 (*Method Signature*)。方法签名由以下三项共同决定：

1. 参数的数量
2. 参数的类型
3. 参数的顺序

重载的每个版本必须具有唯一的签名，否则编译器无法区分。其声明语法与普通方法相同，仅参数列表不同：

```java
float tryMe(int x) {
    return x + .375f;    // f 后缀声明该常量为 float 类型
}

float tryMe(int x, float y) {
    return x * y;
}
```

调用时，编译器根据实际参数匹配对应版本：

```java
result = tryMe(25, 4.32f);  // 匹配第二个版本：tryMe(int, float)
```

> [!note]
> 返回类型不属于签名的一部分。两个方法若仅返回类型不同而参数列表完全相同，编译器将报错，因为调用处无法仅凭返回类型区分应调用哪个版本：
>
> ```java
> int compute(int x)    { ... }
> double compute(int x) { ... }  // 编译错误：签名重复
> ```

`println` 是标准库中方法重载的典型例子，`PrintStream` 类为其定义了多个版本：

```java
println(String s)
println(int i)
println(double d)
// ... 以及更多版本
```

以下两行调用分别匹配不同版本：

```java
System.out.println("The total is:");  // 匹配 println(String)
System.out.println(total);            // 根据 total 的类型匹配对应版本
```

构造器 (*Constructor*) 同样可以重载。重载的构造器为对象的初始化提供多种方式，例如允许调用者选择性地提供初始值，或以不同类型的数据初始化同一对象。

> [!note]
> 方法重载在编译期完成解析，称为静态分派 (*Static Dispatch*)；与之相对的运行期动态分派 (*Dynamic Dispatch*) 则依赖运行时对象的实际类型。重载版本的选择完全由编译器在编译时根据参数类型决定，不涉及动态分派。

## 9.3 参数传递机制

方法被调用时，调用处提供的值称为实际参数 (*Actual Parameter*)，方法头中声明的变量称为形式参数。Java 的参数传递规则统一为值传递：实际参数的值被复制到对应的形式参数中，这一过程等价于一次赋值操作。对基本类型（`int`、`double`、`char` 等）而言，复制的是值本身，形式参数获得一份独立副本；对对象而言，复制的是引用 (*Reference*)，形式参数与实际参数成为同一对象的别名 (*Alias*)。

> [!comparison]
> Java 对基本类型与对象类型都采用值传递，区别在于被复制的"值"是什么。
>
> **基本类型**
>
> 复制的是值本身，形式参数与实际参数在内存中是相互独立的两份数据。对形式参数赋值只改写副本，不影响调用处。
>
> **对象类型**
>
> 复制的是引用，形式参数与实际参数指向同一对象。
>
> - 通过形式参数修改对象内部状态，调用处可见。
> - 将形式参数重新指向新对象，只改写引用副本自身，不影响调用处的原始引用。

> [!note]
> C++ 支持真正的按引用传递（传递变量地址），Java 没有该机制。Java 中对对象的操作之所以"看起来像按引用传递"，是因为复制的是引用副本，两端共享同一对象；但在方法内将形式参数重新指向新对象（如 `f3 = new Num(777)`）时，调用处的原始引用不受影响，这一点与 C++ 的按引用传递有本质不同。

> [!example]- 示例：`Num.java` / `ParameterModifier.java` / `ParameterTester.java`
>
> `Num.java`、`ParameterModifier.java`、`ParameterTester.java` 共同演示基本类型与对象类型在参数传递中的行为差异，以及修改对象内部状态与重新指向新对象的不同效果。`Num.java` 定义封装单个整数的辅助类；`ParameterModifier.java` 提供对三个参数执行不同修改的方法；`ParameterTester.java` 为主程序，调用 `changeValues` 并观察调用前后的状态。
>
> > [!info]- UML 框图
> >
> > ```
> > ┌────────────────────┐
> > │ ParameterTester    │
> > │  + main            │
> > └─────────┬──────────┘
> >           │ 调用
> >           v
> > ┌──────────────────────────────┐
> > │ ParameterModifier            │
> > │  + changeValues(int,Num,Num) │
> > └─────────┬────────────────────┘
> >           │ 操作
> >           v
> > ┌──────────────────────┐
> > │ Num                  │
> > │  - value : int       │
> > │  + Num(int)          │
> > │  + setValue(int)     │
> > │  + toString()        │
> > └──────────────────────┘
> > ```
>
> ```java
> // Num.java —— 封装单个整数的辅助类
> public class Num {
>     private int value;
>
>     public Num(int update) {            // 构造器：初始化 value
>         value = update;
>     }
>
>     public void setValue(int update) {  // 修改内部状态
>         value = update;
>     }
>
>     public String toString() {          // 返回字符串表示
>         return value + "";
>     }
> }
> ```
>
> ```java
> // ParameterModifier.java —— 对三个参数执行不同类型的修改
> public class ParameterModifier {
>     public void changeValues(int f1, Num f2, Num f3) {
>         System.out.println("Before changing the values:");
>         System.out.println("f1\tf2\tf3");
>         System.out.println(f1 + "\t" + f2 + "\t" + f3 + "\n");
>
>         f1 = 999;              // 修改基本类型副本
>         f2.setValue(888);      // 修改对象内部状态
>         f3 = new Num(777);     // 将形式参数重新指向新对象
>
>         System.out.println("After changing the values:");
>         System.out.println("f1\tf2\tf3");
>         System.out.println(f1 + "\t" + f2 + "\t" + f3 + "\n");
>     }
> }
> ```
>
> ```java
> // ParameterTester.java —— 主程序，调用 changeValues 并观察前后变化
> public class ParameterTester {
>     public static void main(String[] args) {
>         ParameterModifier modifier = new ParameterModifier();
>
>         int a1 = 111;
>         Num a2 = new Num(222);
>         Num a3 = new Num(444);
>
>         System.out.println("Before calling changeValues:");
>         System.out.println("a1\ta2\ta3");
>         System.out.println(a1 + "\t" + a2 + "\t" + a3 + "\n");
>
>         modifier.changeValues(a1, a2, a3);
>
>         System.out.println("After calling changeValues:");
>         System.out.println("a1\ta2\ta3");
>         System.out.println(a1 + "\t" + a2 + "\t" + a3 + "\n");
>     }
> }
> ```
>
> 输出：
>
> ```
> Before calling changeValues:
> a1    a2    a3
> 111   222   444
>
> Before changing the values:
> f1    f2    f3
> 111   222   444
>
> After changing the values:
> f1    f2    f3
> 999   888   777
>
> After calling changeValues:
> a1    a2    a3
> 111   888   444
> ```
>
> **基本类型参数 `f1 = 999`**
>
> ```java
> f1 = 999;  // 仅修改形式参数副本
> ```
>
> - `f1` 是 `a1` 值的独立副本，对 `f1` 的赋值不影响 `a1`。
> - 方法返回后 `a1` 仍为 `111`。
>
> **修改对象内部状态 `f2.setValue(888)`**
>
> ```java
> f2.setValue(888);  // 通过共享引用修改同一对象
> ```
>
> - `f2` 与 `a2` 指向同一个 `Num` 对象，`setValue` 直接修改该对象的 `value` 字段。
> - 方法返回后 `a2` 所指对象的值变为 `888`，调用处可见。
>
> **重新指向新对象 `f3 = new Num(777)`**
>
> ```java
> f3 = new Num(777);  // 仅改变形式参数的指向
> ```
>
> - `f3` 是引用的副本，将其指向新对象只改变副本的指向，不影响 `a3`。
> - 方法返回后 `a3` 仍指向原来的 `Num(444)` 对象，值为 `444`。
> - 新创建的 `Num(777)` 对象在方法结束后无任何引用指向它，成为垃圾 (*Garbage*)，由垃圾回收器 (*Garbage Collector*) 回收。

> [!question] 习题：修改后的 `changeValues` 输出分析与垃圾对象判断
> `Num.java` 与 `ParameterTester.java` 保持与示例原版相同，仅将 `ParameterModifier` 中的 `changeValues` 方法替换为以下版本。程序的完整输出是什么？方法执行过程中是否产生垃圾对象？
>
> ```java
> // ParameterModifier.java —— changeValues 替换为新版本
> public class ParameterModifier {
>     public void changeValues(int a1, Num a2, Num a3) {
>         System.out.println("Before changing the values:");
>         System.out.println("a1\ta2\ta3");
>         System.out.println(a1 + "\t" + a2 + "\t" + a3 + "\n");
>
>         a2.setValue(a1);   // 修改 a2 所指对象的内部状态
>         a1 = 999;          // 修改基本类型副本
>         Num a4 = a2;       // a4 与 a2 指向同一对象
>         a2 = a3;           // a2 改为指向 a3 所指对象
>         a3 = a4;           // a3 改为指向原 a2 所指对象
>
>         System.out.println("After changing the values:");
>         System.out.println("a1\ta2\ta3");
>         System.out.println(a1 + "\t" + a2 + "\t" + a3 + "\n");
>     }
> }
> ```
>
> > [!check]-
> >
> > 输出：
> >
> > ```
> > Before calling changeValues:
> > a1    a2    a3
> > 111   222   444
> >
> > Before changing the values:
> > a1    a2    a3
> > 111   222   444
> >
> > After changing the values:
> > a1    a2    a3
> > 999   444   111
> >
> > After calling changeValues:
> > a1    a2    a3
> > 111   111   444
> > ```
> >
> > **调用前主程序状态**
> >
> > | 变量 | 类型 | 值 / 指向 |
> > |------|------|---------|
> > | `a1`（主程序） | `int` | `111` |
> > | `a2`（主程序） | `Num` 引用 | 指向 `Num(222)` |
> > | `a3`（主程序） | `Num` 引用 | 指向 `Num(444)` |
> >
> > **进入方法时形式参数初始状态**（复制自实际参数）
> >
> > | 形式参数 | 值 / 指向 |
> > |---------|---------|
> > | `a1`（方法内） | `111`（副本） |
> > | `a2`（方法内） | 指向 `Num(222)`（与主程序 `a2` 同一对象） |
> > | `a3`（方法内） | 指向 `Num(444)`（与主程序 `a3` 同一对象） |
> >
> > **执行 `a2.setValue(a1)`**
> >
> > ```java
> > a2.setValue(a1);  // a1 此时为 111，将 Num(222) 的 value 改为 111
> > ```
> >
> > - `a2` 所指对象（即主程序 `a2` 所指对象）内部 `value` 由 `222` 变为 `111`。
> > - 主程序的 `a2` 可见此变化。
> >
> > **执行 `a1 = 999`**
> >
> > ```java
> > a1 = 999;  // 仅修改方法内形式参数副本
> > ```
> >
> > - 主程序的 `a1` 仍为 `111`，不受影响。
> >
> > **执行 `Num a4 = a2`**
> >
> > ```java
> > Num a4 = a2;  // a4 与方法内 a2 指向同一对象（value 已为 111）
> > ```
> >
> > - 此时 `a2`、`a4` 均指向 `value = 111` 的对象。
> >
> > **执行 `a2 = a3`**
> >
> > ```java
> > a2 = a3;  // 方法内 a2 改为指向 Num(444)
> > ```
> >
> > - 方法内 `a2` 现在指向 `value = 444` 的对象。
> > - 主程序的 `a2` 引用不受影响，仍指向 `value = 111` 的对象。
> >
> > **执行 `a3 = a4`**
> >
> > ```java
> > a3 = a4;  // 方法内 a3 改为指向 value = 111 的对象
> > ```
> >
> > - 方法内 `a3` 现在指向 `value = 111` 的对象。
> > - 主程序的 `a3` 引用不受影响，仍指向 `value = 444` 的对象。
> >
> > **方法内 "After changing the values" 输出**
> >
> > | 变量 | 值 |
> > |------|----|
> > | `a1`（方法内） | `999` |
> > | `a2`（方法内） | `444` |
> > | `a3`（方法内） | `111` |
> >
> > **方法返回后主程序 "After calling changeValues" 输出**
> >
> > | 变量 | 值 | 原因 |
> > |------|----|------|
> > | `a1`（主程序） | `111` | 基本类型值传递，副本修改不影响原变量 |
> > | `a2`（主程序） | `111` | `setValue(a1)` 修改了对象内部状态，主程序可见 |
> > | `a3`（主程序） | `444` | 方法内仅修改了形式参数的引用指向，主程序 `a3` 不受影响 |
> >
> > **垃圾对象分析**
> >
> > 本题中不产生垃圾对象。方法内所有引用操作均在已有对象之间重新指向，没有使用 `new` 创建任何新对象，因此不存在失去所有引用的对象。对比原版 `changeValues` 中的 `f3 = new Num(777)`，该操作创建了一个新对象并在方法结束后失去引用，才会产生垃圾。

## 9.4 数组作为参数

数组在 Java 中是对象，因此将数组传入方法时，传递的是数组引用的副本，形式参数与实际参数指向同一块数组内存。在方法内修改数组元素，会直接影响原始数组。

传递方式的声明语法为：

```java
void methodName(int[] arr)  // 形式参数类型为 int[]
```

若只传递数组中的某一个元素，则形式参数类型与元素类型相同（如 `int`），此时按基本类型处理，方法内的修改不影响原数组中的该元素。

> [!comparison]
> 数组整体传递与单个元素传递的行为截然不同。
>
> **传递整个数组 `arr`**
>
> 形式参数与原数组共享同一引用，方法内 `arr[i] = x` 修改原数组。
>
> **传递单个元素 `arr[i]`**
>
> 元素按基本类型复制为独立副本，方法内对形式参数的修改不影响 `arr[i]`。

> [!example]- 示例：`BasicArray.java`
>
> 此示例引用于 [[ELEC2543 Ch.8 Arrays and ArrayList#^basicarray|BasicArray.java]]，此处在原骨架基础上新增 `changeElement` 方法，并以调用前后的输出对比展示数组作为参数时引用副本指向同一内存的效果。
>
> `BasicArray.java` 演示将整个数组传入方法后，方法内对元素的修改会反映到原数组。
>
> ```java
> public class BasicArray {
>     // 将数组每个元素乘以 2
>     public static void changeElement(int[] arr) {
>         for (int i = 0; i < arr.length; i++) {
>             arr[i] *= 2;  // 直接修改原数组元素
>         }
>     }
>
>     public static void main(String[] args) {
>         final int LIMIT = 5, MULcomparisonLE = 10;
>
>         int[] list = new int[LIMIT];  // 创建长度为 5 的整型数组
>
>         // 初始化数组：list = {0, 10, 20, 30, 40}
>         for (int index = 0; index < LIMIT; index++)
>             list[index] = index * MULcomparisonLE;
>
>         // 打印调用前的数组内容
>         System.out.println("Before calling changeElement:");
>         for (int value : list)
>             System.out.print(value + "  ");
>         System.out.println();
>         System.out.println();
>
>         changeElement(list);  // 传入数组引用
>
>         // 打印调用后的数组内容
>         System.out.println("After calling changeElement:");
>         for (int value : list)
>             System.out.print(value + "  ");
>         System.out.println();
>     }
> }
> ```
>
> 输出：
>
> ```
> Before calling changeElement:
> 0  10  20  30  40
>
> After calling changeElement:
> 0  20  40  60  80
> ```
>
> **数组引用传递 `changeElement(list)`**
>
> ```java
> changeElement(list);
> ```
>
> - `list` 是数组引用，传入方法后形式参数 `arr` 与 `list` 指向同一数组。
> - 方法内 `arr[i] *= 2` 直接作用于原数组的每个元素。
>
> **`changeElement` 方法体**
>
> ```java
> for (int i = 0; i < arr.length; i++)
>     arr[i] *= 2;
> ```
>
> - `arr.length` 获取数组长度，无需额外传入长度参数。
> - `arr[i] *= 2` 等价于 `arr[i] = arr[i] * 2`，每个元素翻倍。

## 9.5 可变长参数列表

可变长参数列表允许方法在每次调用时接收数量不固定的同类型参数，无需为不同数量的参数分别定义重载版本。其声明语法使用 `...` 标记：

```java
<返回类型> <方法名>(<参数类型> ... <参数名>)
```

`...` 告知编译器该参数可接收零个或多个指定类型的值。在方法体内，该参数被视为一个普通数组，可使用 `.length` 获取传入参数的数量，也可用 `for-each` 循环遍历。

```java
public double average(int ... list) {
    // list 在方法体内等同于 int[] list
}
```

调用时直接传入任意数量的对应类型值：

```java
mean1 = average(42, 69, 37);                  // list.length == 3
mean2 = average(35, 43, 93, 23, 40, 21, 75);  // list.length == 7
```

可变长参数与普通参数可以混用，但须遵守以下规则：

- 可变长参数必须位于形式参数列表的最后一位。
- 一个方法只能有一个可变长参数。

```java
// 合法：普通参数在前，可变长参数在后
public double sumDividedBy(int v, int ... list)

// 非法：可变长参数不在最后
public double illegal(int ... list, int v)  // 编译错误
```

> [!note]
> 可变长参数在编译器层面被转换为数组。调用 `average(3, 4)` 等价于编译器自动构造 `new int[]{3, 4}` 并传入，因此也可以直接传入一个已有的数组如 `average(myArray)`。
>
> 同一参数位置不能同时存在数组重载与可变长参数重载，否则编译器会报歧义错误：
>
> ```java
> public void process(int[] arr) { ... }    // 版本 A：接收数组
> public void process(int ... list) { ... } // 版本 B：接收可变长参数
> ```
>
> 当 `process(new int[]{1, 2, 3})` 被调用时，编译器无法确定应该调用版本 A 还是版本 B，因为 `new int[]{1, 2, 3}` 既可以匹配 `int[]`，也可以被视为可变长参数的数组形式。

> [!example]- 示例：`VarLengthArray.java`
>
> `VarLengthArray.java` 演示可变长参数的声明与调用，以及与普通参数混用的方式。
>
> ```java
> public class VarLengthArray {
>     // 计算任意数量整数的平均值
>     public static double average(int ... list) {
>         double sum = 0;
>
>         if (list.length == 0) return sum;  // 无参数时返回 0.0
>
>         for (int value : list)
>             sum += value;
>
>         return sum / list.length;
>     }
>
>     // 计算可变长参数之和再除以固定参数 v
>     public static double sumDividedBy(int v, int ... list) {
>         double sum = 0;
>
>         for (int value : list)
>             sum += value;
>
>         return sum / v;
>     }
>
>     public static void main(String[] args) {
>         System.out.println(average());                 // 无参数
>         System.out.println(average(3, 4));             // 2 个参数
>         System.out.println(average(3, 4, 10, 20, 30)); // 5 个参数
>
>         System.out.println("divided by 4 = " + sumDividedBy(4, 5, 6));    // v=4, list={5,6}
>         System.out.println("divided by 4 = " + sumDividedBy(4, 5, 6, 7)); // v=4, list={5,6,7}
>     }
> }
> ```
>
> 输出：
>
> ```
> 0.0
> 3.5
> 13.4
> divided by 4 = 2.75
> divided by 4 = 4.5
> ```
>
> **`average`：空参数处理**
>
> ```java
> if (list.length == 0) return sum;  // sum 此时为 0.0
> ```
>
> - 调用 `average()` 时 `list` 为长度为 0 的数组，直接返回 `0.0`，避免除以零。
> - 若不做此检查，`sum / list.length` 将触发除以零；`double` 类型下结果为 `NaN`，`int` 类型下则抛出 `ArithmeticException`。
>
> **`average`：累加与计算**
>
> ```java
> for (int value : list)
>     sum += value;
> return sum / list.length;
> ```
>
> - `sum` 声明为 `double`，累加 `int` 值时自动提升，避免整数除法截断。
> - `sum / list.length` 中 `list.length` 为 `int`，`double / int` 结果为 `double`，无需强制转换。
>
> **`sumDividedBy`：混用普通参数与可变长参数**
>
> ```java
> public static double sumDividedBy(int v, int ... list)
> ```
>
> - 调用 `sumDividedBy(4, 5, 6)` 时，`v = 4`，`list = {5, 6}`，结果为 $(5+6)/4 = 2.75$。
> - 调用 `sumDividedBy(4, 5, 6, 7)` 时，`v = 4`，`list = {5, 6, 7}`，结果为 $(5+6+7)/4 = 4.5$。
> - 编译器将第一个 `int` 参数绑定到 `v`，其余所有 `int` 参数收集到 `list` 数组中。

---
`Pre: ` [[ELEC2543 Ch.8 Arrays and ArrayList]]
`Post:` [[ELEC2543 Ch.10 Static Modifier]]

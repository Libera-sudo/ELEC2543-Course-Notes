#Y2S2 #ELEC2543
# Chapter 13. 接口

`Pre: ` [[ELEC2543 Ch.12 Wrapper Class]]
`Post:` [[ELEC2543 Ch.14 Inheritance]]

> [!abstract]
> 接口 (_Interface_) 是 Java 面向对象类型系统中用于定义行为契约的核心语言机制，属于抽象化设计的基础工具。
>
> Java 严格区分"定义行为"与"实现行为"两个层次：接口只声明方法签名，不提供任何实现；具体类通过 `implements` 关键字承诺并完整实现接口中的所有方法。这一设计使不同类的对象能够以统一的接口类型被处理，是多态性的重要实现途径之一。与 Python 的鸭子类型（_Duck Typing_）不同，Java 的接口机制在编译期强制检查类型契约，提供更强的类型安全保证。
>
> 本章涵盖接口的定义语法与约束规则、类实现接口的方式、接口作为类型名与方法参数的用法，以及标准库中 `Comparable` 接口的语义与自定义实现。
```table-of-contents
maxLevel: 3
```
## 13.1 接口的定义与规则

接口 (*Interface*) 是抽象方法 (*Abstract Method*) 与常量的集合，不包含实例变量，也不提供任何方法的具体实现。接口的核心作用是建立一份行为契约：规定实现该接口的类必须提供哪些方法，但不规定这些方法如何实现。

抽象方法是只有方法头、没有方法体的方法声明。接口中所有方法均为抽象方法，因此 `abstract` 修饰符通常省略不写；每个方法头后直接跟分号，不写花括号。

接口的声明语法如下：

```java
public interface Doable
{
    public void doThis();
    public int doThat();
    public void doThis2(float value, char ch);
    public boolean doTheOther(int num);
}
```

- `interface` 是**保留字**，用于声明接口，与 `class` 平级。
- 接口中的方法默认具有 `public` 可见性，即使省略 `public` 修饰符，编译器也会将其视为 `public`。
- 接口不能被实例化，不能写 `new Doable()`。

> [!note]
> 接口不能包含实例变量，但可以包含常量（`public static final` 字段）。接口中声明的字段默认即为 `public static final`，无需显式写出这三个修饰符。

> [!note]
> 接口不能被实例化的原因在于：接口中的方法均无实现，若允许创建接口对象并调用其方法，运行时将无代码可执行。接口只能作为引用变量的类型，实际指向的对象必须是某个实现了该接口的具体类的实例。

## 13.2 类实现接口

一个类通过在类头中使用 `implements` 关键字声明它实现某个接口，并在类体中为接口中的每一个抽象方法提供完整的方法定义。

```java
public class CanDo implements Doable
{
    public void doThis()
    {
        // 具体实现
    }

    public int doThat()
    {
        // 具体实现
    }

    public void doThis2(float value, char ch)
    {
        // 具体实现
    }

    public boolean doTheOther(int num)
    {
        // 具体实现
    }
}
```

- `implements` 是保留字，紧跟在类名之后，接口名之前。
- 实现类必须为接口中声明的每一个方法提供方法体，一个都不能遗漏。
- 实现类可以在接口方法之外额外定义自己的实例变量和方法。

若一个类声明实现某接口但未实现其中所有方法，编译器将报错：

> [!bug]
> ```java
> public class CanDo implements Doable
> {
>     public void doThis() { }
>     // 其余方法未实现
> }
> ```
> ```
> java: CanDo is not abstract and does not override abstract method
> doTheOther(int) in Doable
> ```

一个类可以同时实现多个接口，在 `implements` 子句中以逗号分隔列出所有接口名，并必须实现所有接口中的全部方法：

```java
class ManyThings implements Interface1, Interface2
{
    // 实现 Interface1 和 Interface2 中的所有方法
}
```

> [!note]
> Java 不支持多重类继承（一个类只能 `extends` 一个父类），但允许实现多个接口。
>
> 这是 Java 在保持类型安全的前提下实现多重行为约定的主要机制。

## 13.3 接口作为类型名

接口虽然不能被实例化，但可以作为引用变量的类型使用。一个接口类型的引用变量可以指向任何实现了该接口的类的对象：

> [!example]- 示例：`Hello.java`
>
> ```java
> public interface Hello {
>     public void sayHello();
> }
> ```
>
> **接口的定义**
> - `interface` 替代 `class`，声明这是一个接口而非普通类。
> - `sayHello()` 只有方法头，以分号结尾，无花括号与方法体。

接口可以由不同的类，以不同的变量结构和方式实现：

> [!example]- 示例：`Chinese` / `English` / `Spanish`
>
> `Chinese`、`English`、`Spanish` 三个类均实现了 `Hello` 接口，各自拥有不同的实例变量结构，并以各自的方式实现 `sayHello()`：
>
> ```java
> // Chinese 类：姓名由 familyname 和 givenname 组成
> public class Chinese implements Hello {
>     private String familyname, givenname;
>
>     public Chinese(String familyname, String givenname) {
>         this.familyname = familyname;
>         this.givenname  = givenname;
>     }
>
>     public void sayHello() {
>         System.out.println(familyname + " " + givenname + " says Ni Hao.");
>     }
> }
> ```
>
> ```java
> // English 类：姓名由 lastname、firstname、middlename 组成
> public class English implements Hello {
>     private String lastname, firstname, middlename;
>
>     public English(String lastname, String firstname, String middlename) {
>         this.lastname   = lastname;
>         this.firstname  = firstname;
>         this.middlename = middlename;
>     }
>
>     public void sayHello() {
>         System.out.println(firstname + " " + middlename + " " + lastname + " says Hello.");
>     }
> }
> ```
>
> ```java
> // Spanish 类：姓名由两个父姓和名组成
> public class Spanish implements Hello {
>     private String p1lastname, p2lastname, firstname;
>
>     public Spanish(String p1lastname, String p2lastname, String firstname) {
>         this.p1lastname = p1lastname;
>         this.p2lastname = p2lastname;
>         this.firstname  = firstname;
>     }
>
>     public void sayHello() {
>         System.out.println(firstname + " " + p1lastname + " " + p2lastname + " says Hola.");
>     }
> }
> ```
>
> **各类实现接口的差异**
> ```java
> // Chinese：familyname + givenname
> // English：firstname + middlename + lastname
> // Spanish：firstname + p1lastname + p2lastname
> ```
> - 三个类的实例变量结构完全不同，体现了接口只约束"做什么"而不约束"怎么做"的设计原则。
> - 每个类均提供了 `sayHello()` 的具体实现，满足接口契约。

由于三个类均实现了 `Hello`，可以用 `Hello` 类型的引用变量指向它们的对象：

```java
Hello c1 = new Chinese("Ying", "Zheng");   // Hello 类型引用指向 Chinese 对象
Hello e1 = new English("Obama", "Barack", "Hussein");  // 指向 English 对象
```

这与直接使用具体类型声明引用变量（`Chinese c1 = new Chinese(...)`）在行为上完全等价，输出结果相同。

> [!note]
> 使用接口类型声明引用变量时，该变量只能调用接口中声明的方法，不能调用实现类中额外定义的方法。编译器以引用变量的静态类型（即接口类型）决定可调用的方法集合，而非以实际对象的类型。

接口类型还可以用作方法的参数类型。将参数声明为接口类型，该方法便可接受任何实现了该接口的类的对象，无需为每个具体类单独编写重载版本：

> [!example]- 示例：`HelloDriver.java`
>
> ```java
> public class HelloDriver {
>
>     // 参数类型为接口 Hello，可接受 Chinese、English、Spanish 等任意实现类的对象
>     public static void invoke_method(Hello person) {
>         person.sayHello();
>     }
>
>     public static void main(String[] args) {
>         Chinese c1 = new Chinese("Ying", "Zheng");
>         Chinese c2 = new Chinese("Liu", "Bang");
>         English e1 = new English("Obama", "Barack", "Hussein");
>
>         invoke_method(c1);  // 传入 Chinese 对象
>         invoke_method(c2);  // 传入 Chinese 对象
>         invoke_method(e1);  // 传入 English 对象
>     }
> }
> ```
>
> 输出：
> ```
> Ying Zheng says Ni Hao.
> Liu Bang says Ni Hao.
> Barack Hussein Obama says Hello.
> ```
> **接口类型作为参数**
> ```java
> public static void invoke_method(Hello person) {
>     person.sayHello();
> }
> ```
> - 形式参数 `person` 的类型为接口 `Hello`，调用时可传入 `Chinese`、`English`、`Spanish` 的任意对象。
> - 运行时，JVM 根据 `person` 实际指向的对象类型决定调用哪个类的 `sayHello()` 实现。

> [!note]
> 接口作为参数类型是多态性 (*Polymorphism*) 的直接体现：`invoke_method` 在编译时只知道参数实现了 `Hello` 接口，运行时根据实际对象的类型调用对应的 `sayHello()` 实现。这种"同一方法调用、不同执行结果"的行为正是多态的核心语义。

## 13.4 `Comparable` 接口

Java 标准类库中预定义了许多接口，`Comparable` 是其中最常用的之一。`Comparable` 接口定义了一个抽象方法 `compareTo`，用于比较两个对象的大小顺序：

```java
public interface Comparable {
    int compareTo(Object other);
}
```

`compareTo` 的返回值语义如下：

- 返回负整数：调用者（`obj1`）小于参数（`obj2`）。
- 返回 `0`：两者相等。
- 返回正整数：调用者（`obj1`）大于参数（`obj2`）。

`String` 类已实现 `Comparable`，这正是 11.3 节中 `name1.compareTo(name2)` 能按词典序比较字符串的原因。任何自定义类均可实现 `Comparable`，从而为该类的对象定义"大小"的含义。

实现 `Comparable` 时，`compareTo` 的参数类型为 `Object`，因此方法体内需要先将参数强制转换 (*Cast*) 为目标类型，再访问其字段：

```java
public class Salary implements Comparable {
    private int basic;
    private int bonus;

    public int compareTo(Object other) {
        Salary salary = (Salary) other;  // 将 Object 转换为 Salary 类型
        int totalOfThis  = basic + bonus;
        int totalOfOther = salary.basic + salary.bonus;

        if (totalOfThis < totalOfOther) return -1;
        if (totalOfThis > totalOfOther) return  1;
        return 0;
    }
}
```

- `(Salary) other` 是向下转型 (*Downcast*)，将 `Object` 类型的引用转换为 `Salary` 类型，使编译器允许访问 `salary.basic` 与 `salary.bonus`。
- 若传入的 `other` 实际上不是 `Salary` 对象，运行时将抛出 `ClassCastException`。

`Fraction` 类的 `compareTo` 实现中，由于分数值为浮点数，采用**容差法**判断相等（与 11.1 节的浮点数比较原则一致）：

```java
public int compareTo(Object other) {
    Fraction f = (Fraction) other;

    if (Math.abs(f.value() - value()) < 0.00001) return 0;  // 容差判等
    if (f.value() > value()) return -1;  // other 更大，调用者更小
    return 1;
}
```

> [!note]
> 注意 `Fraction.compareTo` 中的方向：`f.value() > value()` 表示 `other` 大于 `this`，此时 `this` 较小，应返回负值（`-1`）。初学者容易将方向写反。

#### `Comparable` 作为参数类型

将 `Comparable` 用作方法参数类型，可以编写一个对任意实现了 `Comparable` 的类型均适用的通用方法：

> [!example]- 示例：`ComparableDemo.java`
> 演示将 `Comparable` 接口用作参数类型，编写适用于多种类型的通用 `largest` 方法。
>
> ```java
> // ComparableDemo.java —— 使用原始类型 Comparable 作为参数
> public class ComparableDemo {
>
>     // 返回三个 Comparable 对象中最大的一个
>     public static Comparable largest(Comparable c1, Comparable c2, Comparable c3) {
>         if (c1.compareTo(c2) > 0 && c1.compareTo(c3) > 0) return c1;
>         if (c2.compareTo(c1) > 0 && c2.compareTo(c3) > 0) return c2;
>         return c3;
>     }
>
>     public static void main(String[] args) {
>         // int 自动装箱为 Integer，Integer 实现 Comparable
>         int n1 = 1, n2 = 8, n3 = 6;
>         System.out.println("The largest among n1-n3 is " + largest(n1, n2, n3));
>
>         // String 实现 Comparable，按词典序比较
>         String s1 = "apple", s2 = "bye", s3 = "ELEC2543";
>         System.out.println("The largest among s1-s3 is " + largest(s1, s2, s3));
>
>         // Fraction 实现 Comparable，按分数值比较
>         Fraction f1 = new Fraction(1, 2), f2 = new Fraction(2, 3), f3 = new Fraction(1, 3);
>         System.out.println("The largest among f1-f3 is " + largest(f1, f2, f3));
>     }
> }
> ```
>
> 输出：
> ```
> The largest among n1-n3 is 8
> The largest among s1-s3 is bye
> The largest among f1-f3 is 2/3
> ```
> ** `largest` 方法的逻辑**
> ```java
> if (c1.compareTo(c2) > 0 && c1.compareTo(c3) > 0) return c1;
> if (c2.compareTo(c1) > 0 && c2.compareTo(c3) > 0) return c2;
> return c3;
> ```
> - 若 `c1` 同时大于 `c2` 和 `c3`，则 `c1` 最大；若 `c2` 同时大于 `c1` 和 `c3`，则 `c2` 最大；否则 `c3` 最大（含三者相等的情况）。
> - 方法体对所有实现了 `Comparable` 的类型一视同仁，不依赖具体类型。

`largest` 方法接受 `Comparable` 类型参数，因此 `String`、`Fraction`、以及经自动装箱的 `int`（装箱为 `Integer`，而 `Integer` 实现了 `Comparable`）均可传入同一个方法。

> [!question]
> `largest` 的参数类型为 `Comparable`（对象引用），为何可以传入基本类型 `int`？
> 
>> [!check]- 
>> `int` 在传入时被自动装箱为 `Integer`，而 `Integer` 实现了 `Comparable` 接口，因此满足参数类型要求。

#### 泛型写法消除编译警告

如果程序中使用原始类型 (*Raw Type*) `Comparable`，编译器会产生未经检查的类型转换警告（*unchecked warning*），因为无法在编译期保证三个参数属于同一类型。

泛型写法 `<T extends Comparable<T>>` 在编译期约束三个参数必须为同一类型，消除警告：

> [!example]- 示例：`ComparableNoWarning.java`
>
> 演示以对比原始类型写法与泛型写法的差异。
>
> ```java
> // ComparableNoWarning.java —— 使用泛型约束消除编译警告
> public class ComparableNoWarning {
>
>     // T 必须实现 Comparable<T>，三个参数类型在编译期统一
>     public static <T extends Comparable<T>> T largest(T c1, T c2, T c3) {
>         if (c1.compareTo(c2) > 0 && c1.compareTo(c3) > 0) return c1;
>         if (c2.compareTo(c1) > 0 && c2.compareTo(c3) > 0) return c2;
>         return c3;
>     }
>
>     public static void main(String[] args) {
>         int n1 = 1, n2 = 8, n3 = 6;
>         System.out.println("The largest among n1-n3 is " + largest(n1, n2, n3));
>
>         // Double d1 = 10.56;
>         // largest(d1, n2, n3) 编译错误：Double 与 Integer 类型不一致，T 无法统一
>
>         String s1 = "apple", s2 = "bye", s3 = "ELEC2543";
>         System.out.println("The largest among s1-s3 is " + largest(s1, s2, s3));
>
>         // Fraction 使用原始 Comparable，不满足 Comparable<Fraction> 约束，故注释掉
>     }
> }
> ```
>
> 输出：
> ```
> The largest among n1-n3 is 8
> The largest among s1-s3 is bye
> ```
>
> ** `int` 传入 `Comparable` 参数的原因**
> ```java
> int n1 = 1, n2 = 8, n3 = 6;
> largest(n1, n2, n3);  // int 自动装箱为 Integer
> ```
> - `int` 在传入时触发自动装箱，转换为 `Integer` 对象。
> - `Integer` 实现了 `Comparable`，满足参数类型要求。
>
> **泛型写法的类型安全**
> ```java
> public static <T extends Comparable<T>> T largest(T c1, T c2, T c3)
> ```
> - 编译器在调用 `largest(n1, n2, n3)` 时推断 `T = Integer`，在调用 `largest(s1, s2, s3)` 时推断 `T = String`。
> - 若尝试混传不同类型（如 `largest(d1, n2, n3)`），编译器无法找到满足约束的 `T`，直接报错，避免运行时的 `ClassCastException`。

- `<T extends Comparable<T>>` 声明类型参数 `T`，要求 `T` 必须实现 `Comparable<T>`，即 `T` 类型的对象可以与同类型对象比较。
- 编译器在调用处推断 `T` 的具体类型，并检查三个参数是否类型一致。
	- 如 `Integer` 或 `String` 。
- 注释掉的 `largest(d1, n2, n3)` 无法通过编译，因为 `Double` 与 `Integer` 不是同一类型，违反了 `T` 统一的约束。

> [!note]
> `ComparableNoWarning` 中 `Fraction` 的调用被注释掉，原因是课件中的 `Fraction` 实现的是原始类型 `Comparable`（非泛型版 `Comparable<Fraction>`），不满足 `<T extends Comparable<T>>` 的约束，编译器无法推断出合法的 `T`。若将 `Fraction` 改为 `implements Comparable<Fraction>` 并相应修改 `compareTo` 的参数类型为 `Fraction`，则可正常使用泛型版 `largest`。

---
`Pre: ` [[ELEC2543 Ch.12 Wrapper Class]]
`Post:` [[ELEC2543 Ch.14 Inheritance]]

#Y2S2 #ELEC2543
# Chapter 16. 异常

`Pre: ` [[ELEC2543 Ch.15 Polymorphism]]
`Post:` [[ELEC2543 Ch.17 Recursion]]

> [!abstract]
> 异常 (*Exception*) 是面向对象程序设计中用于表达"程序运行至某处无法按预期继续"的机制。异常处理将程序的正常执行流与异常执行流分离，使核心逻辑不必被分散在各处的错误检查打断，是面向对象语言在错误处理上的基本范式。
>
> Java 将异常本身建模为对象，并以 `Throwable` 为根构建完整的类层次，向下分为 `Error` 与 `Exception` 两支；`Exception` 进一步区分受检异常 (*Checked Exception*) 与非受检异常 (*Unchecked Exception*)，前者必须在源码中显式捕获或在方法头的 `throws` 子句中声明，否则编译不通过。这与 C 语言以返回值传递错误码、或 Python 中所有异常默认均为非受检的做法形成对照：Java 的设计将可恢复错误的处理契约提前到编译期，代价是方法签名承担更多显式约束。
>
> 本章涵盖异常机制与调用栈回溯、`try-catch` 语句与多分支捕获、`finally` 子句的清理职责、异常沿调用链的传播、异常类层次与受检/非受检的区分、`throw` 语句与通过继承自定义异常，以及 I/O 异常与 `PrintWriter` 写入文本文件时的异常处理。

```table-of-contents
maxLevel: 3
```

## 16.1 异常机制概述

异常是描述程序运行期间出现的异常或错误情形的对象。程序在检测到此类情形时抛出 (*throw*) 一个异常对象，程序的其他部分可以捕获 (*catch*) 并处理 (*handle*) 该对象。通过这一机制，程序的控制流被清晰地划分为正常执行流与异常执行流两个分支。

Java 同时将错误 (*Error*) 也建模为对象，但错误通常代表无法恢复的情形，应用程序一般不应对其进行捕获。异常与错误共享同一根类 `Throwable`，但在使用契约上分工明确：错误交由运行时环境处理，异常则由程序自身决定处理方式。

Java API 已预定义了一批常见异常，程序在遇到异常时有三种处理方式：忽略该异常、在异常发生处直接处理、在程序的其他位置处理。采取何种方式是重要的设计决策，而非单纯的编码选择。

若异常未被任何 `catch` 子句捕获，程序终止并输出**调用栈回溯** (*Call Stack Trace*)，其中包含触发异常的代码行以及从 `main` 方法到该行的完整方法调用链。调用栈回溯用于定位异常源与传播路径，是未捕获异常最基本的诊断信息。

> [!example]- 示例：`Zero.java`
> 
> ^example-zero-java
> 
> `Zero.java` 故意执行整数除零操作以触发一个未被捕获的 `ArithmeticException`，用于观察 JVM 在异常未处理时输出的终止信息与调用栈回溯。
>
> ```java
> // Zero.java —— 演示未捕获的算术异常
> public class Zero {
>     public static void main(String[] args) {
>         int numerator = 10;
>         int denominator = 0;
>
>         System.out.println(numerator / denominator);  // 触发 ArithmeticException
>         System.out.println("This text will not be printed.");  // 异常抛出后该行不再执行
>     }
> }
> ```
>
> 输出：
>
> ```
> Exception in thread "main" java.lang.ArithmeticException: / by zero
>     at Zero.main(Zero.java:17)
> ```
>
> **异常抛出后续代码不再执行**
>
> ```java
> System.out.println(numerator / denominator);
> System.out.println("This text will not be printed.");
> ```
>
> - 表达式 `numerator / denominator` 在整数除零时由 JVM 抛出 `ArithmeticException`，控制流立即离开 `main` 方法。
> - 由于没有任何 `catch` 子句处理该异常，其后的 `println` 语句不再执行，程序以非零状态终止。
>
> **调用栈回溯的组成**
>
> ```
> Exception in thread "main" java.lang.ArithmeticException: / by zero
>     at Zero.main(Zero.java:17)
> ```
>
> - 第一行给出异常所在线程、异常的完全限定类名与由 JVM 附加的说明消息 `/ by zero`。
> - `at Zero.main(Zero.java:17)` 指出异常发生的方法、源文件与行号；若调用链更深，栈顶为抛出点，向下逐层列出调用者。

> [!note]
> 忽略异常并非使异常"消失"，而是将处理责任默认交给 JVM；JVM 的默认处理方式就是终止程序并打印调用栈回溯。任何未进入 `catch` 子句的异常，最终都会以这种方式结束执行。


## 16.2 `try-catch` 语句

要在程序内部处理异常，需要使用 `try-catch` 语句。该语句由一个 `try` 块与一个或多个 `catch` 子句组成，每个 `catch` 子句声明一个异常类型，称为异常处理器 (*Exception Handler*)。

`try-catch` 语句的一般形式如下：

```java
try {
    <可能抛出异常的语句>
}
catch (<异常类型 1> <形参名>) {
    <针对异常类型 1 的处理语句>
}
catch (<异常类型 2> <形参名>) {
    <针对异常类型 2 的处理语句>
}
```

`try` 块内的语句在执行过程中若抛出异常，控制流立即离开 `try` 块，跳转到与该异常类型相匹配的第一个 `catch` 子句并执行其中的语句。若没有任何 `catch` 子句匹配，该异常将沿调用链继续向外传播。

`catch` 子句的匹配依据异常对象的类型而非字面书写顺序以外的其他因素——只要异常对象的类型与 `catch` 声明的类型兼容（相同类或其子类），该子句即视为匹配。若存在多个潜在匹配，按书写顺序命中首个匹配的子句。

> [!example]- 示例：`ProductCodes.java`
>
> `ProductCodes.java` 循环读取用户输入的产品代码字符串，从固定位置提取区号字符与地区编号，并统计被禁区域的数量。当用户输入长度异常或地区段非数字的代码时，通过两种不同的 `catch` 子句分别捕获 `StringIndexOutOfBoundsException` 与 `NumberFormatException`，使程序在坏输入下继续提示并收尾统计，而不是因一次坏输入直接终止。
>
> ```java
> // ProductCodes.java —— 使用 try-catch 分别处理两类输入异常
> import java.util.Scanner;
>
> public class ProductCodes {
>     public static void main(String[] args) {
>         String code;
>         char zone;
>         int district, valid = 0, banned = 0;
>
>         Scanner scan = new Scanner(System.in);
>
>         System.out.print("Enter product code (XXX to quit): ");
>         code = scan.nextLine();
>
>         while (!code.equals("XXX")) {
>             try {
>                 zone = code.charAt(9);                              // 代码过短时抛出
>                 district = Integer.parseInt(code.substring(3, 7));  // 非数字段时抛出
>                 valid++;
>                 if (zone == 'R' && district > 2000)
>                     banned++;
>             }
>             catch (StringIndexOutOfBoundsException exception) {
>                 System.out.println("Improper code length: " + code);
>             }
>             catch (NumberFormatException exception) {
>                 System.out.println("District is not numeric: " + code);
>             }
>
>             System.out.print("Enter product code (XXX to quit): ");
>             code = scan.nextLine();
>         }
>
>         System.out.println("# of valid codes entered: " + valid);
>         System.out.println("# of banned codes entered: " + banned);
>     }
> }
> ```
>
> 输出：
>
> ```
> Enter product code (XXX to quit): TRV2475A5R-14
> Enter product code (XXX to quit): TRD1704A7R-12
> Enter product code (XXX to quit): TRL2k74A5R-11
> District is not numeric: TRL2k74A5R-11
> Enter product code (XXX to quit): TRQ2949A6M-04
> Enter product code (XXX to quit): TRV2105A2
> Improper code length: TRV2105A2
> Enter product code (XXX to quit): TRQ2778A7R-19
> Enter product code (XXX to quit): XXX
> # of valid codes entered: 4
> # of banned codes entered: 2
> ```
>
> **异常类型决定分支**
>
> ```java
> zone = code.charAt(9);
> district = Integer.parseInt(code.substring(3, 7));
> ```
>
> - `charAt(9)` 在字符串长度不足 10 时抛出 `StringIndexOutOfBoundsException`；`substring(3, 7)` 同样可能越界，但在此例若能走到该行说明索引 $9$ 已可访问，因此越界主要来自 `charAt`。
> - `Integer.parseInt` 在参数不是合法整数字面量时抛出 `NumberFormatException`，这两类异常各由对应的 `catch` 子句处理。
>
> **异常中断 `try` 块的后续语句**
>
> ```java
> valid++;
> if (zone == 'R' && district > 2000)
>     banned++;
> ```
>
> - `valid++` 写在两次可能抛出异常的调用之后，这样"当前代码已通过解析"这一事实才与 `valid` 的递增对齐。
> - 若任一解析抛出异常，控制流跳至对应 `catch` 子句，`valid++` 与禁区判断均不会执行，避免将无效输入计入有效总数。
>
> **`catch` 子句结束后控制流继续**
>
> ```java
> System.out.print("Enter product code (XXX to quit): ");
> code = scan.nextLine();
> ```
>
> - `catch` 子句执行完毕后控制流并不终止，而是落到 `try-catch` 语句之后的代码。
> - 这使循环体能够继续请求下一次输入，实现"坏输入被处理后程序继续推进"的效果。

> [!example]- 示例：`Zero2.java`
>
> `Zero2.java` 在 `Zero.java` 的基础上将除零操作放入 `try-catch` 结构，用 `catch` 子句承接 `ArithmeticException`，以对照捕获与未捕获两种情形下程序控制流的差异。
>
> 此示例引用于 [[ELEC2543 Ch.16 Exceptions#^example-zero-java|16.1 异常机制概述]]，此处在该示例基础上补充异常被捕获后程序仍可继续执行的角度。
>
> ```java
> // Zero2.java —— 用 try-catch 捕获除零异常
> public class Zero2 {
>     public static void main(String[] args) {
>         int numerator = 10;
>         int denominator = 0;
>
>         try {
>             System.out.println(numerator / denominator);  // 抛出 ArithmeticException
>         }
>         catch (ArithmeticException e) {
>             System.out.println("denominator is zero");
>         }
>
>         System.out.println("This text WILL BE printed :-)");  // 异常已处理，后续正常执行
>     }
> }
> ```
>
> 输出：
>
> ```
> denominator is zero
> This text WILL BE printed :-)
> ```
>
> **异常被处理后恢复正常执行流**
>
> ```java
> catch (ArithmeticException e) {
>     System.out.println("denominator is zero");
> }
>
> System.out.println("This text WILL BE printed :-)");
> ```
>
> - `catch` 子句执行完毕后，控制流不再回到 `try` 块中抛出点之后的位置，而是继续执行 `try-catch` 之后的语句。
> - 与 `Zero.java` 的对比表明：异常是否终止程序取决于是否被捕获，而非异常本身的性质。


## 16.3 `finally` 子句

`try` 语句可以可选地附加一个 `finally` 子句，该子句中的语句在 `try-catch` 语句离开前始终执行：若 `try` 块未抛出异常，则在 `try` 块结束后执行；若抛出并被某 `catch` 匹配，则在该 `catch` 执行结束后执行。

`finally` 子句主要用于承载必须在退出 `try-catch` 前完成的收尾工作，典型用途包括关闭网络连接、关闭文件流或释放其他外部资源；无论正常路径还是异常路径，这些资源都必须被释放，因此适合放入始终执行的块中。

> [!example]- 示例：`DemoCatch.java`
>
> `DemoCatch.java` 尝试从名为 `test.txt` 的文件构造 `Scanner` 并读取一个整数，通过两个 `catch` 子句分别处理文件不存在与读入类型不匹配的情形，并在 `finally` 子句中输出固定消息，用以观察 `finally` 子句在异常发生与未发生两种路径下的一致执行行为。
>
> ```java
> // DemoCatch.java —— 演示 try-catch-finally 的控制流
> import java.util.*;
> import java.io.*;
>
> public class DemoCatch {
>     public static void main(String[] args) {
>         Scanner scan;
>
>         try {
>             scan = new Scanner(new File("test.txt"));  // 文件不存在时抛出 FileNotFoundException
>             scan.nextInt();                             // 读到非整数时抛出 InputMismatchException
>         }
>         catch (FileNotFoundException e) {
>             System.out.println("test.txt does not exist.");
>         }
>         catch (InputMismatchException e) {
>             System.out.println("IO exception");
>         }
>         finally {
>             System.out.println("finally");              // 无论是否抛出异常都会执行
>         }
>
>         System.out.println("This text WILL BE printed :-)");
>     }
> }
> ```
>
> 输出（`test.txt` 不存在时）：
>
> ```
> test.txt does not exist.
> finally
> This text WILL BE printed :-)
> ```
>
> **`finally` 的执行时机**
>
> ```java
> catch (FileNotFoundException e) {
>     System.out.println("test.txt does not exist.");
> }
> finally {
>     System.out.println("finally");
> }
> ```
>
> - 若 `try` 块抛出 `FileNotFoundException`，先执行匹配的 `catch` 子句，再执行 `finally` 子句。
> - 若 `try` 块未抛出任何异常，`finally` 子句紧接 `try` 块结束后执行；即使既无异常也无 `catch` 被执行，`finally` 仍然运行。
>
> **`finally` 与 `try-catch` 之后代码的区别**
>
> ```java
> finally {
>     System.out.println("finally");
> }
>
> System.out.println("This text WILL BE printed :-)");
> ```
>
> - `finally` 与 `try-catch` 绑定为同一语句的一部分；即便 `try-catch` 内部发生未被匹配的异常而再次向外抛出，`finally` 仍会在传播前执行。
> - `finally` 之后的语句仅在 `try-catch-finally` 整体未继续抛出异常时才会执行，其执行条件严格弱于 `finally`。


## 16.4 异常传播

若异常在抛出处不便处理，可以让其沿方法调用链向上传播 (*propagate*)，由更高层的方法捕获并处理。异常会逐层离开未捕获它的方法，直至被某个 `catch` 子句匹配，或抵达 `main` 方法仍未被捕获而导致程序终止。

异常传播使异常处理位置与异常发生位置得以分离：低层方法只负责报告问题，如何应对该问题由对调用上下文更了解的高层方法决定。与之对应，调用栈回溯提供了异常实际途经的完整方法链，而不是仅停留在抛出点。

> [!example]- 示例：`Propagation.java` / `ExceptionScope.java`
>
> 两文件共同演示异常的跨层传播：`ExceptionScope` 类中的 `level3` 方法触发除零异常但不处理，该异常经 `level2` 向上传播至 `level1` 中的 `try-catch` 被捕获；`Propagation` 类的 `main` 方法驱动整个调用过程，用以观察异常在未处理方法中"穿过不停留"、在处理方法中"命中并捕获"两种行为的差异。
>
> ```java
> // ExceptionScope.java —— 提供三层方法调用，仅在 level1 处理异常
> public class ExceptionScope {
>     public void level1() {
>         System.out.println("Level 1 beginning.");
>
>         try {
>             level2();  // 调用 level2，期待接收其传播上来的异常
>         }
>         catch (ArithmeticException problem) {
>             System.out.println();
>             System.out.println("The exception message is: " + problem.getMessage());
>             System.out.println();
>             System.out.println("The call stack trace:");
>             problem.printStackTrace();  // 输出完整调用栈
>             System.out.println();
>         }
>
>         System.out.println("Level 1 ending.");
>     }
>
>     // 不捕获异常，仅让其穿过
>     public void level2() {
>         System.out.println("Level 2 beginning.");
>         level3();
>         System.out.println("Level 2 ending.");  // 异常传播时该行不会执行
>     }
>
>     // 实际抛出异常的层
>     public void level3() {
>         int numerator = 10, denominator = 0;
>         System.out.println("Level 3 beginning.");
>         int result = numerator / denominator;  // 抛出 ArithmeticException
>         System.out.println("Level 3 ending.");  // 不会执行
>     }
> }
> ```
>
> ```java
> // Propagation.java —— 驱动类，触发 level1 开始演示
> public class Propagation {
>     public static void main(String[] args) {
>         ExceptionScope demo = new ExceptionScope();
>
>         System.out.println("Program beginning.");
>         demo.level1();
>         System.out.println("Program ending.");
>     }
> }
> ```
>
> 输出：
>
> ```
> Program beginning.
> Level 1 beginning.
> Level 2 beginning.
> Level 3 beginning.
>
> The exception message is: / by zero
>
> The call stack trace:
> java.lang.ArithmeticException: / by zero
>     at ExceptionScope.level3(ExceptionScope.java:54)
>     at ExceptionScope.level2(ExceptionScope.java:41)
>     at ExceptionScope.level1(ExceptionScope.java:18)
>     at Propagation.main(Propagation.java:17)
>
> Level 1 ending.
> Program ending.
> ```
>
> **异常穿过未捕获的方法**
>
> ```java
> public void level2() {
>     System.out.println("Level 2 beginning.");
>     level3();
>     System.out.println("Level 2 ending.");
> }
> ```
>
> - `level2` 内部没有 `try-catch`，其调用的 `level3` 抛出的异常原样向上传递。
> - `Level 2 ending.` 不会被打印，因为控制流在 `level3` 返回前已离开 `level2`；从输出中这一缺失即可判定异常确实穿过而未停留。
>
> **异常在 `level1` 被捕获后程序继续**
>
> ```java
> try {
>     level2();
> }
> catch (ArithmeticException problem) {
>     ...
> }
>
> System.out.println("Level 1 ending.");
> ```
>
> - `level1` 的 `catch` 子句匹配 `ArithmeticException`，传播在此结束，控制流进入 `catch` 块并最终落到 `level1` 末尾的输出语句。
> - `main` 方法因此可以正常打印 `Program ending.`，说明异常被就地消化，未继续上浮到 `main`。
>
> **调用栈回溯反映传播路径**
>
> ```
> at ExceptionScope.level3(ExceptionScope.java:54)
> at ExceptionScope.level2(ExceptionScope.java:41)
> at ExceptionScope.level1(ExceptionScope.java:18)
> at Propagation.main(Propagation.java:17)
> ```
>
> - 栈顶为 `level3`，即实际抛出位置；自上而下列出将控制流带到抛出点的完整调用序列。
> - `printStackTrace` 打印的栈并不因异常已被捕获而缩短，仍反映异常诞生时的完整调用上下文。


## 16.5 异常类层次

Java API 中的异常与错误类之间通过继承组织为统一的类层次；所有异常与错误类均为 `Throwable` 的后代，`Throwable` 向下直接分为 `Error` 与 `Exception` 两支。`Exception` 下进一步派生出 `RuntimeException` 这一特殊子支，其余 `Exception` 的直接子类（如 `IllegalAccessException`、`NoSuchMethodException`、`ClassNotFoundException`）则并列存在于同一层。

课件给出的异常类层次如下：

```
                Object
                  △
                  │
               Throwable
                  △
         ┌────────┴────────┐
         │                 │
       Error           Exception
         △                 △
   ┌─────┼─────┐    ┌──────┼────────────┐
   │     │     │    │                   │
 Linkage Thread VM  RuntimeException    │
 Error   Death Error    △              │
         │              │               ├─ IllegalAccessException
       AWTError         ├─ Arithmetic   ├─ NoSuchMethodException
                        │    Exception  └─ ClassNotFoundException
                        ├─ IndexOutOf
                        │    Bounds
                        │    Exception
                        └─ NullPointer
                             Exception
```

自定义异常通过继承 `Exception` 类或其某个后代类实现；所选父类取决于新异常的使用意图——若希望该异常成为受检异常，直接继承 `Exception`；若希望保持与运行时异常相同的使用形态，继承 `RuntimeException` 或其后代。

#### 受检异常与非受检异常

任一异常要么是受检异常 (*Checked Exception*)，要么是非受检异常 (*Unchecked Exception*)。两者的区别在于编译器的约束强度：

- 受检异常必须被捕获，或必须在任何可能抛出或传播它的方法头中通过 `throws` 子句声明，否则编译错误。
- 非受检异常不要求显式处理，但仍可通过 `try-catch` 捕获；Java 中唯一的非受检异常是 `RuntimeException` 及其所有后代。
- 错误类在"无需显式处理"这一点上与 `RuntimeException` 相似：不应被捕获，也不要求在 `throws` 子句中声明。

`throws` 子句追加在方法头末尾，用于声明该方法可能向调用者抛出的受检异常类型。调用这类方法的代码必须在捕获与再次通过 `throws` 声明之间二选一。

```java
<访问修饰符> <返回类型> <方法名>(<参数列表>) throws <异常类型 1>, <异常类型 2> {
    <方法体>
}
```

> [!example]- 示例：`DemoChecked.java`
>
> `DemoChecked.java` 不使用 `try-catch`，而是在 `main` 方法头中以 `throws FileNotFoundException` 声明将受检异常向上抛出，用以演示受检异常在不被捕获时如何通过 `throws` 子句满足编译约束。
>
> ```java
> // DemoChecked.java —— 以 throws 子句声明受检异常
> import java.util.*;
> import java.io.*;
>
> public class DemoChecked {
>     public static void main(String[] args) throws FileNotFoundException {
>         Scanner scan;
>         scan = new Scanner(new File("test.txt"));  // 可能抛出 FileNotFoundException
>         scan.nextInt();
>     }
> }
> ```
>
> **`throws` 子句的作用**
>
> ```java
> public static void main(String[] args) throws FileNotFoundException
> ```
>
> - `new Scanner(new File(...))` 可能抛出受检异常 `FileNotFoundException`；若不捕获，必须在方法头中声明。
> - `throws` 子句将处理契约转交给调用者；对于 `main` 方法而言，实际"调用者"是 JVM，未处理的异常最终仍以程序终止的形式暴露。

> [!question] 习题：受检异常与非受检异常判别
> 下列异常类分别属于受检异常还是非受检异常？
>
> - `NullPointerException`
> - `IndexOutOfBoundsException`
> - `ClassNotFoundException`
> - `NoSuchMethodException`
> - `ArithmeticException`
>
> > [!check]-
> > 判断依据为：凡 `RuntimeException` 及其后代均为非受检异常；其余 `Exception` 的后代均为受检异常。
> >
> > - `NullPointerException`：`RuntimeException` 的后代，非受检。
> > - `IndexOutOfBoundsException`：`RuntimeException` 的后代，非受检。
> > - `ClassNotFoundException`：直接继承自 `Exception`，受检。
> > - `NoSuchMethodException`：直接继承自 `Exception`，受检。
> > - `ArithmeticException`：`RuntimeException` 的后代，非受检。


## 16.6 `throw` 语句与自定义异常

异常由 `throw` 语句显式抛出。`throw` 语句的形式如下：

```java
throw <异常对象表达式>;
```

`throw` 语句通常嵌在 `if` 语句中，仅当某个条件满足时才执行，否则程序将无条件抛出异常，后续代码成为永远无法到达的死代码。

自定义异常通过继承 `Exception`（或其后代）实现。最简自定义异常仅需提供一个接收消息字符串的构造器，将消息通过 `super(message)` 交由父类保存，之后即可通过 `getMessage` 读取。

> [!example]- 示例：`CreatingExceptions.java` / `OutOfRangeException.java`
>
> 两文件配合演示自定义异常的定义与抛出：`OutOfRangeException` 通过继承 `Exception` 定义一个新的受检异常；`CreatingExceptions` 读取一个整数，若超出 $[25, 40]$ 范围则抛出 `OutOfRangeException`，在 `main` 方法中通过 `throws` 子句将其向上传递。
>
> ```java
> // OutOfRangeException.java —— 表示取值越界的自定义受检异常
> public class OutOfRangeException extends Exception {
>     OutOfRangeException(String message) {
>         super(message);  // 将消息交由 Exception 的构造器保存
>     }
> }
> ```
>
> ```java
> // CreatingExceptions.java —— 创建并条件性抛出自定义异常
> import java.util.Scanner;
>
> public class CreatingExceptions {
>     public static void main(String[] args) throws OutOfRangeException {
>         final int MIN = 25, MAX = 40;
>
>         Scanner scan = new Scanner(System.in);
>
>         OutOfRangeException problem =
>             new OutOfRangeException("Input value is out of range.");
>
>         System.out.print("Enter an integer value between " + MIN +
>                          " and " + MAX + ", inclusive: ");
>         int value = scan.nextInt();
>
>         if (value < MIN || value > MAX)  // 仅在越界时抛出
>             throw problem;
>
>         System.out.println("End of main method.");  // 越界时不会执行
>     }
> }
> ```
>
> 输出（输入 `69`）：
>
> ```
> Enter an integer value between 25 and 40, inclusive: 69
> Exception in thread "main" OutOfRangeException: Input value is out of range.
>     at CreatingExceptions.main(CreatingExceptions.java:20)
> ```
>
> **自定义异常的最小实现**
>
> ```java
> public class OutOfRangeException extends Exception {
>     OutOfRangeException(String message) {
>         super(message);
>     }
> }
> ```
>
> - 继承 `Exception` 使 `OutOfRangeException` 成为受检异常，所有可能抛出它的方法必须捕获或在 `throws` 中声明。
> - 构造器仅将消息字符串转交父类保存，`getMessage` 便能返回该消息，无需额外字段。
>
> **条件抛出**
>
> ```java
> if (value < MIN || value > MAX)
>     throw problem;
> ```
>
> - `throw` 语句被放入 `if` 判断之后，只有输入越界时才真正抛出；合法输入下程序继续执行后续语句。
> - 异常对象可以在抛出前一次性创建好，也可以在判断体内通过 `throw new OutOfRangeException(...)` 就地构造，两者在语义上等价。
>
> **受检异常的声明义务**
>
> ```java
> public static void main(String[] args) throws OutOfRangeException
> ```
>
> - `main` 方法既没有将抛出点放入 `try-catch`，也就必须在方法头中通过 `throws` 声明该异常，否则无法通过编译。
> - 未处理的受检异常传播到 `main` 之外时，JVM 按常规流程终止程序并打印调用栈回溯。

> [!example]- 示例：`MakeSureInputCorrect.java`
>
> `MakeSureInputCorrect.java` 在 `CreatingExceptions` 的基础上使用循环与 `try-catch` 组合，实现"越界则提示并重试、合法则返回并退出循环"的交互模式。自定义异常仍来自 `OutOfRangeException`，但抛出后改由 `main` 方法就地捕获而非继续向上传播。
>
> ```java
> // MakeSureInputCorrect.java —— 以 try-catch + 循环实现输入重试
> import java.util.Scanner;
>
> public class MakeSureInputCorrect {
>     final static int MIN = 25, MAX = 40;
>
>     // 读取一次输入，越界时抛出受检异常交由调用者处理
>     public static int userInput() throws OutOfRangeException {
>         Scanner scan = new Scanner(System.in);
>
>         OutOfRangeException problem =
>             new OutOfRangeException("Input value is out of range.");
>
>         System.out.print("Enter an integer value between " + MIN +
>                          " and " + MAX + ", inclusive: ");
>
>         int value = scan.nextInt();
>
>         if (value < MIN || value > MAX)
>             throw problem;
>         else
>             return value;
>     }
>
>     public static void main(String[] args) {
>         int value;
>         boolean done = false;
>
>         while (!done) {
>             try {
>                 value = userInput();  // 合法时返回并跳过 catch
>                 done = true;          // 退出循环
>             }
>             catch (OutOfRangeException e) {
>                 System.out.println("The input was invalid");
>             }
>         }
>
>         System.out.println("End of main method.");
>     }
> }
> ```
>
> **异常驱动的控制流**
>
> ```java
> while (!done) {
>     try {
>         value = userInput();
>         done = true;
>     }
>     catch (OutOfRangeException e) {
>         System.out.println("The input was invalid");
>     }
> }
> ```
>
> - `userInput` 合法返回时 `done` 被置为 `true`，循环退出；越界时 `throw` 跳过 `done = true` 直接进入 `catch`，`done` 保持 `false`，循环再跑一次。
> - 异常在此承担了"非法分支"的控制角色，与合法分支通过同一 `return` 表达不同出口。
>
> **调用者与抛出者的分工**
>
> ```java
> public static int userInput() throws OutOfRangeException { ... }
> ```
>
> - `userInput` 声明 `throws OutOfRangeException`，意味着它本身不处理越界情形，只负责报告。
> - `main` 方法选择在调用处 `catch` 该异常，从而将"如何应对错误"的决定保留在更了解交互上下文的一侧。

> [!question] 习题：无条件 `throw` 的问题
> 下列代码存在什么问题？
>
> ```java
> System.out.println("Before throw");
> throw new OutOfRangeException("Too High");
> System.out.println("After throw");
> ```
>
> > [!check]-
> > `throw` 语句无条件执行，程序到达该行时必定抛出 `OutOfRangeException`，后续的 `System.out.println("After throw")` 成为不可达代码，Java 编译器将直接拒绝编译。
> >
> > 要让后续语句可能被执行，`throw` 必须置于条件判断内部，使其仅在需要时才触发。


## 16.7 I/O 异常

在讨论异常与输入输出的交互时，需要先明确流 (*Stream*) 的概念。流是从源流向目的地的字节序列：程序从输入流读取信息，向输出流写入信息；一个程序可以同时管理多个流。

Java 提供三种标准 I/O 流：

- 标准输出 (*standard output*)：由 `System.out` 定义，`println` 语句即向该流写出；通常指向控制台窗口。
- 标准输入 (*standard input*)：由 `System.in` 定义，`Scanner` 在前置章节中即通过该流读取键盘输入。
- 标准错误 (*standard error*)：由 `System.err` 定义，通常也指向控制台窗口，用于与普通输出区分的诊断信息。

某些 I/O 操作可能抛出 `IOException`，典型原因包括目标文件不存在、即使存在程序也无法定位该文件、或文件内容的类型不符合预期。`IOException` 是受检异常，使用这些 I/O 操作的方法必须在 `try-catch` 中处理或在 `throws` 子句中声明。

#### 写入文本文件

在 [[ELEC2543 Ch.5 Data Visibility|Ch.5]] 中已经通过 `Scanner` 从文本文件读取输入。与之对应，`PrintWriter` 类用于表示一个文本输出文件，将写入的字符顺序输出到目标文件。写入完成后，输出流应当显式关闭；未关闭的输出流可能因缓冲导致内容未落盘，或持有操作系统资源无法及时释放。

> [!example]- 示例：`TestData.java`
>
> `TestData.java` 使用 `PrintWriter` 创建文本输出文件 `test.txt`，向其中写入 10 行、每行 10 个 $[10, 99]$ 范围内的随机整数，用以演示文本输出流的创建、使用与关闭，并通过 `main` 方法头的 `throws IOException` 声明应对 `PrintWriter` 构造时可能抛出的受检 I/O 异常。
>
> ```java
> // TestData.java —— 生成随机测试数据文本文件
> import java.util.Random;
> import java.io.*;
>
> public class TestData {
>     public static void main(String[] args) throws IOException {
>         final int MAX = 10;
>         int value;
>         String fileName = "test.txt";
>
>         PrintWriter outFile = new PrintWriter(fileName);  // 可能抛出 IOException 的受检异常
>
>         Random rand = new Random();
>
>         for (int line = 1; line <= MAX; line++) {
>             for (int num = 1; num <= MAX; num++) {
>                 value = rand.nextInt(90) + 10;  // 生成 [10, 99] 的随机整数
>                 outFile.print(value + "   ");
>             }
>             outFile.println();
>         }
>
>         outFile.close();  // 显式关闭，确保内容落盘并释放资源
>         System.out.println("Output file has been created: " + fileName);
>     }
> }
> ```
>
> 输出：
>
> ```
> Output file has been created: test.txt
> ```
>
> `test.txt` 示例内容：
>
> ```
> 77 46 24 67 45 37 32 40 39 10
> 90 91 71 64 82 80 68 18 83 89
> 25 80 45 75 74 40 15 90 79 59
> 44 43 95 85 93 61 15 20 52 86
> 60 85 18 73 56 41 35 67 21 42
> 93 25 89 47 13 27 51 94 76 13
> 33 25 48 42 27 24 88 18 32 17
> 71 10 90 88 60 19 89 54 21 92
> 45 26 47 68 55 98 34 38 98 38
> 48 59 90 12 86 36 11 65 41 62
> ```
>
> **`PrintWriter` 构造与受检异常声明**
>
> ```java
> public static void main(String[] args) throws IOException {
>     ...
>     PrintWriter outFile = new PrintWriter(fileName);
> ```
>
> - `PrintWriter(String)` 在无法创建或打开目标文件时抛出 `FileNotFoundException`，后者是 `IOException` 的后代。
> - `main` 方法通过 `throws IOException` 一次覆盖所有可能的 I/O 受检异常，免去对具体子类的逐一声明。
>
> **`PrintWriter` 的写入接口**
>
> ```java
> outFile.print(value + "   ");
> outFile.println();
> ```
>
> - `PrintWriter` 对外提供与 `System.out` 同名的 `print` 与 `println` 方法；差别仅在于写入目标从标准输出改为文件。
> - 同名 API 使得原先面向控制台写出的代码可以在几乎不改动逻辑的情况下切换为写入文件。
>
> **显式关闭输出流**
>
> ```java
> outFile.close();
> ```
>
> - 关闭操作会刷新输出缓冲区，确保已写入但尚未落盘的字符被真正写入文件。
> - 若省略 `close`，程序结束时部分内容可能因缓冲未刷新而丢失；在长期运行的程序中还会持续占用文件句柄等操作系统资源。


---
`Pre: ` [[ELEC2543 Ch.15 Polymorphism]]
`Post:` [[ELEC2543 Ch.17 Recursion]]

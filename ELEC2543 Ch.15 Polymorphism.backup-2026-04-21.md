#Y2S2 #ELEC2543
# Chapter 15. 多态性

`Pre: ` [[ELEC2543 Ch.14 Inheritance]]
`Post:` [[ELEC2543 Ch.16 Exceptions]]

> [!abstract]
> 多态性 (_Polymorphism_) 是面向对象程序设计中描述"同一操作作用于不同类型对象时产生不同行为"这一现象的核心概念，是 Java 类型系统灵活性的集中体现。
>
> Java 通过延迟方法绑定 (_Late Binding_) 实现多态：方法调用在编译期仅做类型检查，实际绑定到哪个方法体由运行时对象的真实类型决定。这与 Python 的鸭子类型在效果上相似，但 Java 在编译期严格约束引用变量只能调用其静态类型所声明的方法，提供更强的类型安全保证。
>
> 本章涵盖动态绑定的机制与意义、通过继承建立多态引用、通过接口建立多态引用，以及将多态性应用于选择排序 (_Selection Sort_) 算法的通用实现。
```table-of-contents
maxLevel: 3
```
## 15.1 动态绑定

多态性 (*Polymorphism*) 字面含义为"具有多种形态"，在 Java 中指同一方法调用在运行时根据对象的实际类型执行不同的方法体。多态引用 (*Polymorphic Reference*) 是一个可以在不同时刻指向不同类型对象的引用变量。

方法绑定 (*Binding*) 是指将一次方法调用与其对应的方法体关联起来的过程。考虑以下调用：

```java
obj.doIt();
```

- 若绑定发生在编译期（静态绑定），则该行代码每次执行时都调用同一个方法体，无法根据对象类型变化。
- Java 将方法绑定推迟到运行时，称为动态绑定 (*Dynamic Binding*) 或延迟绑定 (*Late Binding*)。运行时，JVM 检查 `obj` 实际指向的对象类型，调用该类型对应的方法体。

Java 中所有对象引用都具有潜在的多态性。例如：

```java
Occupation job;
```

`job` 可以指向 `Occupation` 对象，也可以指向任何与 `Occupation` 兼容的子类或实现类的对象。这种兼容性通过继承或接口建立。

> [!note]
> 动态绑定是多态性的底层机制。正是因为方法绑定推迟到运行时，同一行 `job.someMethod()` 才能在 `job` 指向不同对象时执行不同的代码，这是多态性的本质。

## 15.2 通过继承实现多态

父类类型的引用变量可以指向任何子类的对象，这是继承建立的 is-a 关系在引用赋值上的直接体现。

```java
Holiday day;
day = new Christmas();  // 合法：Christmas is-a Holiday
```

将子类对象赋给父类引用是简单赋值，无需任何转型操作。但若将将父类对象反向赋给子类引用，则必须使用强制转型 (*Cast*)。

这在逻辑上通常是不合理的，因为并非所有 `Holiday` 都是 `Christmas`。

```java
Holiday day = new Christmas();
Christmas xmas = (Christmas) day;  // 向下转型，需显式 cast
```

通过父类引用调用方法时，JVM 在运行时根据引用实际指向的对象类型决定调用哪个版本的方法。假设 `Holiday` 定义了 `celebrate()` 方法，`Christmas` 对其进行了覆写 (*Overriding*)：

```java
Holiday day = new Christmas();
day.celebrate();  // 运行时调用 Christmas.celebrate()
```

编译器以引用变量的静态类型（即声明类型）决定哪些方法可以被调用。若 `Christmas` 定义了 `Holiday` 中不存在的方法 `getTree()`，通过 `Holiday` 类型的引用调用该方法将导致编译错误：

```java
day.getTree();              // 编译错误：Holiday 中未声明 getTree()
((Christmas) day).getTree(); // 合法：向下转型后，编译器以 Christmas 类型检查
```

> [!note]
> 编译器与 JVM 在多态中的分工：
> - 编译器基于引用的**静态类型**（声明类型）检查方法调用是否合法，决定"能调用什么"。
> - JVM 基于引用实际指向的**动态类型**（对象类型）决定执行哪个方法体，决定"调用哪个版本"。
>
> 这两步分离是 Java 类型安全与多态灵活性并存的关键。

> [!example]- 示例：`Firm.java` / `Staff.java` / `StaffMember.java` / `Volunteer.java` / `Employee.java` / `Executive.java` / `Hourly.java`
> 演示通过继承层次结构中的父类引用数组实现多态方法调用，同一 `pay()` 调用对不同子类对象产生不同行为。
>
> 类层次结构如下：
> ```
>         +---------------------+
>         |    StaffMember      |  abstract
>         |---------------------|
>         | # name: String      |
>         | # address: String   |
>         | # phone: String     |
>         |---------------------|
>         | + toString(): String|
>         | + pay(): double     |  abstract
>         +---------------------+
>                   △
>          ┌────────┴────────┐
>          │                 │
>  +-------------+   +------------------+
>  |  Volunteer  |   |    Employee      |
>  |-------------|   |------------------|
>  |             |   | # socialSec...   |
>  |-------------|   | # payRate        |
>  | + pay()     |   |------------------|
>  +-------------+   | + toString()     |
>                    | + pay()          |
>                    +------------------+
>                             △
>                    ┌────────┴────────┐
>                    │                 │
>            +-------------+   +-------------+
>            |  Executive  |   |   Hourly    |
>            |-------------|   |-------------|
>            | - bonus     |   | - hoursWork.|
>            |-------------|   |-------------|
>            | + awardBon. |   | + addHours()|
>            | + pay()     |   | + pay()     |
>            +-------------+   | + toString()|
>                              +-------------+
>
> ```
>
> ```java
> // StaffMember.java —— 抽象父类，定义通用字段与抽象方法 pay()
> abstract public class StaffMember
> {
>     protected String name;
>     protected String address;
>     protected String phone;
>
>     public StaffMember(String eName, String eAddress, String ePhone)
>     {
>         name = eName;
>         address = eAddress;
>         phone = ePhone;
>     }
>
>     public String toString()
>     {
>         String result = "Name: " + name + "\n";
>         result += "Address: " + address + "\n";
>         result += "Phone: " + phone;
>         return result;
>     }
>
>     // 抽象方法：由子类各自实现薪资计算逻辑
>     public abstract double pay();
> }
> ```
>
> ```java
> // Volunteer.java —— 志愿者，pay() 返回 0
> public class Volunteer extends StaffMember
> {
>     public Volunteer(String eName, String eAddress, String ePhone)
>     {
>         super(eName, eAddress, ePhone);
>     }
>
>     public double pay()
>     {
>         return 0.0;
>     }
> }
> ```
>
> ```java
> // Employee.java —— 普通雇员，pay() 返回固定薪资
> public class Employee extends StaffMember
> {
>     protected String socialSecurityNumber;
>     protected double payRate;
>
>     public Employee(String eName, String eAddress, String ePhone, String socSecNumber, double rate)
>     {
>         super(eName, eAddress, ePhone);
>         socialSecurityNumber = socSecNumber;
>         payRate = rate;
>     }
>
>     public String toString()
>     {
>         String result = super.toString();
>         result += "\nSocial Security Number: " + socialSecurityNumber;
>         return result;
>     }
>
>     public double pay()
>     {
>         return payRate;
>     }
> }
> ```
>
> ```java
> // Executive.java —— 高管，pay() 在普通薪资基础上加一次性奖金
> public class Executive extends Employee
> {
>     private double bonus;
>
>     public Executive(String eName, String eAddress, String ePhone, String socSecNumber, double rate)
>     {
>         super(eName, eAddress, ePhone, socSecNumber, rate);
>         bonus = 0;  // 奖金尚未发放
>     }
>
>     public void awardBonus(double execBonus)
>     {
>         bonus = execBonus;
>     }
>
>     public double pay()
>     {
>         double payment = super.pay() + bonus;  // 基本薪资 + 奖金
>         bonus = 0;  // 奖金为一次性，发放后清零
>         return payment;
>     }
> }
> ```
>
> ```java
> // Hourly.java —— 时薪雇员，pay() 按时薪 × 工时计算
> public class Hourly extends Employee
> {
>     private int hoursWorked;
>
>     public Hourly(String eName, String eAddress, String ePhone, String socSecNumber, double rate)
>     {
>         super(eName, eAddress, ePhone, socSecNumber, rate);
>         hoursWorked = 0;
>     }
>
>     public void addHours(int moreHours)
>     {
>         hoursWorked += moreHours;
>     }
>
>     public double pay()
>     {
>         double payment = payRate * hoursWorked;
>         hoursWorked = 0;  // 发薪后清零工时
>         return payment;
>     }
>
>     public String toString()
>     {
>         String result = super.toString();
>         result += "\nCurrent hours: " + hoursWorked;
>         return result;
>     }
> }
> ```
>
> ```java
> // Staff.java —— 管理员工数组，演示多态调用
> public class Staff
> {
>     private StaffMember[] staffList;
>
>     public Staff()
>     {
>         staffList = new StaffMember[6];
>
>         // 父类数组中存储不同子类对象
>         staffList[0] = new Executive("Sam", "123 Main Line",
>             "555-0469", "123-45-6789", 2423.07);
>         staffList[1] = new Employee("Carla", "456 Off Line",
>             "555-0101", "987-65-4321", 1246.15);
>         staffList[2] = new Employee("Woody", "789 Off Rocker",
>             "555-0000", "010-20-3040", 1169.23);
>         staffList[3] = new Hourly("Diane", "678 Fifth Ave.",
>             "555-0690", "958-47-3625", 10.55);
>         staffList[4] = new Volunteer("Norm", "987 Suds Blvd.",
>             "555-8374");
>         staffList[5] = new Volunteer("Cliff", "321 Duds Lane",
>             "555-7282");
>
>         // 向下转型后调用子类特有方法
>         ((Executive) staffList[0]).awardBonus(500.00);
>         ((Hourly) staffList[3]).addHours(40);
>     }
>
>     public void payday()
>     {
>         double amount;
>
>         for (int count = 0; count < staffList.length; count++)
>         {
>             System.out.println(staffList[count]);  // 多态调用 toString()
>
>             amount = staffList[count].pay();  // 多态调用 pay()
>
>             if (amount == 0.0)
>                 System.out.println("Thanks!");
>             else
>                 System.out.println("Paid: " + amount);
>
>             System.out.println("-----------------------------------");
>         }
>     }
> }
> ```
>
> ```java
> // Firm.java —— 驱动类
> public class Firm
> {
>     public static void main(String[] args)
>     {
>         Staff personnel = new Staff();
>         personnel.payday();
>     }
> }
> ```
>
> 输出：
> ```
> Name: Sam
> Address: 123 Main Line
> Phone: 555-0469
> Social Security Number: 123-45-6789
> Paid: 2923.07
> -----------------------------------
> Name: Carla
> Address: 456 Off Line
> Phone: 555-0101
> Social Security Number: 987-65-4321
> Paid: 1246.15
> -----------------------------------
> Name: Woody
> Address: 789 Off Rocker
> Phone: 555-0000
> Social Security Number: 010-20-3040
> Paid: 1169.23
> -----------------------------------
> Name: Diane
> Address: 678 Fifth Ave.
> Phone: 555-0690
> Social Security Number: 958-47-3625
> Current hours: 40
> Paid: 422.0
> -----------------------------------
> Name: Norm
> Address: 987 Suds Blvd.
> Phone: 555-8374
> Thanks!
> -----------------------------------
> Name: Cliff
> Address: 321 Duds Lane
> Phone: 555-7282
> Thanks!
> -----------------------------------
> ```
>
> **父类数组存储不同子类对象**
> ```java
> private StaffMember[] staffList;
> staffList[0] = new Executive(...);
> staffList[1] = new Employee(...);
> staffList[3] = new Hourly(...);
> staffList[4] = new Volunteer(...);
> ```
> - 数组元素类型为 `StaffMember`（抽象父类），但每个元素实际指向不同子类的对象。
> - 这是多态的典型应用场景：用统一的父类类型管理一组异构对象。
>
> **多态调用 `pay()` **
> ```java
> amount = staffList[count].pay();
> ```
> - 编译器检查 `StaffMember` 中是否声明了 `pay()` 方法——存在（抽象方法），编译通过。
> - 运行时，JVM 根据 `staffList[count]` 实际指向的对象类型决定调用哪个 `pay()` 版本：
>   - `Executive`：`super.pay() + bonus` = $2423.07 + 500.00 = 2923.07$
>   - `Employee`：直接返回 `payRate`
>   - `Hourly`：`payRate * hoursWorked` = $10.55 \times 40 = 422.0$
>   - `Volunteer`：返回 `0.0`
>
> **多态调用 `toString()` **
> ```java
> System.out.println(staffList[count]);
> ```
> - `println` 内部调用对象的 `toString()` 方法。
> - `Executive` 未覆写 `toString()`，沿继承链向上查找，使用 `Employee.toString()`（输出姓名、地址、电话、社保号）。
> - `Hourly` 覆写了 `toString()`，在 `Employee.toString()` 基础上追加工时信息。
> - `Volunteer` 未覆写 `toString()`，使用 `StaffMember.toString()`（仅输出姓名、地址、电话）。
>
> **向下转型调用子类特有方法**
> ```java
> ((Executive) staffList[0]).awardBonus(500.00);
> ((Hourly) staffList[3]).addHours(40);
> ```
> - `awardBonus` 是 `Executive` 特有方法，`addHours` 是 `Hourly` 特有方法，均不在 `StaffMember` 中声明。
> - 通过 `StaffMember` 引用无法直接调用这些方法，必须先向下转型为具体子类类型。
> - 若转型目标与对象实际类型不匹配（如将 `staffList[1]` 转为 `Executive`），运行时抛出 `ClassCastException`。
>
> ** `Executive.pay()` 中 `super.pay()` 的复用**
> ```java
> public double pay()
> {
>     double payment = super.pay() + bonus;
>     bonus = 0;
>     return payment;
> }
> ```
> - `super.pay()` 调用 `Employee.pay()`，返回 `payRate`（$2423.07$）。
> - `Executive` 在此基础上加上奖金，发放后将 `bonus` 清零，确保奖金为一次性。

> [!note]
> `payday()` 方法中使用 `amount == 0.0` 判断是否为志愿者。在实际开发中，浮点数的 `==` 比较存在精度风险（参见 11.1 节）。此处因 `Volunteer.pay()` 直接返回字面量 `0.0`，不涉及浮点运算，故比较结果可靠。但若 `pay()` 的返回值经过计算得出，应改用容差法或 `instanceof` 判断类型。

## 15.3 通过接口实现多态

接口同样可以作为引用变量的类型，建立多态引用。接口类型的引用变量可以指向任何实现了该接口的类的对象，方法调用在运行时根据对象的实际类型动态绑定。

假设声明如下接口：

```java
public interface Speaker
{
    public void speak();
    public void announce(String str);
}
```

用接口类型声明引用变量：

```java
Speaker current;
```

`current` 可以指向任何实现了 `Speaker` 接口的类的对象。假设 `Philosopher` 与 `Dog` 均实现了 `Speaker`，各自提供不同的 `speak()` 实现：

```java
Speaker guest = new Philosopher();
guest.speak();   // 调用 Philosopher.speak()

guest = new Dog();
guest.speak();   // 调用 Dog.speak()
```

同一引用变量 `guest` 在两次调用之间指向了不同类型的对象，`speak()` 的执行结果也随之不同，这正是多态引用的行为。

与继承多态相同，编译器以引用变量的静态类型（接口类型）限制可调用的方法集合。若 `Philosopher` 额外定义了 `pontificate()` 方法，但该方法未在 `Speaker` 接口中声明，则通过 `Speaker` 类型引用调用该方法将导致编译错误：

```java
Speaker special = new Philosopher();
special.pontificate();  // 编译错误：Speaker 中未声明 pontificate()
```

接口引用的赋值兼容性规则与继承引用相同：

```java
Speaker first = new Dog();          // 合法：Dog 实现了 Speaker
Philosopher second = new Philosopher();
second.pontificate();               // 合法：通过具体类型引用调用
first = second;                     // 合法：Philosopher 实现了 Speaker，可赋给 Speaker 引用
```

> [!note]
> 通过继承建立的多态与通过接口建立的多态在机制上完全一致，均依赖动态绑定。两者的区别在于兼容性的来源：继承多态要求对象类型与引用类型之间存在 is-a 关系；接口多态要求对象所属的类实现了引用变量声明的接口类型。

## 15.4 多态性在排序中的应用

排序 (*Sorting*) 是将一组元素按特定顺序排列的过程。排序的依据因场景而异：数值按大小、字符串按词典序、自定义对象按业务规则定义的顺序。将多态性与 `Comparable` 接口结合，可以编写一个对任意可比较类型均适用的通用排序方法。

#### 选择排序 (*Selection Sort*)

选择排序的策略是每轮从未排序部分找出最小值，将其交换到已排序部分的末尾，重复直至所有元素就位。

具体步骤：
- 从整个数组中找到最小值，与第 0 个位置的元素交换。
- 从第 1 个位置起的剩余部分找到最小值，与第 1 个位置的元素交换。
- 重复上述过程，每轮将未排序部分缩短一个元素，直至只剩一个元素。

以数组 `[3, 9, 6, 1, 2]` 为例，排序过程如下：

| 轮次 | 操作 | 数组状态 |
|------|------|----------|
| 初始 | — | `3  9  6  1  2` |
| 第 1 轮 | 最小值 `1`（位置 3）与位置 0 的 `3` 交换 | `1  9  6  3  2` |
| 第 2 轮 | 最小值 `2`（位置 4）与位置 1 的 `9` 交换 | `1  2  6  3  9` |
| 第 3 轮 | 最小值 `3`（位置 3）与位置 2 的 `6` 交换 | `1  2  3  6  9` |
| 第 4 轮 | 最小值 `6`（位置 3）与位置 3 的 `6` 交换（原地） | `1  2  3  6  9` |

交换两个元素需要借助临时变量，直接互赋会导致其中一个值被覆盖：

```java
temp   = first;   // 保存 first 的值
first  = second;  // 用 second 覆盖 first
second = temp;    // 用保存的原 first 值覆盖 second
```

#### 插入排序 (*Insertion Sort*)

插入排序的策略是维护一个已排序的前缀，每次将下一个元素插入到前缀中的正确位置。

具体步骤：
- 从第 1 个元素开始，将其视为"待插入键"。
- 将键与其左侧已排序部分逐一比较，将所有大于键的元素向右移动一位。
- 将键插入空出的位置。
- 重复直至所有元素处理完毕。

> [!example]- 示例：`PhoneList.java` / `Sorting.java` / `Contact.java`
> 演示将 `Comparable` 接口用作排序方法的参数类型，实现对任意实现了 `Comparable` 的对象数组进行排序；`Contact` 类通过实现 `compareTo` 定义按姓氏（同姓再按名字）排序的规则。
>
> ```java
> // Contact.java —— 实现 Comparable，定义联系人的比较规则
> public class Contact implements Comparable
> {
>     private String firstName, lastName, phone;
>
>     public Contact(String first, String last, String telephone)
>     {
>         firstName  = first;
>         lastName   = last;
>         phone      = telephone;
>     }
>
>     // 输出格式：姓, 名\t电话
>     public String toString()
>     {
>         return lastName + ", " + firstName + "\t" + phone;
>     }
>
>     // 按姓名判断相等：姓与名均相同则视为同一联系人
>     public boolean equals(Object other)
>     {
>         return (lastName.equals(((Contact) other).getLastName()) &&
>                 firstName.equals(((Contact) other).getFirstName()));
>     }
>
>     // 比较规则：先按姓氏，同姓再按名字，均使用 String.compareTo（词典序）
>     public int compareTo(Object other)
>     {
>         int result;
>         String otherFirst = ((Contact) other).getFirstName();
>         String otherLast  = ((Contact) other).getLastName();
>
>         if (lastName.equals(otherLast))
>             result = firstName.compareTo(otherFirst);  // 同姓按名字排
>         else
>             result = lastName.compareTo(otherLast);    // 不同姓按姓氏排
>
>         return result;
>     }
>
>     public String getFirstName() { return firstName; }
>     public String getLastName()  { return lastName;  }
> }
> ```
>
> ```java
> // Sorting.java —— 提供通用选择排序与插入排序的静态方法
> public class Sorting
> {
>     // 选择排序：参数为 Comparable 数组，适用于任何实现了 Comparable 的类型
>     public static void selectionSort(Comparable[] list)
>     {
>         int min;
>         Comparable temp;
>
>         for (int index = 0; index < list.length - 1; index++)
>         {
>             min = index;  // 假设当前位置为最小值
>
>             // 在未排序部分中找真正的最小值位置
>             for (int scan = index + 1; scan < list.length; scan++)
>                 if (list[scan].compareTo(list[min]) < 0)
>                     min = scan;
>
>             // 将最小值交换到当前位置
>             temp         = list[min];
>             list[min]    = list[index];
>             list[index]  = temp;
>         }
>     }
>
>     // 插入排序：将每个元素插入其左侧已排序部分的正确位置
>     public static void insertionSort(Comparable[] list)
>     {
>         for (int index = 1; index < list.length; index++)
>         {
>             Comparable key = list[index];  // 待插入的键
>             int position   = index;
>
>             // 将所有大于 key 的元素向右移动一位
>             while (position > 0 && key.compareTo(list[position - 1]) < 0)
>             {
>                 list[position] = list[position - 1];
>                 position--;
>             }
>
>             list[position] = key;  // 将键插入正确位置
>         }
>     }
> }
> ```
>
> ```java
> // PhoneList.java —— 驱动类，创建 Contact 数组，排序后打印
> public class PhoneList
> {
>     public static void main(String[] args)
>     {
>         Contact[] friends = new Contact[8];
>
>         friends[0] = new Contact("John",   "Smith",   "610-555-7384");
>         friends[1] = new Contact("Sarah",  "Barnes",  "215-555-3827");
>         friends[2] = new Contact("Mark",   "Riley",   "733-555-2969");
>         friends[3] = new Contact("Laura",  "Getz",    "663-555-3984");
>         friends[4] = new Contact("Larry",  "Smith",   "464-555-3489");
>         friends[5] = new Contact("Frank",  "Phelps",  "322-555-2284");
>         friends[6] = new Contact("Mario",  "Guzman",  "804-555-9066");
>         friends[7] = new Contact("Marsha", "Grant",   "243-555-2837");
>
>         Sorting.selectionSort(friends);  // Contact[] 传入 Comparable[] 参数
>
>         for (Contact friend : friends)
>             System.out.println(friend);
>     }
> }
> ```
>
> 输出：
> ```
> Barnes, Sarah   215-555-3827
> Getz, Laura     663-555-3984
> Grant, Marsha   243-555-2837
> Guzman, Mario   804-555-9066
> Phelps, Frank   322-555-2284
> Riley, Mark     733-555-2969
> Smith, John     610-555-7384
> Smith, Larry    464-555-3489
> ```
>
> ** `selectionSort` 的参数类型**
> ```java
> public static void selectionSort(Comparable[] list)
> ```
> - 参数类型为 `Comparable[]`，`Contact[]` 可以直接传入，因为 `Contact` 实现了 `Comparable`，`Contact[]` 是 `Comparable[]` 的子类型。
> - 方法体内只调用 `compareTo()`，不依赖任何具体类型，因此对 `String[]`、`Integer[]`、`Contact[]` 等均适用。
>
> ** `compareTo` 的多态调用**
> ```java
> if (list[scan].compareTo(list[min]) < 0)
>     min = scan;
> ```
> - `list[scan]` 的静态类型为 `Comparable`，编译器检查 `Comparable` 中是否声明了 `compareTo()` ——存在，编译通过。
> - 运行时，JVM 根据 `list[scan]` 实际指向的对象类型（此处为 `Contact`）调用 `Contact.compareTo()`。
> - 这正是多态性使排序算法与具体类型解耦的关键：排序逻辑不需要知道元素是什么类型，只需要它们实现了 `Comparable`。
>
> ** `Contact.compareTo` 的比较逻辑**
> ```java
> if (lastName.equals(otherLast))
>     result = firstName.compareTo(otherFirst);
> else
>     result = lastName.compareTo(otherLast);
> ```
> - 首先比较姓氏；若姓氏相同（如两个 `Smith`），再按名字排序。
> - `String.compareTo` 按词典序（Unicode 码点顺序）比较，大写字母排在小写字母之前。
> - 输出中 `Smith, John` 排在 `Smith, Larry` 之前，因为 `"John".compareTo("Larry") < 0`（`J` < `L`）。
>
> **插入排序的移位逻辑**
> ```java
> while (position > 0 && key.compareTo(list[position - 1]) < 0)
> {
>     list[position] = list[position - 1];  // 将较大元素右移
>     position--;
> }
> list[position] = key;
> ```
> - 外层循环每次取出 `list[index]` 作为键，内层循环将左侧所有大于键的元素依次右移一位，为键腾出位置。
> - 与选择排序相比，插入排序对接近有序的数组效率更高；选择排序每轮必须扫描整个未排序部分，插入排序在已排序部分有序时内层循环可提前终止。

> [!comparison]
> 选择排序与插入排序的时间复杂度均为 $O(n^2)$，适用于小规模数据。两者的主要区别在于：
>
> | | 选择排序 | 插入排序 |
> |---|---|---|
> | 核心操作 | 每轮找最小值并交换 | 每轮将元素插入已排序部分 |
> | 交换次数 | 恰好 $n-1$ 次 | 最多 $O(n^2)$ 次移位 |
> | 对有序数组 | 仍需 $O(n^2)$ 次比较 | 内层循环立即终止，$O(n)$ |
> | 稳定性 | 不稳定 | 稳定 |


---
`Pre: ` [[ELEC2543 Ch.14 Inheritance]]
`Post:` [[ELEC2543 Ch.16 Exceptions]]

#Y2S2 #ELEC2543
# Chapter 19. 数据结构

`Pre: ` [[ELEC2543 Ch.18 Sorting]]
`Post:` [[ELEC2543 Ch.20 Intro Trees]]

> [!abstract]
> 数据结构 (*Data Structure*) 指用于在内存中组织与管理一组数据的方式，决定了该组数据在程序中可以做的操作及其代价；抽象数据类型 (*Abstract Data Type, ADT*) 进一步把"数据的组织"与"管理操作的接口"封装在一起，对外仅暴露操作契约，对内由具体数据结构实现。同一 ADT 往往可由多种不同的数据结构承载，这种"逻辑使用方式"与"物理实现细节"相分离的观念，也是集合 (*Collection*) 类抽象的共同出发点。
>
> Java 把 ADT 的表达分为两层：语言层面以 `List`、`Set`、`Map`、`Queue` 等接口承载操作集合，以 `ArrayList`、`LinkedList`、`Stack` 等类承载具体实现；底层链接则依赖对象引用 (*Reference*) 把一组节点串成可动态增删的链式结构，以此突破数组的固定长度限制。对于小规模自定义数据结构，课程示例仍以手写的节点类为底层载体，便于观察引用之间的连接关系；工程实践中则更多直接使用 Java 集合 API (*Java Collections API*) 提供的成熟类。
>
> 本章先说明集合与 ADT 的概念，再以引用为链条展开动态数据结构 (*Dynamic Data Structure*)、单链表与双向链表的增删操作；随后给出线性 ADT 的两种典型——队列 (*Queue*) 与栈 (*Stack*)，分别讨论它们的 FIFO 与 LIFO 语义、基于链表与数组的两种实现，以及 `java.util.Stack` 的使用示例；最后引入非线性 ADT 的树 (*Tree*) 与图 (*Graph*)，并以 Java 集合 API 与泛型 (*Generic Type*) 收束。

```table-of-contents
maxLevel: 3
```


## 19.1 集合与抽象数据类型

集合 (*Collection*) 是用于组织、管理一组对象的对象，把一组元素聚合在一起，并为读取、添加、删除等常见操作提供统一入口。本章讨论的链表、队列、栈、树、图都是不同形态的集合，差别在于各自对"元素之间的结构关系"与"可执行操作集"的约定不同。

抽象数据类型 (*Abstract Data Type, ADT*) 把这种"一组数据 + 一组管理操作"组合成一个整体单元，并通过公开接口与私有实现的划分，使外部代码仅依赖接口约定、不感知具体实现。Java 通过"接口 + 实现类"的组合承载 ADT：`List`、`Stack`、`Queue`、`Set`、`Map` 等接口描述操作集合，`ArrayList`、`LinkedList`、`Stack` 等类以具体数据结构实现这些接口。

```
+-------------+         +-------------------------------+
| Application |  接口   |  Abstract Data Type            |
|   Program   |◄──────► |   Public Functions             |
+-------------+         |   Private Functions            |
                        |   Data Structures              |
                        |   (Array / Linked List / ...)  |
                        +-------------------------------+
```

Java 标准库中常见的数据结构包括 `Array`、`Linked List`、`Stack`、`Queue`、`Binary Tree`、`Binary Search Tree`、`Heap`、`Hashing` 与 `Graph`；本章集中讨论链表、队列、栈，以及树与图的基本形式。


## 19.2 动态结构与链表

静态数据结构 (*Static Data Structure*) 指大小在定义时即固定、运行期间不再改变的结构；Java 中数组一旦经 `new` 分配长度，便无法再扩容。动态数据结构 (*Dynamic Data Structure*) 则在运行期间根据内容按需增长或收缩，其底层实现依赖对象引用——一个引用变量保存另一对象的地址，充当指针 (*Pointer*) 的角色。

> [!note]
> 这里的 "static" 强调结构大小固定，与 `static` 修饰符的语义无关；`static` 修饰符标注类级别的字段或方法，与数据结构的大小无关。

当一个类的字段本身是指向同类对象的引用时，就可用该字段把多个同类对象串成链式结构，节点之间通过引用相互连接。以最基本的节点类为例：

```java
class Node {
    int info;
    Node next;
}
```

反复沿用 `next` 引用就能构成单链表 (*Singly Linked List*)：链表以一个头引用 (*Head Reference*) 标识首节点，每个节点保存一个信息字段与一个指向下一节点的 `next`，末节点的 `next` 为 `null`。

```
list ──► [info|next] ──► [info|next] ──► [info|next] ──► null
```

#### 中间节点与对象独立性

直接把 `next` 字段写在业务对象（如 `Student`）内，会使业务类与具体数据结构耦合；被存储的对象不应感知自身将被放入何种数据结构。惯用做法是引入中间节点 (*Intermediate Node*)：独立的节点类封装一个指向业务对象的引用与一个指向下一节点的 `next`，由链表内部维护，业务类保持不变。

> [!example]- 示例：`MagazineRack.java` / `MagazineList.java` / `Magazine.java`
>
> `MagazineRack.java` 作为驱动程序创建 `MagazineList` 对象并向其中加入若干 `Magazine`，随后整体打印；`MagazineList.java` 以私有内部类 `MagazineNode` 包裹 `Magazine` 引用与 `next` 链接，自行维护链表头 `list`；`Magazine.java` 仅持有单个标题字符串，不感知任何链表结构。
>
> > [!info]- UML 框图
> >
> > ```
> > +----------------+        +----------------------+        +------------+
> > | MagazineRack   | -----> |  MagazineList        | -----> | Magazine   |
> > | main 驱动程序  |        |  - list : Node       |        | - title    |
> > +----------------+        |  + add / toString    |        | + toString |
> >                           +----------------------+        +------------+
> >                                       △ inner
> >                           +----------------------+
> >                           |  MagazineNode        |
> >                           |  magazine, next      |
> >                           +----------------------+
> > ```
>
> ```java
> public class MagazineRack {
>     public static void main(String[] args) {
>         MagazineList rack = new MagazineList();
>
>         rack.add(new Magazine("Time"));
>         rack.add(new Magazine("Woodworking Today"));
>         rack.add(new Magazine("Communications of the ACM"));
>         rack.add(new Magazine("House and Garden"));
>         rack.add(new Magazine("GQ"));
>
>         System.out.println(rack);
>     }
> }
>
> public class MagazineList {
>     private MagazineNode list;
>
>     public MagazineList() {
>         list = null;
>     }
>
>     // 把新节点追加到链表末尾
>     public void add(Magazine mag) {
>         MagazineNode node = new MagazineNode(mag);
>         MagazineNode current;
>
>         if (list == null) {            // 空链表：新节点即为表头
>             list = node;
>         } else {                       // 非空链表：遍历到末节点再追加
>             current = list;
>             while (current.next != null) {
>                 current = current.next;
>             }
>             current.next = node;
>         }
>     }
>
>     // 从表头开始遍历，将各节点的标题按行拼成字符串
>     public String toString() {
>         String result = "";
>         MagazineNode current = list;
>         while (current != null) {
>             result += current.magazine + "\n";
>             current = current.next;
>         }
>         return result;
>     }
>
>     // 内部节点类：包裹一个 Magazine 与一个 next 引用
>     private class MagazineNode {
>         public Magazine magazine;
>         public MagazineNode next;
>
>         public MagazineNode(Magazine mag) {
>             magazine = mag;
>             next = null;
>         }
>     }
> }
>
> public class Magazine {
>     private String title;
>
>     public Magazine(String newTitle) {
>         title = newTitle;
>     }
>
>     public String toString() {
>         return title;
>     }
> }
> ```
>
> 输出：
>
> ```
> Time
> Woodworking Today
> Communications of the ACM
> House and Garden
> GQ
> ```
>
> **业务类与链表完全解耦**
>
> ```java
> public class Magazine {
>     private String title;
>     ...
> }
> ```
>
> - `Magazine` 只保存标题字段与相应的构造器、`toString`，不包含任何 `next`、`prev` 字段或链表相关操作。
> - 若后续要把同一批 `Magazine` 改放入另一种数据结构（如数组、`ArrayList` 或其他链表），`Magazine` 类无需修改。
>
> **内部节点类 `MagazineNode` 封装链接职责**
>
> ```java
> private class MagazineNode {
>     public Magazine magazine;
>     public MagazineNode next;
>     ...
> }
> ```
>
> - `MagazineNode` 是 `MagazineList` 的私有内部类；字段声明为 `public` 仅方便同一外部类内部使用，私有内部类使外部代码无法直接访问节点结构。
> - 每个节点持有一个 `Magazine` 引用与一个 `MagazineNode next`，把"业务对象"与"链接关系"分离开来。
>
> **`add` 方法：按链表头状态分两支追加**
>
> ```java
> if (list == null) {
>     list = node;
> } else {
>     current = list;
>     while (current.next != null) {
>         current = current.next;
>     }
>     current.next = node;
> }
> ```
>
> - 链表空时，新节点直接成为 `list` 所指的表头，不需移动任何指针。
> - 链表非空时，`current` 从表头逐步沿 `next` 走到 `current.next == null` 的末节点，再把末节点的 `next` 指向新节点；循环条件用 `current.next` 而非 `current`，保证停留在末节点而非越出。
>
> **`toString` 方法：从表头顺序遍历**
>
> ```java
> MagazineNode current = list;
> while (current != null) {
>     result += current.magazine + "\n";
>     current = current.next;
> }
> ```
>
> - `current` 从 `list` 起沿 `next` 逐节点推进；循环在 `current == null` 时终止，越过末节点后不再访问。
> - `current.magazine` 通过隐式 `toString` 转字符串拼接，`MagazineList` 本身不直接读取 `Magazine` 的私有字段。


#### 节点的插入与删除

单链表的插入与删除不移动其他节点，只需改动相邻节点的 `next` 引用。

```
插入 newNode 到 current 之后：

  ... ──► [current] ──┈┈► [ .. ] ──► ...       插入前
                ╲
                 ╲─► [newNode]
                          ╲
                           ╲─► [ .. ] ──► ...  插入后
```

> [!question] 习题：Quick Check (1)
> 在单链表中，`current` 指向某个已有节点，`newNode` 为已创建的新节点。写出将 `newNode` 插入到 `current` 之后的代码。
>
> > [!check]-
> > 先让 `newNode.next` 指向 `current` 原先的后继，再让 `current.next` 指向 `newNode`，整体只需两条语句。两条语句的顺序不能颠倒：若先改 `current.next`，原后继引用将丢失。
> >
> > ```java
> > newNode.next = current.next;
> > current.next = newNode;
> > ```

> [!question] 习题：Quick Check (2)
> 在单链表中，写出删除 `current` 节点之后那一节点的代码；若 `current` 已是末节点，则保持链表不变。
>
> > [!check]-
> > 以 `current.next != null` 判断 `current` 之后是否存在可删节点；若存在，将 `current.next` 跨过其后继直接指向后继的后继即可。原被删节点不再被任何引用持有，由垃圾回收器回收。
> >
> > ```java
> > if (current.next != null) {
> >     current.next = current.next.next;
> > }
> > ```


#### 双向链表

单链表的节点只保存指向后继的 `next`，无法在 $O(1)$ 时间内访问前驱；双向链表 (*Doubly Linked List*) 在节点中额外保存一个 `prev` 引用，使两个方向的遍历与修改都成为可能。

```
list ──► [info|next|prev] ◄──► [info|next|prev] ◄──► [info|next|prev] ──► null
```

> [!question] 习题：Quick Check (3)
> 在双向链表中，写出删除 `current` 节点之后那一节点的代码。
>
> > [!check]-
> > 先按单链表的写法断开 `next` 链；此时若 `current.next` 不为空（即删除后仍存在新的后继），再把该后继的 `prev` 回指 `current`，使双向链保持一致。
> >
> > ```java
> > if (current.next != null) {
> >     current.next = current.next.next;
> >     if (current.next != null) {
> >         current.next.prev = current;
> >     }
> > }
> > ```


#### Java 的 `LinkedList` 类

`java.util.LinkedList<E>` 是 Java 集合 API 中以双向链表实现的通用列表类，隐藏了节点结构，对外提供一组统一方法：`add` 与 `addLast` 在尾部追加，`addFirst` 在头部插入，`add(int, E)` 按指定下标插入；`remove(Object)` 按值删除首个匹配项，`remove(int)` 按下标删除，`removeFirst` 与 `removeLast` 作用于两端；`set(int, E)` 按下标覆写已有位置，`get(int)` 按下标读取。

> [!example]- 示例：`Test.java`
>
> `Test.java` 创建一个 `LinkedList<String>`，依次演示在不同位置插入、按值与按下标删除，并通过两次 `println` 打印插入与删除后的完整列表。
>
> ```java
> import java.util.*;
>
> public class Test {
>     public static void main(String[] args) {
>         LinkedList<String> ll = new LinkedList<String>();
>
>         // 插入：默认尾部、显式尾部、显式头部、按下标插入
>         ll.add("A");
>         ll.add("B");
>         ll.addLast("C");
>         ll.addFirst("D");
>         ll.add(2, "E");
>
>         System.out.println(ll);
>
>         // 删除：按值、按下标、按两端
>         ll.remove("B");
>         ll.remove(3);
>         ll.removeFirst();
>         ll.removeLast();
>
>         System.out.println(ll);
>     }
> }
> ```
>
> 输出：
>
> ```
> [D, A, E, B, C]
> [A]
> ```
>
> **追加与按下标插入的语义区分**
>
> ```java
> ll.add("A");        // → [A]
> ll.add("B");        // → [A, B]
> ll.addLast("C");    // → [A, B, C]
> ll.addFirst("D");   // → [D, A, B, C]
> ll.add(2, "E");     // → [D, A, E, B, C]
> ```
>
> - `add(E)` 与 `addLast(E)` 均追加到末尾；`addFirst(E)` 插入到开头，整体下标后移一位。
> - `add(int, E)` 将新元素放入指定下标处，原有元素整体向后挪一位；下标 `2` 插入 `"E"` 后，原 `"B"` 被推到下标 3。
>
> **按值删除与按下标删除的歧义边界**
>
> ```java
> ll.remove("B");     // → [D, A, E, C]
> ll.remove(3);       // → [D, A, E]
> ll.removeFirst();   // → [A, E]
> ll.removeLast();    // → [A]
> ```
>
> - `remove(Object)` 按值删除首次出现，不按下标；`remove(int)` 按下标删除指定位置的元素。
> - 当实参既可解释为对象也可解释为下标时（如 `Integer` 与 `int`），编译器按静态类型选择重载；传字面量 `3` 时调用按下标版本。


## 19.3 线性结构：队列与栈

队列 (*Queue*) 与栈 (*Stack*) 是两种最基本的线性 ADT。它们对读写位置加以限制，由此形成与普通列表不同的行为语义。

### 队列

队列是一种只从末端 (*Rear*) 加入元素、只从前端 (*Front*) 取出元素的线性结构，遵循先入先出 (*First-In, First-Out, FIFO*) 的访问顺序，常用于模拟排队等待处理的场景。队列的三种核心操作为 `enqueue`（末端入队）、`dequeue`（前端出队）与 `empty`（判空）。

```
   入队 (enqueue)                                 出队 (dequeue)
        │                                              ▲
        ▼                                              │
      ┌───┬───┬───┬───┬───┬───┬───┐
      │   │   │   │   │   │   │   │
      └───┴───┴───┴───┴───┴───┴───┘
        rear ◄──────────────────► front
```

底层实现上，队列可由单链表承载——链表指针从前端指向末端，入队直接改写尾节点、出队直接摘除头节点；也可由数组承载，并用取余运算 `%` 在数组尾部与头部之间"绕回"，形成循环队列。

> [!question] 习题：Exercise（队列）
> 初始为空队列，依次执行以下操作后，写出队列内容（自队首至队尾）：
>
> ```
> enqueue(45);
> enqueue(12);
> enqueue(28);
> dequeue();
> dequeue();
> enqueue(69);
> enqueue(27);
> ```
>
> > [!check]-
> > 三次 `enqueue` 后队列为 `45, 12, 28`；两次 `dequeue` 依 FIFO 顺序从队首先后移出 `45` 与 `12`，剩下 `28`；其后两次 `enqueue` 在队尾追加 `69` 与 `27`。最终从队首到队尾为 `28, 69, 27`。

> [!example]- 示例：`MyQueue.java`
>
> `MyQueue.java` 给出队列的两个等价实现：v1 以手写链表承载，由 `front`、`rear` 分别指向队首与队尾，入队把新节点接到 `rear` 之后再更新 `rear`；v2 直接复用 `ArrayList`，入队对应 `add`，出队对应 `remove(0)`，判空委托 `isEmpty`。两种实现对外提供完全相同的方法集。
>
> ```java
> // v1：链表实现
> public class MyQueue {
>     private MyNode front, rear;
>
>     public MyQueue() {
>         front = rear = null;
>     }
>
>     public boolean empty() {
>         return (front == null);
>     }
>
>     public void enqueue(Object item) {
>         MyNode newNode = new MyNode(item);
>         if (empty()) {
>             front = rear = newNode;   // 空队列：首尾指针同时指向新节点
>             return;
>         } else {
>             rear.next = newNode;      // 非空队列：先挂到旧尾之后
>         }
>         rear = newNode;               // 再更新尾指针
>     }
>
>     public Object dequeue() {
>         if (empty()) return null;
>         Object obj = front.obj;
>         if (front == rear) {
>             front = rear = null;      // 唯一节点：两端指针一同置空
>         } else {
>             front = front.next;       // 一般情形：首指针前移
>         }
>         return obj;
>     }
> }
>
> // v2：ArrayList 实现
> public class MyQueue {
>     private ArrayList<Object> queue;
>
>     public MyQueue() {
>         queue = new ArrayList<Object>();
>     }
>
>     public boolean empty() {
>         return queue.isEmpty();
>     }
>
>     public void enqueue(Object item) {
>         queue.add(item);
>     }
>
>     public Object dequeue() {
>         if (queue.isEmpty()) return null;
>         return queue.remove(0);
>     }
> }
> ```
>
> **链表实现：双指针维护首尾**
>
> ```java
> public void enqueue(Object item) {
>     MyNode newNode = new MyNode(item);
>     if (empty()) {
>         front = rear = newNode;
>         return;
>     } else {
>         rear.next = newNode;
>     }
>     rear = newNode;
> }
> ```
>
> - 链表实现需同时维护 `front` 与 `rear`：`enqueue` 通过 `rear` 在 $O(1)$ 时间内追加末节点，`dequeue` 通过 `front` 在 $O(1)$ 时间内摘除首节点；若只保留 `front`，每次 `enqueue` 都需遍历整条链。
> - 空队列与单元素队列为两个边界情形：`enqueue` 空队列时需同时给 `front` 与 `rear` 赋值；`dequeue` 最后一个元素时需把 `rear` 也清为 `null`，否则 `rear` 将成为悬垂引用。
>
> **ArrayList 实现：委托给库类**
>
> ```java
> public void enqueue(Object item) { queue.add(item); }
> public Object dequeue() {
>     if (queue.isEmpty()) return null;
>     return queue.remove(0);
> }
> ```
>
> - `ArrayList` 版本代码量远少于链表版本，但 `remove(0)` 会引发剩余元素整体前移，单次 `dequeue` 在最坏情形下为 $O(n)$；而链表版本的两个操作均为 $O(1)$。
> - 两种实现提供完全相同的公开接口，外部调用方无需感知底层差异；这正是 ADT 所强调的"接口与实现分离"。


### 栈

栈是一种只在同一端读写的线性结构，遵循后入先出 (*Last-In, First-Out, LIFO*)——后放入的元素先被取出，如同一摞盘子或书本。栈的核心操作为 `push`（压栈）、`pop`（弹栈）、`peek`（或 `top`，读取栈顶但不弹出）与 `empty`（判空）。

```
     push ──►  ┌───┐  ──► pop
               │   │
               ├───┤
               │   │        ◄── 栈顶
               ├───┤
               │   │
               ├───┤
               │   │        ◄── 栈底
               └───┘
```

栈同样可由单链表或数组承载：链表实现以表头为栈顶，`push` 与 `pop` 均 $O(1)$；数组实现以下标 `0` 为栈底，栈顶下标随元素增减而变动。

> [!question] 习题：Exercise（栈）
> 初始为空栈，依次执行以下操作后，写出栈内容（自栈底至栈顶）：
>
> ```
> push(45);
> push(12);
> push(28);
> pop();
> pop();
> push(69);
> push(27);
> push(99);
> ```
>
> > [!check]-
> > 三次 `push` 后栈自底至顶为 `45, 12, 28`；两次 `pop` 按 LIFO 依次弹出 `28` 与 `12`，只剩 `45`；其后三次 `push` 依次在栈顶增添 `69`、`27`、`99`。最终自栈底至栈顶为 `45, 69, 27, 99`。

`java.util` 包提供 `Stack` 类，其操作作用于 `Object` 引用；对应方法 `push`、`pop`、`peek`、`empty` 与上文语义一致。

> [!example]- 示例：`Decode.java`
>
> `Decode.java` 使用 `java.util.Stack` 还原一段"每词字母被反转"的编码消息：按空格划分单词，每个字符依次压栈；空格出现时将整栈弹空，由于 LIFO 顺序使字符逆序吐出，正好抵消编码时的反转。
>
> ```java
> import java.util.*;
>
> public class Decode {
>     public static void main(String[] args) {
>         Scanner scan = new Scanner(System.in);
>         Stack word = new Stack();
>         String message;
>         int index = 0;
>
>         System.out.println("Enter the coded message:");
>         message = scan.nextLine();
>         System.out.println("The decoded message is:");
>
>         while (index < message.length()) {
>             // 将当前单词的字符依次压栈
>             while (index < message.length() && message.charAt(index) != ' ') {
>                 word.push(new Character(message.charAt(index)));
>                 index++;
>             }
>             // 遇到空格或串末：弹空栈以逆序打印本单词
>             while (!word.empty()) {
>                 System.out.print(((Character) word.pop()).charValue());
>             }
>             System.out.print(" ");
>             index++;
>         }
>         System.out.println();
>     }
> }
> ```
>
> 输出（输入 `artxE eseehc esaelp`）：
>
> ```
> Enter the coded message:
> artxE eseehc esaelp
> The decoded message is:
> Extra cheese please
> ```
>
> **外层循环按单词推进**
>
> ```java
> while (index < message.length()) { ... index++; }
> ```
>
> - 外层 `while` 以 `index` 走过整条输入串；内部两层循环共同处理一个单词，单词结尾或串末处 `index++` 越过空格。
>
> **压栈阶段：读入本单词全部字符**
>
> ```java
> while (index < message.length() && message.charAt(index) != ' ') {
>     word.push(new Character(message.charAt(index)));
>     index++;
> }
> ```
>
> - 循环在遇到空格或串末时终止；`message.charAt(index)` 为基本类型 `char`，由 `new Character(...)` 包装为 `Stack` 所接收的对象引用。
>
> **弹栈阶段：反转输出**
>
> ```java
> while (!word.empty()) {
>     System.out.print(((Character) word.pop()).charValue());
> }
> ```
>
> - `Stack.pop` 返回 `Object`，需显式向下类型转换为 `Character` 再取 `charValue()`；由于 LIFO，字符按入栈的相反顺序输出，正好还原原单词。


## 19.4 非线性结构：树与图

队列、栈与链表都属于线性结构：元素之间的"前-后"关系是唯一且单向的。非线性结构 (*Non-linear Structure*) 则允许每个元素关联多个其他元素，由此派生出两类经典结构——树与图。

### 树

树 (*Tree*) 是一种层级化的非线性结构，由一个根节点 (*Root Node*) 与若干子节点层层构成。没有子节点的节点称为叶节点 (*Leaf Node*)；既非根也非叶的节点称为内部节点 (*Internal Node*)。一般树允许任何节点拥有任意多个子节点。

```
                ●                ◄── 根节点
              / | \
             ●  ●  ●
            /|     |
           ● ●     ●
                   |
                   ●             ◄── 叶节点
```

二叉树 (*Binary Tree*) 在一般树的基础上进一步约束：每个节点至多两个子节点。对二叉树而言，每个节点只需持有对左、右子节点的两条引用即可完整描述其层级结构，通常也以动态链表式节点承载——节点类中除业务字段外，只保留 `left` 与 `right` 两个引用。

### 图

图 (*Graph*) 是另一类非线性结构，不设定根节点；图中任意两节点都可通过边 (*Edge*) 相连，不存在固定的层级关系。

```
    ●───●         ●
         ╲         ╱ │
    ●───●─●───● ●
     ╲  │    │
      ●─●────●
        │
        ●
```

在有向图 (*Directed Graph, Digraph*) 中，每条边具有方向，这种带方向的边也称为弧 (*Arc*)，常用于刻画"从 A 到 B 单向可达、但未必反向可达"的关系，如航班的起讫站。

图与有向图均可采用两类底层表示：以动态链表把相邻节点串起，或以二维数组（邻接矩阵）记录节点对之间是否存在边。选用哪种表示取决于所关注的操作——频繁查询某对节点是否相连时邻接矩阵更直接，频繁遍历某节点的邻居时链表更省空间。


## 19.5 Java 集合 API 与泛型

Java 标准库提供一组用于表达集合的类，统称 Java 集合 API (*Java Collections API*)。其底层实现策略常直接体现在类名中：`ArrayList` 以数组承载、`LinkedList` 以链表承载；对应的 `List`、`Set`、`SortedSet`、`Map`、`SortedMap` 等接口则从"可执行的操作"角度描述集合，与具体实现解耦。

泛型 (*Generic Type*) 是 Java 支持的类型参数机制，在集合定义中尤为关键。集合类被定义为可操作通用数据类型，具体的元素类型在实例化时以类型参数指定：

```java
LinkedList<元素类型> <变量名> = new LinkedList<元素类型>();

LinkedList<Book> myList = new LinkedList<Book>();
```

一旦指定类型参数，集合只接受该类型（或其子类型）的元素；取出元素时类型已由编译器确定，免去显式向下类型转换。这将集合从"只能装 `Object`"的早期写法改造为在编译期即可检查类型一致性的数据结构，显著减少运行时类型错误。


---
`Pre: ` [[ELEC2543 Ch.18 Sorting]]
`Post:` [[ELEC2543 Ch.20 Intro Trees]]

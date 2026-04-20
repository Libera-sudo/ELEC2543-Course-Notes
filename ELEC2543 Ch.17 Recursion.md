#Y2S2 #ELEC2543
# Chapter 17. 递归

`Pre: ` [[ELEC2543 Ch.16 Exceptions]]
`Post:` [[ELEC2543 Ch.18 Sorting]]

> [!abstract]
> 递归 (*Recursion*) 是将一个问题表述为同类、规模更小的子问题之组合的程序设计技术，并以显式的基础情形 (*Base Case*) 终止求解过程。对于本身具有自相似结构的问题，如列表、树、分治与搜索，递归往往比迭代 (*Iteration*) 更贴近问题定义本身，也更易于表达与验证。
>
> Java 不为递归设立专门语法，递归方法只是沿调用链直接或间接调用自身的一般方法；每一次调用按普通方法调用规则建立新的执行环境，拥有独立的形参与局部变量，并在完成时将控制权交还给上一次调用。递归的正确性完全依赖基础情形的可达性，而非任何特殊语法；递归的可行性则受限于 JVM 方法调用栈的容量，这与以尾调用优化作为语言保证的函数式语言形成对照。
>
> 本章涵盖递归思维与递归定义、基础情形与无限递归、以阶乘与求和为代表的递归展开过程、递归方法的结构与执行模型、递归与迭代的权衡、直接递归与间接递归的区分，以及递归在迷宫遍历与汉诺塔问题上的经典应用。

```table-of-contents
maxLevel: 3
```

## 17.1 递归思维与递归定义

递归定义 (*Recursive Definition*) 指在定义自身时再次使用所定义的概念本身。在一般语言解释中，这种自指定义往往无助于说明；但在具有自相似结构的问题上，递归反而是最贴近事物本身的表达方式。

以数字序列为例，可将一个 `List` 递归地定义为：

```
List:  number
     | number  ,  List
```

即 `List` 要么是一个 number，要么是一个 number 后接逗号再接一个 `List`。以 `24, 88, 40, 37` 为例，递归地展开为：`24` 是 number，`88, 40, 37` 仍是 List；其中 `88` 是 number，`40, 37` 仍是 List；该 List 再分解为 `40` 与 `37`；`37` 单独构成一个 number，递归终止。

```
LIST:  number  comma  LIST
        24      ,     88, 40, 37
                      number  comma  LIST
                        88      ,    40, 37
                                     number  comma  LIST
                                       40      ,    37
                                                    number
                                                      37
```

非递归的分支称为基础情形 (*Base Case*)。所有递归定义都必须至少包含一个基础情形，否则递归分支会无止境地指向自身，形成无限递归 (*Infinite Recursion*)。无限递归在形态上类似于死循环，但"不终止"的部分来自定义本身而非某条不退出的语句。

> [!note]
> 递归定义的正确性由递归情形与基础情形共同保证：仅有递归情形会导致无限展开，仅有基础情形则退化为非递归定义；只有当递归情形每次都将问题规模推向基础情形时，整个定义才能在有限步内收敛。


## 17.2 递归阶乘

阶乘 (*Factorial*) 是递归定义的典型例子。对任意正整数 $N$，$N!$ 定义为 $1$ 到 $N$ 之间所有整数的乘积。该定义可改写为递归形式：

$$
1! = 1, \qquad N! = N \times (N-1)!
$$

其中 $1! = 1$ 为基础情形，$N! = N \times (N-1)!$ 为递归情形；后者将 $N!$ 的求解交还给规模更小的同类问题 $(N-1)!$，最终在 $N = 1$ 处收敛。

以 $5!$ 为例，递归先自上而下逐层将问题移交给更小的子问题，待基础情形返回后再自下而上逐层回填：

```
  5!
= 5 * 4!
=     4 * 3!
=         3 * 2!
=             2 * 1!
=                 1          (base case)
=             2 * 1 = 2
=         3 * 2     = 6
=     4 * 6         = 24
= 5 * 24            = 120
```

每一次 $(N-1)!$ 的调用都暂存当前层的乘法；直到基础情形返回具体数值，各层的乘法才按自底向上的顺序依次完成。

> [!question] 习题：`5 * n` 的递归定义
> 将 `5 * n` 表达为递归定义，其中 $n > 0$。
>
> > [!check]-
> > 以 $n = 1$ 作为基础情形、以 $n > 1$ 作为递归情形：
> >
> > $$
> > 5 \times 1 = 5, \qquad 5 \times n = 5 + 5 \times (n - 1)
> > $$
> >
> > 思路与阶乘一致：将乘法拆解为加法的重复叠加，每一层在规模更小的子问题基础上累加一次 $5$，最终在 $n = 1$ 处取得基础值 $5$。


## 17.3 递归方法的结构与执行

递归方法 (*Recursive Method*) 是调用自身的方法，是递归思维在程序中的直接实现。一个可正确终止的递归方法必须同时处理两类情形：基础情形直接返回结果而不再调用自身，递归情形通过对规模更小的子问题发起新调用获得部分结果。

每一次方法调用都按普通方法调用规则建立独立的执行环境，包含自己的形参与局部变量；当该次调用完成时，控制权交还给调用方，而调用方可能是上一层递归调用，也可能是最初的非递归调用。递归并不需要专门的语言机制，只是常规方法调用在"调用者的调用链最终回到自身"时的自然形态。

> [!example]- 示例：`sum` 方法
>
> `sum` 方法以递归形式计算 $1$ 到 `num` 的求和 $\sum_{i=1}^{\text{num}} i$，用以观察递归方法基础情形与递归情形的分工，以及递归调用在运行期间逐层展开、再逐层回填的过程。
>
> ```java
> // 递归求 1 + 2 + ... + num
> public int sum(int num) {
>     int result;
>
>     if (num == 1)
>         result = 1;                    // base case
>     else
>         result = num + sum(num - 1);   // recursive case
>
>     return result;
> }
> ```
>
> 以 `sum(4)` 为例，调用关系自上而下展开、自下而上回填：
>
> ```
> main  ──→ sum(4)   num=4, result = 4 + sum(3)
>              │
>              ├──→ sum(3)   num=3, result = 3 + sum(2)
>              │       │
>              │       ├──→ sum(2)   num=2, result = 2 + sum(1)
>              │       │       │
>              │       │       ├──→ sum(1)   num=1, result = 1
>              │       │       │←── return 1
>              │       │←── return 3
>              │←── return 6
>        ←── return 10
> ```
>
> **基础情形与递归情形的二选一**
>
> ```java
> if (num == 1)
>     result = 1;
> else
>     result = num + sum(num - 1);
> ```
>
> - `num == 1` 时直接以具体值赋 `result`，不再调用自身，递归终止。
> - `num > 1` 时将原问题拆为"当前 `num`"与"`sum(num-1)`"之和，把规模减一的子问题交给下一层递归。
>
> **每一层调用的独立执行环境**
>
> ```java
> public int sum(int num) {
>     int result;
>     ...
> }
> ```
>
> - `num` 与 `result` 是每次调用独立的局部变量，`sum(4)` 的 `num` 与 `sum(3)` 的 `num` 互不影响。
> - 调用栈同时保存尚未返回的多层 `sum`，每层各自持有自己的 `num` 与未完成的加法，待下层返回后才能完成本层计算。

递归与迭代在表达力上是等价的：任何可以用迭代表达的问题也都可以用递归表达，反之亦然。对 $1$ 到 $N$ 的求和这类线性累积问题，迭代版本通常更直接，不应为了使用递归而使用递归；但对自相似结构天然存在的问题，递归常常能给出更贴近问题定义、也更易于理解的实现。**是否使用递归应由问题结构决定，而非由技术可行性决定**。


## 17.4 直接递归与间接递归

按调用链的形态，递归可分为两类：

- 直接递归 (*Direct Recursion*) 指方法直接调用自身，如 [[ELEC2543 Ch.17 Recursion#17.3 递归方法的结构与执行|17.3]] 中的 `sum`。
- 间接递归 (*Indirect Recursion*) 指方法经过若干中间方法后再次回到自身，例如 `m1` 调用 `m2`，`m2` 调用 `m3`，`m3` 又调用 `m1`。

```
m1 ──→ m2 ──→ m3 ──→ m1 ──→ m2 ──→ m3 ──→ m1 ──→ ...
```

间接递归与直接递归在正确性要求上完全一致，同样需要某个基础情形以终止调用链；区别仅在于递归路径分布在多个方法之间。也正因如此，间接递归在阅读与调试时更难追踪：单独查看任何一个方法都看不到递归关系，只有把整个调用链铺开才能发现循环。


## 17.5 迷宫遍历

迷宫遍历 (*Maze Traversal*) 是递归的一个经典应用：在以 $0$ 表示阻塞、$1$ 表示可通行的二维网格中，寻找一条从起点到终点的路径。从任一位置出发，向上下左右四个方向各发起一次递归探索；递归的基础情形包括两类——"越界或不可通行"使当前分支立即终止，"到达终点"使整条路径确认为解。

> [!example]- 示例：`MazeSearch.java` / `Maze.java`
>
> 两文件共同演示递归在迷宫搜索中的实现：`Maze` 保存迷宫网格并提供 `traverse` 方法沿上下左右递归探路；`MazeSearch` 的 `main` 方法创建迷宫、触发 `traverse(0, 0)` 从左上角开始搜索，并在搜索前后打印网格。网格使用 $0$ 表示阻塞、$1$ 表示待尝试、$3$ (`TRIED`) 表示走过但不在最终路径上、$7$ (`PATH`) 表示属于最终路径。
>
> > [!info]- UML 框图
> >
> > ```
> > ┌──────────────────────────────┐
> > │ MazeSearch                   │
> > │  + main(String[]): void      │
> > └──────────────┬───────────────┘
> >                │ creates & uses
> >                ▼
> > ┌──────────────────────────────┐
> > │ Maze                         │
> > │  - TRIED: int                │
> > │  - PATH: int                 │
> > │  - grid: int[][]             │
> > │  + traverse(int, int): bool  │
> > │  - valid(int, int): bool     │
> > │  + toString(): String        │
> > └──────────────────────────────┘
> > ```
>
> ```java
> // MazeSearch.java —— 创建迷宫并发起递归搜索
> public class MazeSearch {
>     public static void main(String[] args) {
>         Maze labyrinth = new Maze();
>
>         System.out.println(labyrinth);
>
>         if (labyrinth.traverse(0, 0))
>             System.out.println("The maze was successfully traversed!");
>         else
>             System.out.println("There is no possible path.");
>
>         System.out.println(labyrinth);
>     }
> }
> ```
>
> ```java
> // Maze.java —— 以递归 traverse 搜索可行路径
> public class Maze {
>     private final int TRIED = 3;
>     private final int PATH  = 7;
>
>     private int[][] grid = { {1,1,1,0,1,1,0,0,0,1,1,1,1},
>                              {1,0,1,1,1,0,1,1,1,1,0,0,1},
>                              {0,0,0,0,1,0,1,0,1,0,1,0,0},
>                              {1,1,1,0,1,1,1,0,1,0,1,1,1},
>                              {1,0,1,0,0,0,0,1,1,1,0,0,1},
>                              {1,0,1,1,1,1,1,1,0,1,1,1,1},
>                              {1,0,0,0,0,0,0,0,0,0,0,0,0},
>                              {1,1,1,1,1,1,1,1,1,1,1,1,1} };
>
>     // 递归尝试从 (row, column) 出发到达右下角
>     public boolean traverse(int row, int column) {
>         boolean done = false;
>
>         if (valid(row, column)) {
>             grid[row][column] = TRIED;                         // 标记本格已尝试
>
>             if (row == grid.length - 1 &&
>                 column == grid[0].length - 1)
>                 done = true;                                   // 到达终点
>             else {
>                 done = traverse(row + 1, column);              // down
>                 if (!done) done = traverse(row, column + 1);   // right
>                 if (!done) done = traverse(row - 1, column);   // up
>                 if (!done) done = traverse(row, column - 1);   // left
>             }
>
>             if (done)
>                 grid[row][column] = PATH;                      // 属于最终路径
>         }
>
>         return done;
>     }
>
>     // 当前格是否在界内且仍可尝试
>     private boolean valid(int row, int column) {
>         boolean result = false;
>
>         if (row >= 0 && row < grid.length &&
>             column >= 0 && column < grid[row].length)
>             if (grid[row][column] == 1)
>                 result = true;
>
>         return result;
>     }
>
>     public String toString() {
>         String result = "\n";
>         for (int row = 0; row < grid.length; row++) {
>             for (int column = 0; column < grid[row].length; column++)
>                 result += grid[row][column] + "";
>             result += "\n";
>         }
>         return result;
>     }
> }
> ```
>
> 输出：
>
> ```
> 1110110001111
> 1011101111001
> 0000101010100
> 1110111010111
> 1010000111001
> 1011111101111
> 1000000000000
> 1111111111111
>
> The maze was successfully traversed!
>
> 7770110001111
> 3077707771001
> 0000707070300
> 7770777070333
> 7070000773003
> 7077777703333
> 7000000000000
> 7777777777777
> ```
>
> **基础情形：越界、阻塞与到达终点**
>
> ```java
> if (valid(row, column)) {
>     ...
>     if (row == grid.length - 1 &&
>         column == grid[0].length - 1)
>         done = true;
>     else { ... }
> }
> return done;
> ```
>
> - `valid` 不成立时 `done` 保持 `false`，当前层立即返回，递归不再向外扩展；这承担了"越界或不可通行"的基础情形。
> - `valid` 成立且已到达右下角时 `done` 置为 `true`，承担"到达终点"的基础情形；此时不再向四个方向继续发起递归。
>
> **递归情形：四方向依次尝试**
>
> ```java
> done = traverse(row + 1, column);
> if (!done) done = traverse(row, column + 1);
> if (!done) done = traverse(row - 1, column);
> if (!done) done = traverse(row, column - 1);
> ```
>
> - 四个方向按"下、右、上、左"顺序依次尝试，一旦某一方向返回 `true`，后续的 `if (!done)` 即被短路，避免在已找到路径后继续发起无意义的递归。
> - 每一层只负责扩展邻接的四格，整条路径由层层嵌套的 `traverse` 调用自然拼接。
>
> **路径标记：`TRIED` 与 `PATH` 的分工**
>
> ```java
> grid[row][column] = TRIED;
> ...
> if (done)
>     grid[row][column] = PATH;
> ```
>
> - 进入一格时先置为 `TRIED`，`valid` 随后不再将其判为可尝试，天然避免回头与环路。
> - 仅当本层 `done` 为 `true` 时再覆写为 `PATH`；未能通向终点的格子保留 `TRIED`，以此在最终网格中区分"探索过"与"属于解"的位置。


## 17.6 汉诺塔

汉诺塔 (*Towers of Hanoi*) 是由三根竖直柱与一组大小不同的圆盘组成的古典谜题。初始时所有圆盘按从大到小的顺序叠在一根柱上，目标是在以下两条规则下将所有圆盘全部移到另一根柱：

- 每次只能移动一张圆盘。
- 较大的圆盘不能压在较小的圆盘之上。

迭代解法相当复杂，而递归解法给出一个自然的自相似分解：将 $n$ 张圆盘从起始柱移到目标柱这一问题，总可以拆成三步——先把上方 $n-1$ 张圆盘借助目标柱移到辅助柱，再把最底下的一张从起始柱直接移到目标柱，最后把暂存在辅助柱上的 $n-1$ 张再借助起始柱移到目标柱。基础情形是 $n = 1$，此时只需一次直接移动即可完成。

> [!example]- 示例：`SolveTowers.java` / `TowersOfHanoi.java`
>
> 两文件共同演示递归在汉诺塔问题上的实现：`TowersOfHanoi` 封装圆盘数量并提供 `solve` 作为入口，`moveTower` 以递归形式打印全部移动指令；`SolveTowers` 的 `main` 方法创建一个 $4$ 张圆盘的实例并调用 `solve`，用以观察递归解法在具体规模下展开的完整移动序列。`solve` 的调用约定为"从柱 $1$ 出发、以柱 $2$ 为辅助、移到柱 $3$"。
>
> > [!info]- UML 框图
> >
> > ```
> > ┌──────────────────────────────────┐
> > │ SolveTowers                      │
> > │  + main(String[]): void          │
> > └──────────────┬───────────────────┘
> >                │ creates & uses
> >                ▼
> > ┌──────────────────────────────────┐
> > │ TowersOfHanoi                    │
> > │  - totalDisks: int               │
> > │  + TowersOfHanoi(int)            │
> > │  + solve(): void                 │
> > │  - moveTower(int,int,int,int):   │
> > │      void                        │
> > │  - moveOneDisk(int, int): void   │
> > └──────────────────────────────────┘
> > ```
>
> ```java
> // SolveTowers.java —— 触发递归解法
> public class SolveTowers {
>     public static void main(String[] args) {
>         TowersOfHanoi towers = new TowersOfHanoi(4);
>         towers.solve();
>     }
> }
> ```
>
> ```java
> // TowersOfHanoi.java —— 以递归打印移动序列
> public class TowersOfHanoi {
>     private int totalDisks;
>
>     public TowersOfHanoi(int disks) {
>         totalDisks = disks;
>     }
>
>     // 将 totalDisks 张圆盘从柱 1 借助柱 2 移到柱 3
>     public void solve() {
>         moveTower(totalDisks, 1, 3, 2);
>     }
>
>     // 将 numDisks 张从 start 借助 temp 移到 end
>     private void moveTower(int numDisks, int start, int end, int temp) {
>         if (numDisks == 1)
>             moveOneDisk(start, end);                         // base case
>         else {
>             moveTower(numDisks - 1, start, temp, end);       // 1. 上层先挪到辅助柱
>             moveOneDisk(start, end);                         // 2. 最底一张直达目标
>             moveTower(numDisks - 1, temp, end, start);       // 3. 上层再从辅助柱挪到目标
>         }
>     }
>
>     private void moveOneDisk(int start, int end) {
>         System.out.println("Move one disk from " + start + " to " + end);
>     }
> }
> ```
>
> 输出（$4$ 张圆盘）：
>
> ```
> Move one disk from 1 to 2
> Move one disk from 1 to 3
> Move one disk from 2 to 3
> Move one disk from 1 to 2
> Move one disk from 3 to 1
> Move one disk from 3 to 2
> Move one disk from 1 to 2
> Move one disk from 1 to 3
> Move one disk from 2 to 3
> Move one disk from 2 to 1
> Move one disk from 3 to 1
> Move one disk from 2 to 3
> Move one disk from 1 to 2
> Move one disk from 1 to 3
> Move one disk from 2 to 3
> ```
>
> **基础情形：单张圆盘直接移动**
>
> ```java
> if (numDisks == 1)
>     moveOneDisk(start, end);
> ```
>
> - 只有一张圆盘时不存在"上方 $n-1$ 张"需要让路，`moveTower` 直接调用 `moveOneDisk(start, end)` 输出一次移动并返回上一层。
> - 基础情形取 $n = 1$ 而非 $n = 0$，是为了让每次终止都对应一条可打印的移动指令；$n = 0$ 虽然在语义上对应"无操作"也能终止递归，但不输出有效结果。
>
> **递归情形：三步分解与柱编号的换位**
>
> ```java
> moveTower(numDisks - 1, start, temp, end);
> moveOneDisk(start, end);
> moveTower(numDisks - 1, temp, end, start);
> ```
>
> - 第一次递归调用将 $n-1$ 张从 `start` 经由 `end` 移到 `temp`：原先的目标柱在此子问题中被借作辅助。
> - 中间的 `moveOneDisk` 将最底一张从 `start` 移到 `end`；由于上方 $n-1$ 张已挪走，此时 `end` 上必然无更小圆盘，不违反规则。
> - 第二次递归调用将这 $n-1$ 张从 `temp` 经由 `start` 移到 `end`：原先的起始柱在此子问题中被借作辅助；两次递归调用之间 `end` 与 `temp` 的角色互换是递归分解的关键。
>
> **移动次数与调用深度**
>
> ```java
> moveTower(totalDisks, 1, 3, 2);
> ```
>
> - 设 $T(n)$ 为移动 $n$ 张圆盘所需的总次数，由递归结构得 $T(1) = 1$、$T(n) = 2T(n - 1) + 1$，解得 $T(n) = 2^n - 1$；$n = 4$ 时为 $15$ 次，与输出条数一致。
> - 递归调用的最大深度为 $n$，与圆盘数量呈线性关系，远小于移动总次数。


---
`Pre: ` [[ELEC2543 Ch.16 Exceptions]]
`Post:` [[ELEC2543 Ch.18 Sorting]]

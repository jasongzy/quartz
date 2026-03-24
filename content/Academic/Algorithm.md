---
publish: true
created: 2024-06-21T15:00:46.000+08:00
cssclasses: ""
---


## 基础知识

### 时间复杂度

$\Theta$：渐进紧确界

$O$：渐进上界（最坏情况）

$\Omega$：渐进下界（最好情况）

从上到下复杂度依次增加：

|  非正式术语   |   $O$ 记号    |
| :-----------: | :-----------: |
|    常数阶     |    $O(1)$     |
|    对数阶     | $O(\log{n})$  |
|    线性阶     |    $O(n)$     |
| $n\log{n}$ 阶 | $O(n\log{n})$ |
|    平方阶     |   $O(n^2)$    |
|    立方阶     |   $O(n^3)$    |
|  常数次方阶   |   $O(n^k)$    |
|    指数阶     |   $O(2^n)$    |
|    阶乘阶     |    $O(n!)$    |
|  $n$ 次方阶   |   $O(n^n)$    |

## 数据结构

**线性表（List）的概念**

> 零个或多个元素的有限序列

从存取结构上分为顺序（顺序表）和链式（链表）两种。栈和队列都属于线性表。

### 栈和队列

栈：先进后出（LIFO）

在 Python 中可以使用 `list` 直接实现，增删元素使用 `.append()` 和 `.pop()` 即可。

队列：先进先出（FIFO）

在 Python 中可以使用 `collections.deque` 来实现双端队列，使用与 list 类似，在左端（开头）增删元素可以使用 `.appendleft()` 和 `.popleft()`

#### 单调栈

[参考](https://leetcode.cn/problems/daily-temperatures/solutions/283196/mei-ri-wen-du-by-leetcode-solution/comments/2337370/)

常见题型：在 $O(n)$ 的时间复杂度内，找到数组中各个元素的“左侧/右侧首个比当前元素大/小的元素”的位置。

### 链表

#### 链表的后序遍历

法 1：**递归**

只需在递归的反向回溯阶段（递归调用之后）对当前值进行处理（如打印 / 加进列表 / return）：

```python
def linked_in_reverse(head)
    if head is not None:
        linked_in_reverse(head.next)
        print(head.val)
        return head
```

这样首次执行到递归调用语句的下一行的，就是满足递归边界条件的节点——链表的最后一个节点，然后将逆序遍历各节点。

法 2：正向遍历入栈，随后出栈

### 二叉树

定义：节点的度（子树数目）不大于 2 的有序树

#### 深度优先遍历（DFS）

都可以用  **递归**  或  **栈**  实现

其中递归隐式地利用了*调用栈*

递归方法的函数 `search` 非常简洁：按照指定的顺序执行 `visit(node)`、`search(node.left)`、`search(node.right)` 即可

##### 前序遍历

先序遍历

Pre-order

按照“根节点、左子节点、右子节点”的顺序遍历

用 **栈** 实现：

根节点 push 入栈，然后循环执行直到栈为空：

取栈顶节点并 pop 出栈，对该节点访问处理后，将该节点的**右左**子节点依次 push 入栈（节点为空则不入栈）。

##### 中序遍历

In-order

按照“左子节点、根节点、右子节点”的顺序遍历

用 **栈** 实现：

需要先下潜到最左下角才开始访问。这意味着左侧这条路径上的节点会被两次访问到（第一次下潜途中，第二次回溯正式访问），因此需要略微修改循环结构。

初始化空栈 `stack` ，初始化当前节点 `current = root`，然后循环执行以下步骤，循环条件是 `stack or current`：

- 子循环找到最左下角的节点：子循环条件 `current is not None`：`current` 节点入栈，然后令 `current = current.left`；跳出循环后，栈顶就是（未访问过的节点中的）最左下角的节点；
- 栈顶节点出栈赋值给 `current`，对该节点访问处理，然后令 `current = current.right`，然后开始下一次循环即可（此时已经完成了对该节点的左子节点和根节点的遍历，下次循环会把右子节点入栈并开始遍历该右子树）。

##### 后序遍历

Post-order

按照“左子节点、右子节点、根节点”的顺序遍历

用 **栈** 实现：

需要先下潜到最左下角才开始访问。可以用与[[Academic/Algorithm#中序遍历]]相仿的循环思路：

**（完全一致）**初始化空栈 `stack` ，初始化当前节点 `current = root`，然后循环执行以下步骤，循环条件是 `stack or current`：

- **（完全一致）**子循环找到最左下角的节点：子循环条件 `current is not None`：`current` 节点入栈，然后令 `current = current.left`；跳出循环后，栈顶就是（未访问过的节点中的）最左下角的节点；
- 栈顶节点赋值给 `current` 但**不出栈不访问**，判断：**上一次出栈访问的节点是否就是 `current.right` 或 `current.right is None`**：
  - 若是：说明已经完成了该节点的左子节点、右子节点遍历，下面遍历本身即可，因此将该节点出栈并访问处理（同时更新*上一次出栈访问的节点*），且**令 `current = None`（防止下次循环重复入栈）**；
  - 若否：说明只完成了左子节点遍历，下面遍历右子节点**（与中序遍历一致）**，因此令 `current = current.right`，然后开始下一次循环即可。

#### 广度优先遍历/层序遍历（BFS）

从左至右遍历完一层后再遍历下一层

用 **队列**（先入先出）实现：根节点入列，然后循环执行直到队列为空：

1. 从队列中 pop 出队
2. 对该节点访问处理
3. 将该节点的左右子节点依次 push 入队（子节点为空不要入队）

#### 平衡二叉树

任一节点对应的两棵子树的高度差 <= 1

#### 二叉搜索树

Binary Search Tree (BST)

对于每一个节点都满足：左子树的所有值都严格小于根节点的值，右子树的所有值都严格大于根节点的值。

允许左右节点为空；全空树也属于二叉搜索树；树内不存在重复值。

二叉搜索树的**中序遍历**是一个严格升序的数组。

### 优先队列/堆

添加元素后自动维护最值

Python 标准库中的 `heapq` 库默认是小根堆，即堆顶是最小值，其数组表示是升序排列，其二叉树表示中父节点始终小于等于其子节点。

```python
import heapq

h = mylist  # 堆没有自己的类，而是直接用数组形式表示
heapq.heapify(h)  # 堆化，原位操作
heapq.heappush(h, 4)  # 入堆并维护堆的性质
heapq.heappop(h)  # 弹出最小值
```

手动实现：[参考](https://www.hello-algo.com/chapter_heap/heap/)

### 图

## 基础算法

### 排序

#### 选择排序

##### 简单选择排序

Selection sort

最基础的排序算法

先遍历一遍数组，找到数组中的**最小值**，然后把它和数组的第一个元素交换位置；接着再遍历一遍数组，找到第二小的元素，和数组的第二个元素交换位置；以此类推，直到整个数组有序。

时间复杂度 $O(n^2)$
空间复杂度  $O(1)$

##### 堆排序

Heap sort

时间复杂度 $O(n\log{n})$
空间复杂度  $O(1)$

#### 交换排序

##### 冒泡排序

Bubble sort

**稳定排序**算法
时间复杂度 $O(n^2)$
空间复杂度  $O(1)$

##### 快速排序

Quick sort，分区交换排序

**分治 + 递归**

Python 的双指针原位交换实现（Hoare 霍尔分区方案）：

```python
def partition(arr, start: int, end: int):
  # 1. 选择基准值，这里我们选择最左边的元素
  pivot = arr[start]

  # 2. 初始化左右指针
  left = start
  right = end

  # 3. 开始分区循环，直到左右指针相遇
  while left < right:
    # 从右向左找，找到第一个小于基准值的数
    while left < right and arr[right] >= pivot:
      right -= 1

    # 从左向右找，找到第一个大于基准值的数
    while left < right and arr[left] <= pivot:
      left += 1

    # 交换左右指针指向的元素
    if left < right:
      arr[left], arr[right] = arr[right], arr[left]

  # 将基准值（最初在 arr[start]）与相遇点的元素交换
  arr[start], arr[left] = arr[left], arr[start]

  # 4. 返回基准值的最终索引
  return left

def quicksort(arr, start: int = None, end: int = None):
  # --- 处理初始调用 ---
  if start is None:
    start = 0
  if end is None:
    end = len(arr) - 1

  # --- 递归逻辑 ---
  # 递归的基线条件：当子列表只有一个或零个元素时 (start >= end) 停止
  if start < end:
    # 1. 调用 partition 函数，将列表分区并获取基准值的索引
    pivot_index = partition(arr, start, end)
    # 2. 对基准值左边的子列表进行递归排序
    quicksort(arr, start, pivot_index - 1)
    # 3. 对基准值右边的子列表进行递归排序
    quicksort(arr, pivot_index + 1, end)
```

注意，快排的基准值 pivot 理论上可以选择任意一个元素，但上述算法实现仅适用于指定**首个数**作为 pivot 的情形，因为我们是首先右向左找第一个小于基准值的数，然后再从左向右找第一个大于基准值的数，这两个操作的顺序不能交换，这样做保证了相遇点必然小于等于基准值，于是最后基准值与相遇点的元素交换时能够保证正确地把一个较小的数交换到左边（第一个位置）。

此方案的另一种不需要交换操作的实现：（[排序算法：快速排序 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/35946897)）

```c
QUICKSORT(A, low, high) {
  if low >= high
    return;
  i = low, j = high;
  x = a[low];  // 基准
  while (i < j) {
    // 找到最靠右首个小于基准的位置
    while (i < j && A[j] >= x) j--;
    if (i < j) A[i++] = A[j];  // 将A[j]填入A[i]，并将i向右移动
    // 找到最靠左首个大于基准的位置
    while (i < j && A[i] < x) i++;
    if (i < j) A[j--] = A[i];  // 将A[i]填入A[j]，并将j向右移动
    // 此时 i==j
    A[i] = x;  // 将基准数填入最后的坑
    QUICKSORT(A, low, i-1);
    QUICKSORT(A, i+1, high);
  }
}

QUICKSORT(A, 0, A.size()-1)
```

一种形似冒泡的实现（Lomuto 洛穆托分区方案）：

```c
QUICKSORT(A, p, r)
    if p < r
        q = PARTITION(A, p, r)
        QUICKSORT(A, p, q-1)
        QUICKSORT(A, q+1, r)
```

```c
PARTITION(A, p, r)
    x = A[r]  // 基准
    i = p
    for j = p to r-1
        if A[j] < x
            // 把比基准小的数移到左端
            exchange A[i] with A[j]
            i = i + 1
            // 此时下标<i的数必然都小于基准数
    // 此时i指向（从左到右）首个比基准数大的数
    exchange A[i] with A[r]  // 把基准数放到交界位置i上
    return i
```

#### 插入排序

##### 直接插入排序

Insertion sort

有序输出序列最初包含首个元素，然后迭代后续元素。
每次迭代中，从输入数组中移除一个待排序的元素，找到它在有序输出序列中适当的位置，并将其插入。
重复直到所有输入数据插入完为止。

**稳定排序**算法
时间复杂度 $O(n^2)$
空间复杂度  $O(1)$

##### 希尔排序

Shell sort，缩小增量排序法

对插入排序的简单改进

#### 归并排序

Merge sort

基于分治思想，将数组分段排序后合并

最核心的部分是**合并有序数组**过程：将两个有序（升序）的数组 `a` 和 `b` 合并为一个有序数组。方法是同时依次遍历两个数组的元素，将 `a[i]` 和 `b[j]` 中的较小值放入新数组。
`a` 和 `b` 实际上是原始数组的前段和后端，为了保证算法的稳定性，`a[i] == b[j]` 时应将 `a[i]` 视为较小值。

然后可以用两种不同的方法实现排序：

分治法（递归/自顶向下）：

1. 当数组长度为 $1$ 时，该数组就已经是有序的，不用再分解，直接返回；
2. 当数组长度大于 $1$ 时，就将该数组分为两段（为保证排序的复杂度，通常将数组分为尽量等长的两段），分别递归调用本排序函数对它们进行排序。然后再将它们合并为一个有序数组。

倍增法（自底向上）：

已知当数组长度为 $1$ 时，该数组就已经是有序的。

将数组全部切成长度为 $1$ 的段。

从左往右依次合并两个长度为 $1$ 的有序段，得到一系列长度 $\le 2$ 的有序段；

从左往右依次合并两个长度 $\le 2$ 的有序段，得到一系列长度 $\le 4$ 的有序段；

从左往右依次合并两个长度 $\le 4$ 的有序段，得到一系列长度 $\le 8$ 的有序段；

……

重复上述过程直至数组只剩一个有序段，该段就是排好序的原数组。

基于比较的**稳定排序**算法
时间复杂度 $O(n\log{n})$
空间复杂度  $O(n)$

#### 线性时间排序

##### 计数排序

Counting sort

##### 基数排序

Radix sort

##### 桶排序

Bucket sort

#### Tim 排序

Timsort

Python 的标准库默认算法，巧妙结合了[[Academic/Algorithm#插入排序]]和[[Academic/Algorithm#归并排序]]的优点

**稳定排序**算法

### 二分查找

```python
idx = bisect.bisect_left(array, target)
```

时间复杂度为 $O(\log{n})$

二分的核心思想在于：每次淘汰一半，在另一半中继续二分

要求待查找的数组是单调（升序）的

模板：

```python
left = 0
right = len(nums) - 1
while left <= right:
    mid = (left + right) // 2
    if target <= nums[mid]:
        right = mid - 1
    else:
        left = mid + 1
return left  # 首个数值大于等于target的下标
```

关于上述程序模板中的 `<=` 和 `+-1` 等细节问题：记住 `target` 的可能位置始终在一个**左闭右闭**的区间 `[left, right]`，因此： `right` 初始化在数组有效范围内；`left == right` 是有意义的，因此 `while` 的条件是 `<=`；一旦 `target` 不等于 `mid` 值，新的可能区间就一定不包含 `mid`，因此两个指针在移动时都要 `+ 1 / - 1`（[参考](https://programmercarl.com/0704.%E4%BA%8C%E5%88%86%E6%9F%A5%E6%89%BE.html#%E6%80%9D%E8%B7%AF)）。

### 双指针

> 在遍历元素的过程中，不是使用单个指针进行访问，而是使用两个指针进行访问，从而达到相应的目的。如果两个指针方向相反，则称为「对撞指针」。如果两个指针方向相同，则称为「快慢指针」。如果两个指针分别属于不同的数组 / 链表，则称为「分离双指针」。

在数组的区间问题上，暴力算法的时间复杂度往往是 $O(n^2)$。而双指针利用了区间**单调性**的性质，可以将时间复杂度降到 $O(n)$，即在一个循环内完成了原本需要两个循环的工作。

移动速度之间是 2 倍关系的快慢指针还可以用来寻找链表等结构的**中点**。

### 滑动窗口

滑动窗口可以归为快慢双指针，一快一慢两个指针前后相随，中间的部分就是窗口。

滑动窗口算法技巧主要用来解决子数组问题，比如寻找符合某个条件的最长/最短子数组。

穷举所有可能的子数组需要用两层 for 循环，时间复杂度 $O(n^2)$。但很多问题中，并不需要穷举所有子串就能找到答案，因此就可以用滑动窗口法将复杂度降为 $O(n)$。

滑动窗口的代码框架是用两层循环，分别右移左右指针，以减小和增大窗口。因为两个指针都不会回退，所以数组中的每个元素都最多只会进入窗口一次/移出窗口一次，不会重复进入和离开，于是就对穷举过程完成了剪枝优化，避免了冗余计算。

## 高级设计和分析技术

### 递归

[递归之我见](https://leetcode.cn/problems/fan-zhuan-lian-biao-lcof/solution/kan-bu-dong-di-gui-de-kan-guo-lai-xi-wan-1akq/)

[干货满满！全面详解如何用递归解题！](https://mp.weixin.qq.com/s/krVKQXbcqdK298rOh0uiFA)

### 回溯

回溯的本质是穷举，用于解决排列组合等需要**寻找所有可行解**的问题。这些问题用遍历的思路需要嵌套层数为变量的 for 循环，很难用代码写出。回溯算法就是一种方便代码实现的“聪明地穷举”方法。

回溯用递归实现，是一个多叉树遍历问题，核心是先做出本次选择，接着递归调用，然后在递归调用之后的回溯阶段**撤销**本次选择。

模板：

```python
def backtrack(路径, 选择列表):
    if 终止条件:
        存放当前路径作为结果
        return

    for 选择 in 选择列表:
        # 做选择：
        将该选择从选择列表移除
        路径.add(选择)
        backtrack(路径, 选择列表)
        # 撤销选择：
        路径.remove(选择)
        将该选择再加入选择列表
```

其中，for 循环可以理解为在多叉树的一个子树中横向遍历（这个子树的根节点度数就是循环次数），backtrack 递归过程就是在多叉树中纵向遍历。

### 动态规划

[概念参考](https://blog.csdn.net/IT__learning/article/details/121883738)

- 自顶向下的动态规划：带缓存的递归
- 自底向上的动态规划：复杂度更低
  一维情况可以用缓存数组或**滑窗**（滚动数组）实现，前者空间复杂度 $O(n)$ ，后者反复迭代状态转移方程中的变量，因此空间复杂度$O(1)$。

[代码参考](https://labuladong.online/algo/essential-technique/dynamic-programming-framework-2)

状态是 `dp` 数组的下标索引，子问题答案是 `dp` 数组的存储内容

一维动态规划的自底向上实现模板：

```python
dp = [0] * (len(arr) + 1)
dp [0] = 1  # 根据实际情况初始化一个或多个 base cases
for i in range(1, len(arr) + 1):  # 遍历所有未初始化的状态
    for 选择 in 选择列表:
        利用 dp 中的先前结果和状态转移方程，求此选择下的答案
    整合所有选择的答案（累计或择优），并更新 dp[i]
return dp[-1]  # 有时问题的最终答案也可能类似 max(dp)
```

#### 0-1 背包问题

[0-1 背包问题](https://www.hello-algo.com/chapter_dynamic_programming/knapsack_problem/)（每个物品只能选择一次）：

做出某一选择后，子问题的选择列表和答案会发生变化，因此需要用二维的状态来记录 `dp` 表。

`dp[i][c]` 的定义如下：只可以选择前 `i` 个物品，当前背包的容量为 `c`，这种情况下可以装的最大价值是 `dp[i][c]`。

```python
def knapsack(cap: int, wgt: List[int], val: List[int]) -> int:
    assert len(wgt) == len(val)
    N = len(wgt)  # 物品种类
    # base case 已初始化
    dp = [[0] * (cap+1) for _ in range(N+1)]  # (N+1) * (cap+1)
    for i in range(1, N+1):  # 遍历可选物品（可选前i个）
        for c in range(1, cap+1):  # 遍历所有要求的背包容量
            if c - wgt[i-1] < 0:
                # 超过背包容量，只能选择不装物品i
                dp[i][c] = dp[i-1][c]
            else:
                # 装入或者不装入背包，择优
                dp[i][c] = max(
                    dp[i-1][c - wgt[i-1]] + val[i-1],
                    dp[i-1][c]
                )
    return dp[N][cap]
```

从状态转移方程可以看出，`dp[i]` 只与其上一行的 `dp[i-1]`（以及第 `i` 个物品自身的重量和价值）有关，即当前状态是从左上和正上的状态转移而来的，因此可以进行空间优化，用一维 `dp` 数组来实现。在代码实现中，我们仅需将数组 `dp` 的第一维（`i`）直接删除，并且把内循环（`c`）更改为**倒序遍历**（防止左边的值被覆盖）即可（刚开始遍历第 `i` 个物体时，一维 `dp` 数组内存储的是第 `i-1` 个物体的所有结果）：

```python
def knapsack(cap: int, wgt: List[int], val: List[int]) -> int:
    assert len(wgt) == len(val)
    N = len(wgt)  # 物品种类
    # base case 已初始化
    dp =[0] * (cap+1)
    for i in range(1, N+1):  # 遍历可选物品（可选前i个）
        for c in range(cap, 0, -1):  # 倒序遍历所有要求的背包容量
            if c - wgt[i-1] < 0:
                # 超过背包容量，只能选择不装物品i
                dp[c] = dp[c]
            else:
                # 装入或者不装入背包，择优
                dp[c] = max(
                    dp[c - wgt[i-1]] + val[i-1],
                    dp[c]
                )
    return dp[cap]
```

---

#### 完全背包问题

[完全背包问题](https://www.hello-algo.com/chapter_dynamic_programming/unbounded_knapsack_problem/)（不限制物品的选择次数）：

```python
def unbounded_knapsack(cap: int, wgt: List[int], val: List[int]) -> int:
    assert len(wgt) == len(val)
    N = len(wgt)  # 物品种类
    # base case 已初始化
    dp = [[0] * (cap+1) for _ in range(N+1)]  # (N+1) * (cap+1)
    for i in range(1, N+1):  # 遍历可选物品（可选前i个）
        for c in range(1, cap+1):  # 遍历所有要求的背包容量
            if c - wgt[i-1] < 0:
                # 超过背包容量，只能选择不装物品i
                dp[i][c] = dp[i-1][c]
            else:
                # 装入或者不装入背包，择优
                dp[i][c] = max(
                    dp[i][c - wgt[i-1]] + val[i-1],
                    dp[i-1][c]
                )
    return dp[N][cap]
```

对比 0-1 背包问题，完全背包的状态转移中有一处从 `i-1` 变为 `i`（因为即使选择放入物品 `i`，子问题也仍然可以继续使用物品 `i`），其余完全一致。

当前状态是从左边和上边的状态转移而来的，因此进行类似的空间优化，每次对一行进行**正序遍历**即可（右边的值依赖于左边的历史值），实现上只需删除 `dp` 数组的第一个维度：

```python
def unbounded_knapsack(cap: int, wgt: List[int], val: List[int]) -> int:
    assert len(wgt) == len(val)
    N = len(wgt)  # 物品种类
    # base case 已初始化
    dp =[0] * (cap+1)
    for i in range(1, N+1):  # 遍历可选物品（可选前i个）
        for c in range(1, cap+1):  # 遍历所有要求的背包容量
            if c - wgt[i-1] < 0:
                # 超过背包容量，只能选择不装物品i
                dp[c] = dp[c]
            else:
                # 装入或者不装入背包，择优
                dp[c] = max(
                    dp[c - wgt[i-1]] + val[i-1],
                    dp[c]
                )
    return dp[cap]
```

此实现与 0-1 背包问题实现的差别仅仅在于内层循环是否倒序遍历。即：**正序遍历状态是可以无限选择的完全背包问题，倒序遍历状态是只能选择一次的 0-1 背包问题**。

可以观察到，空间优化后的代码结构与[[Academic/Algorithm#动态规划\|一维动态规划实现模板]]是等价的（内外层循环交换了一下，对于完全背包问题没有实质影响）。这是因为，一维动规的状态只有背包容量，也就是说，任意背包容量的子问题都可以随意使用全部的物品（选择列表始终不变），这与完全背包问题的定义是一致的。

### 贪心算法

## 其他算法

### 快速幂

分治法：若指数是偶数，可以将底数平方，那么指数就可以减半；若指数是奇数，可以先乘一次底数，然后剩余的指数就成为了偶数，依然可以底数平方 + 指数减半。

递归实现：

```python
def power(x: int, n: int):
    if n == 0:
        return 1
    if n < 0:
        return 1 / power(x, -n)

    half = power(x, n // 2)
    if n % 2 == 0:
        return half * half
    else:
        return half * half * x
```

迭代实现：

```python
def power(x: int, n: int):
    if n < 0:
        x = 1 / x
        n = -n

    result = 1
    current_product = x
    while n > 0:
        if n % 2 == 1:
            result *= current_product
        current_product *= current_product
        n //= 2
    return result
```

### 辗转相除法

求最大公约数：$\operatorname{gcd}(a,\, b) = \operatorname{gcd}(b, \, a \mod b)$

```python
def gcd(a: int , b: int):
    while b:
        a, b = b, a % b
    return a
```

输入的两个数字的大小关系是无所谓的：若 a 小于 b，第一次迭代就会使二者交换。

PS：标准库实现

```python
import math

# 最大公约数
math.gcd(a, b)
# 最小公倍数
math.lcm(a, b) == (a * b) // math.gcd(a, b)
```

### Dijkstra

使用单源最短路径算法 Dijkstra 算法，可以在图中**寻找一个“源节点”到所有其他节点的最短路径**，其主要思想是贪心算法。

Dijkstra 算法只能用在**权重非负**的图中，即不能出现“多绕路反而路线更短”的情况。对于带负权重边的图，需要使用其他算法（如 Bellman-Ford 算法）来解决最短路径问题。

参考题解：[743. 网络延迟时间 - 力扣（LeetCode）](https://leetcode.cn/problems/network-delay-time/solutions/2668220/liang-chong-dijkstra-xie-fa-fu-ti-dan-py-ooe8/)

### 数位 DP

虽然是动态规划，但由于情况复杂，更适合采用自顶向下的“记忆化搜索”递归方案。

**通用模板**参考题解：[902. 最大为 N 的数字组合 - 力扣（LeetCode）](https://leetcode.cn/problems/numbers-at-most-n-given-digit-set/solutions/1900101/shu-wei-dp-tong-yong-mo-ban-xiang-xi-zhu-e5dg/?envType=problem-list-v2&envId=x9p9agni)

### 树状数组

### 线段树

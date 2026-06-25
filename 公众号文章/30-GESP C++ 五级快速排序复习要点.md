
# GESP C++ 五级快速排序复习要点


GESP C++ 五级真题的客观题中`快速排序`是高频考点。一是考察分治的思想，二是分治思想应用过程中分区、合并等实操技巧。题型覆盖：代码阅读题、代码补全、编程大题，常结合 **时间复杂度、稳定性、算法原理、编程手写代码** 综合考查，是五级区分基础考生与高分考生的重点。

前面的文章有简单的总结，感兴趣可以阅读参考。

我们今天主要是通过真题考察掌握的程度和深入的分析理解。

---

快排的一般是递归实现，核心逻辑就三步：
1. 分区，根据基准值（pivot）把数据分成左、右、pivot三部分 
2. 快排左分区 
3. 快排右分区

递归终止条件就是分区中只有一个元素，双指针情况下就是 `left == right`

真题主要是围绕着分区的方法，从编程技巧的角度考察实现分区逻辑的细节：遍历、交换和基准值（pivot）归位。
### 2026年03月-经典霍尔分区

第 13 题：有 位同学的成绩已经从⼩到⼤排好序，现在对它执⾏下⾯这段以第⼀个元素为 pivot 的快速排序，请问此次排序的时间复杂度是（ ）。

```cpp
void quicksort(vector<int>& a, int l, int r) {
	if (l >= r) return;
	int pivot = a[l];
	int i = l, j = r;
	while (i < j) {
		while (i < j && a[j] >= pivot) j--;
		while (i < j && a[i] <= pivot) i++;
		if (i < j) swap(a[i], a[j]);
	}
	swap(a[l], a[i]);
	quicksort(a, l, i - 1);
	quicksort(a, i + 1, r);
}
```

2025年9月和12月有个相近的考题
```cpp
int partition(vector<int>& arr, int low, int high) {
	int i = low, j = high;
	int pivot = arr[low]; // 以首元素为基准
	while (i < j) {
		while (i < j && arr[j] >= pivot) j--;
		while (i < j && arr[i] <= pivot) i++;
		if (i < j) swap(arr[i], arr[j]);
	}
	swap(arr[i], arr[low]);
	return i;
}
void quickSort(vector<int>& arr, int low, int high) {
	if (low >= high) return;
	int p = partition(arr, low, high);
	quickSort(arr, low, p - 1);
	quickSort(arr, p + 1, high);
}
```


答案：$O(n^2)$

虽然考察的是最坏情况下的时间复杂度，但是我们还是认真的分析一下源代码，理解和掌握`经典霍尔分区`的数据交换、pivot归位的编程细节。
#### 核心分区逻辑

1. 先动`右指针 j`，再动`左指针 i`（顺序不能颠倒，是经典霍尔分区的关键点）；
2. 规则：
    
    - `j` 从右往左扫，跳过所有 `≥ pivot`的数，停在**小于pivot**的位置；
    - `i` 从左往右扫，跳过所有 `≤ pivot`的数，停在**大于pivot**的位置；
    
3. 两指针`i`和`j`未相遇就交换元素，分别把大小元素放到了合适的分区，继续循环；
4. `i == j` 时，该位置就是`pivot`的最终落点，将`pivot`，即：`arr[low]` 与 当前`i`和`j`的相遇点：`arr[i]` 交换，`pivot` 归位，也就是放到了中间的位置，形成了一个`左边`都是`比pivot小`，`右边`都是`比pivot大`的状态。

#### 为什么必须先移动右指针？

> 可以想思考一下......，然后再看下文分析

基准pivot在最左侧 `arr[low]`：

如果先移左指针，最后相遇位置的元素`> pivot`，导致运行`	swap(arr[i], arr[low]);` 交换后会破坏分区规则，把一个大于pivot的数据放到左边分区中；

先移右指针，相遇位置元素一定 `≤ pivot`，交换后天然满足：

- `[low, i-1]`：全部 ≤ pivot
- `arr[i]`：pivot（分界点）
- `[i+1, high]`：全部 ≥ pivot


说了这么多“逻辑”过程，我们可以这样理解这个分区的编程技巧：
1. 第一个元素是 pivoit
2. 双指针，左右交换大小元素；
3. 在数组的中间相遇，相遇点的元素大小 ≤ pivot;
4. 把相遇点的元素跟 pivot 交换；

### 2025年3月 - 霍尔分区变种

考虑以下C++代码实现的快速排序算法，将数据从⼩到⼤排序，则横线上应填的最佳代码是( )。

```cpp
int partition(vector<int>& arr, int low, int high) {
	int pivot = arr[high]; // 基准值
	int i = low - 1;
	for (int j = low; j < high; j++) {
		________________________________ // 在此处填入代码
	}
	swap(arr[i + 1], arr[high]);
	return i + 1;
}
// 快速排序
void quickSort(vector<int>& arr, int low, int high) {
	if (low < high) {
		int pi = partition(arr, low, high);
		quickSort(arr, low, pi - 1);
		quickSort(arr, pi + 1, high);
	}
}
```

**答案：**

```cpp
	if (arr[j] < pivot) {
		i++;
		swap(arr[i], arr[j]);
	}
```

2025年6月有一个相似的考题
```cpp

int randomPartition(std::vector<int>& arr, int low, int high) {
	int random = low + rand() % (high - low + 1);
	std::swap(arr[random], arr[high]);
	int pivot = arr[high];
	int i = low - 1;
	for (int j = low; j < high; j++) {
		if (arr[j] <= pivot) {
			i++;
			std::swap(arr[i], arr[j]);
		}
	}
	std::swap(arr[i + 1], arr[high]);
	return i + 1;
}
void quickSort(std::vector<int>& arr, int low, int high) {
	if (low < high) {
		int pi = randomPartition(arr, low, high);
		quickSort(arr, low, pi - 1);
		quickSort(arr, pi + 1, high);
	}
}
```

这两个整体都是**经典单指针版 Partition**（霍尔分区变种，Lomuto分区，以最右侧元素为基准）。其中第二个考题，增加了**随机选基准**的优化。我们依旧逐行分析代码，掌握该变种的pivot选取、分区流程、元素交换与 pivot 归位的编程细节。

考虑到“细节”的文字太多，容易乏味，我们可以这样简单的理解这个分区的编程技巧：
1. `最后一个元素`是 pivoit；
2. `i` 指向 `≤pivot` 的`最后一个元素`，一开始没有，就指向了 `-1`;
3. 单指针 `j`，从左向右遍历元素，遇到 `>pivot` 的元素是就继续，因此`i` 和 `j` 之间的元素都是大于 pivot 的。
4. 单指针 `j`, 从左向右遍历元素，当遇到 `≤pivot` 的元素时，就要把这个元素放到 `i`指向元素的后面。这个就是为什么要 `i++` 的原因。因为`i++` 指向的元素肯定 `＞pivot`，这个已经被 `j` 证明过了。因此交换后，`j` 指向的 `≤pivot` 元素放到了 `i++`, `i++` 指向的 `＞pivot` 元素放到了 `j`。完美的实现了大小元素的交换，放置到合理的分区。
5. `j` 走到了最右边，循环结束，就把最后一个 `pivot` 元素跟 `i++` 指向的元素交换；这样 `pivot` 就放到了小元素的最后面，大元素的最前面。

😭好像说的也不少..........，希望有助于理解更加详细的代码分析

#### 核心分区逻辑

1. 首先在当前区间 `[low, high]` 内随机选出一个下标，将该位置元素与区间最右端元素交换，把随机元素转为新的 pivot 值；
2. 定义分界指针 `i`，初始赋值为 `low - 1`，代表`≤ pivot区域的末尾位置`，初始落在有效区间左侧之外，即：-1；
3. 遍历指针 `j` 从区间左边界 `low` 开始，一直遍历到 `high - 1`：
    - 若当前元素 `arr[j] ≤ pivot`，说明该元素属于左侧区间，先将分界指针 `i` 右移一位，再交换 `arr[i]` 与 `arr[j]`；
    - 若当前元素 `arr[j] > pivot`，不做任何操作，该元素保留在右侧区间；
4. 一轮遍历结束后，`[low, i]` 区间全部元素 `≤ pivot`，`[i+1, high-1]` 区间全部元素 `> pivot`；
5. 将最右端的pivot元素 `arr[high]`，与 `arr[i+1]` 交换，完成 pivot 归位，`i+1` 即为`pivot`的最终分界下标。

#### 指针 i 的精确含义

在单指针版的分区里：

- `pivot = arr [high]`（最右为基准）
- `i = low − 1`
- `i 的语义`：`始终指向「已确认 ≤ pivot 区域的最后一个位置」`

初始时：

- 还没有任何元素被确认 ≤ pivot
- 所以 `≤ pivot 的区域` 是空的
- 空区域的 “末尾” 自然就在整个区间 `[low, high‑1]`的左边外面 → 就是 `low − 1`

#### 为什么要增加随机选基准的逻辑？

先思考一下原有固定取端点做基准的问题，再看下方分析。

原版代码固定选取区间最右端 `arr[high]` 作为基准：
当数组本身**完全有序、完全逆序**时，每次分区都会出现一侧为空、一侧包含几乎所有元素的情况，递归深度接近数组长度，算法时间复杂度退化到最坏的 $O(n^2)$，同时还容易触发递归栈溢出。

引入随机选基准后：
1. `随机交换后`再以`最右端`元素作为 pivot，打乱了有序序列带来的极端分区情况，大幅降低出现最坏分区的概率；
2. 绝大多数场景下算法都能维持平均时间复杂度 $O(n\log n)$，是快速排序最常用、最简单的工程优化手段；
3. 分区的核心交换、指针逻辑完全沿用经典单指针分区规则，仅改变了基准的来源，分区后依旧满足：
    - `[low, pi-1]`：全部 `≤ pivot`
    - `arr[pi]`：`pivot`（分界点）
    - `[pi+1, high]`：全部 `> pivot`


如果仍然难以理解，可以用微信搜索“快速排序分区”的视频，根据形象的动画演示，结合文字，更加深刻的理解。

加油💪，满分！！！



# GESP C++ 分治算法思想——归并排序


归并排序是分治算法的典型应用，由冯·诺依曼于1945年提出，通过“先分解后合并”的方式实现排序。

**算法原理**：**不断二分拆分** + **有序合并**
- **分解**：将数组从中间位置划分为左右两个子数组，递归执行分解操作，直至每个子数组仅含1个元素（此时子数组天然有序）。
- **合并**：将两个有序子数组合并为一个更大的有序数组，通过双指针依次比较两个子数组的元素，将较小值放入临时数组，最终将临时数组结果复制回原数组。

## 一、详细步骤（逐层展示）

以数组：`[38, 27, 43, 3, 9, 82, 10]`为例，逐层演示”分解+合并“的过程。注意观察递归的特点，自顶向下的分解，触底后，再自下向上的合并的过程。跟堆栈数据结构，函数的栈帧等概念一起联想。

注：也可以通过微信搜索”归并排序“，我看了前面的几个视频，配合动画演示，讲解地还是挺清楚的。

### 第 1 次拆分
`[38, 27, 43, 3, 9, 82, 10]`
→ 左：`[38, 27, 43]`
→ 右：`[3, 9, 82, 10]`

---

### 处理左半部分：`[38, 27, 43]`
#### 第 2 次拆分
`[38, 27, 43]`
→ 左：`[38]`
→ 右：`[27, 43]`

#### 第 3 次拆分（拆到最小单元）
`[27, 43]`
→ 左：`[27]`
→ 右：`[43]`

#### 开始合并
1. 合并 `[27]` 和 `[43]` → `[27, 43]`
2. 合并 `[38]` 和 `[27, 43]` → `[27, 38, 43]`

左半部分最终有序：
`[27, 38, 43]`

---

### 处理右半部分：`[3, 9, 82, 10]`
#### 第 2 次拆分
`[3, 9, 82, 10]`
→ 左：`[3, 9]`
→ 右：`[82, 10]`

#### 第 3 次拆分（拆到最小单元）
`[3, 9]` → `[3]`、`[9]`
`[82, 10]` → `[82]`、`[10]`

#### 开始合并
1. 合并 `[3]` 和 `[9]` → `[3, 9]`
2. 合并 `[82]` 和 `[10]` → `[10, 82]`
3. 合并 `[3, 9]` 和 `[10, 82]` → `[3, 9, 10, 82]`

右半部分最终有序：
`[3, 9, 10, 82]`

---

### 顶层最终合并
合并：
`[27, 38, 43]` 和 `[3, 9, 10, 82]`

→ 最终结果：
`[3, 9, 10, 27, 38, 43, 82]`

---

### 完整树形结构
```
                  [38,27,43,3,9,82,10]
                 /                    \
        [38,27,43]                  [3,9,82,10]
        /        \                  /          \
    [38]     [27,43]           [3,9]        [82,10]
             /     \            /   \        /     \
          [27]   [43]        [3]   [9]   [82]   [10]
```


## 二、**代码实现**
```cpp
#include <iostream>
#include <vector>
using namespace std;

// 合并两个有序子数组：[left, mid] 和 [mid+1, right]
void merge(vector<int>& arr, int left, int mid, int right) {
    int n1 = mid - left + 1; // 左子数组长度
    int n2 = right - mid;    // 右子数组长度
    vector<int> L(n1), R(n2); // 临时数组存储子数组元素

    // 复制原数组元素到临时数组
    for (int i = 0; i < n1; i++) L[i] = arr[left + i];
    for (int j = 0; j < n2; j++) R[j] = arr[mid + 1 + j];

    // 双指针合并临时数组到原数组
    int i = 0, j = 0, k = left;
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) {
            arr[k++] = L[i++];
        } else {
            arr[k++] = R[j++];
        }
    }
    // 复制剩余元素
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}

// 归并排序主函数
// 要结合上面的逐层展示的步骤来看：左半部分、右半部分和整体合并三大步
void mergeSort(vector<int>& arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2; // 防止整数溢出
        mergeSort(arr, left, mid);   // 递归排序左子数组
        mergeSort(arr, mid + 1, right); // 递归排序右子数组
        merge(arr, left, mid, right); // 合并两个有序子数组
    }
}

int main() {
    vector<int> arr = {38, 27, 43, 3, 9, 82, 10};
    mergeSort(arr, 0, arr.size() - 1);
    for (int x : arr) cout << x << " ";
    return 0;
}
```

下面是用AI工具（豆包）增加了打印的内容，可以看到详细的分解和合并的过程。

```cpp
#include <iostream>
#include <vector>
using namespace std;

// 打印数组
void printArr(vector<int>& arr, int left, int right) {
    for (int i = left; i <= right; i++) {
        cout << arr[i] << " ";
    }
}

// 合并两个有序子数组：[left, mid] 和 [mid+1, right]
void merge(vector<int>& arr, int left, int mid, int right) {
    int n1 = mid - left + 1;
    int n2 = right - mid;
    vector<int> L(n1), R(n2);

    for (int i = 0; i < n1; i++) L[i] = arr[left + i];
    for (int j = 0; j < n2; j++) R[j] = arr[mid + 1 + j];

    cout << "→ 合并：[";
    printArr(L, 0, L.size()-1);
    cout << "] 和 [";
    printArr(R, 0, R.size()-1);
    cout << "]\n";

    int i = 0, j = 0, k = left;
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) arr[k++] = L[i++];
        else arr[k++] = R[j++];
    }
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];

    cout << "  合并结果：[";
    printArr(arr, left, right);
    cout << "]\n\n";
}

// 归并排序主函数
void mergeSort(vector<int>& arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;

        cout << "拆分：[";
        printArr(arr, left, right);
        cout << "] → 左：[";
        printArr(arr, left, mid);
        cout << "], 右：[";
        printArr(arr, mid+1, right);
        cout << "]\n";

        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);
        merge(arr, left, mid, right);
    } else {
        cout << "最小单元：[" << arr[left] << "]\n"; // 只有一个元素
    }
}

int main() {
    vector<int> arr = {38, 27, 43, 3, 9, 82, 10};
    cout << "原始数组：";
    for (int x : arr) cout << x << " ";
    cout << "\n-------------------------\n";

    mergeSort(arr, 0, arr.size() - 1);

    cout << "-------------------------\n最终排序：";
    for (int x : arr) cout << x << " ";
    return 0;
}
```

## 三、**性能分析**

- **时间复杂度**：无论输入数据是否有序，分解层数固定为log n，每层合并操作耗时O(n)，总时间复杂度为`O(n log n)`。
- **空间复杂度**：需要额外`O(n)`的临时数组存储合并结果。
- **稳定性**：`稳定排序`，相等元素的相对位置在合并过程中不会改变。

### 时间复杂度

1. 拆分过程（递归）
	- 每次把数组**一分为二**，一共会递归拆分出 **log₂n 层**。
	- 例如 n=7，层数 ≈ 3 层。
2. 合并过程
	- 每一层所有子数组加起来，**总长度都是 n**。
	- 每一层的合并操作都是 **O(n)**。
3. 总时间复杂度
	- 总共有 log₂n 层，每层 O(n)
	- → 时间复杂度：O(n log n)
4. 特点
	- 最好、最坏、平均情况 **都是 O(n log n)**
	- 不会像快排那样退化到 O(n²)
	- 稳定排序（相等元素相对位置不变）

---

### 空间复杂度

根据代码：
```cpp
vector<int> L(n1), R(n2);
```
每次 merge 都开辟了**临时数组**。

1. 递归栈空间
	- 递归深度是 log₂n
	- → 栈空间：**O(log n)**
2.  临时数组空间
	- 合并时最多需要 **n 大小的辅助空间**
	- → 堆/临时空间：**O(n)**
3. 总空间复杂度
	- 取最大项
	- → **空间复杂度：O(n)**


### 稳定性的原因分析

原因就在这一行代码里：
```cpp
if (L[i] <= R[j]) {
    arr[k++] = L[i++];
} else {
    arr[k++] = R[j++];
}
```

关键点：
- 当 **左边元素 ≤ 右边元素** 时，优先取**左边**的元素
- 也就是说：**值相等时，原来在前的元素依然在前**

这就保证了相等元素的相对顺序不变。

举个简单的例子
- 原始：`[ 3, 2ₐ, 2ᵦ, 4 ]`
- 拆分后左：`[3, 2ₐ]`，右：`[2ᵦ, 4]`
- 合并比较 `2ₐ` 和 `2ᵦ` 时：
	- `2ₐ <= 2ᵦ` → 先放 `2ₐ`，再放 `2ᵦ`
	- 顺序保持 `2ₐ` 在前，`2ᵦ` 在后

所以归并排序稳定。

学习归并排序前需要深入的了解递归的知识，GESP五级的考试大纲也有具体的要求。后面我根据情况再展开学习。
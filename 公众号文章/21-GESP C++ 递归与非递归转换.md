# GESP C++ 递归与非递归转换

递归与非递归转换的核心是：**尾递归/单向递归可直接转循环；复杂递归必须用栈模拟系统调用栈**。下面按方法分类，给出通用步骤、原理与代码示例。

### 一、递归转非递归的核心动机
- 避免**栈溢出**（递归深度过大）
- 提升**执行效率**（减少函数调用开销）
- 适配**无递归环境**（部分嵌入式/底层开发）

### 二、主流转换方法（按复杂度排序）
#### 1. 尾递归 → 直接迭代（最简单）
**定义**：递归调用是函数**最后一步操作**，无后续计算。
**原理**：无需保存当前栈帧，直接用循环更新参数。
**通用步骤**：
1. 提取**基线条件（base case）** 作为循环终止条件
2. 用变量保存**累加器/中间结果**
3. 循环更新参数，直到满足基线

**示例：阶乘（尾递归→迭代）**
```cpp
// 尾递归版
long long fact_tail(int n, long long acc = 1) {
    if (n <= 1) return acc;
    return fact_tail(n-1, n * acc); // 最后一步是递归
}

// 非递归（迭代）版
long long fact_iter(int n) {
    long long res = 1;
    for (int i = 2; i <= n; i++) res *= i;
    return res;
}
```

#### 2. 单向递归 → 迭代（如斐波那契）
**定义**：递归方向单一（如 n→n-1→…→0），无分支回溯。
**原理**：用变量保存前序状态，循环递推。
**示例：斐波那契**
```cpp
// 递归版
int fib(int n) {
    if (n <= 1) return n;
    return fib(n-1) + fib(n-2);
}

// 非递归版
int fib_iter(int n) {
    if (n <= 1) return n;
    int a = 0, b = 1, c;
    for (int i = 2; i <= n; i++) {
        c = a + b; a = b; b = c;
    }
    return b;
}
```

#### 3. 通用方法：栈模拟递归（万能）
**核心思想**：用**显式栈**保存系统调用栈的关键信息：
- 函数参数
- 局部变量
- **返回地址/执行阶段（tag）**
**通用步骤**：
1. 定义栈节点结构（含参数、局部变量、执行标记）
2. 初始化栈，压入**初始调用**
3. 循环处理栈：
   - 出栈，恢复现场
   - 若为**基线条件**：计算结果并保存
   - 若为**递归阶段**：
     - 压入当前状态（标记为“待返回”）
     - 压入子问题（按逆序，保证执行顺序）
   - 若为**结果返回阶段**：合并子结果

**示例：二叉树中序遍历（递归→非递归）**
```cpp
struct TreeNode { int val; TreeNode *left, *right; };

// 递归版
void inorder_recur(TreeNode* root) {
    if (!root) return;
    inorder_recur(root->left);
    cout << root->val;
    inorder_recur(root->right);
}

// 非递归版（栈模拟）
void inorder_iter(TreeNode* root) {
    stack<pair<TreeNode*, bool>> st; // (节点, 是否已访问)
    if (root) st.push({root, false});
    while (!st.empty()) {
        auto [node, visited] = st.top(); st.pop(); // 出栈
        if (!node) continue;
        if (!visited) {
            // 逆序压栈：右 → 根(标记已访问) → 左
            st.push({node->right, false});
            st.push({node, true});
            st.push({node->left, false});
        } else {
            cout << node->val; // 访问节点
        }
    }
}
```

**示例：归并排序（递归→非递归）**
```cpp
// 递归版（核心）
void mergeSort(vector<int>& a, int l, int r) {
    if (l >= r) return;
    int mid = (l+r)/2;
    mergeSort(a, l, mid);
    mergeSort(a, mid+1, r);
    merge(a, l, mid, r);
}

// 非递归版（栈模拟）
void mergeSort_iter(vector<int>& a) {
    int n = a.size();
    stack<tuple<int, int, bool>> st; // (l, r, 是否处理)
    st.push({0, n-1, false});
    while (!st.empty()) {
        auto [l, r, done] = st.top(); st.pop(); // 出栈
        if (l >= r) continue;
        if (!done) {
            int mid = (l+r)/2;
            st.push({l, r, true}); // 标记待合并
            st.push({mid+1, r, false}); // 右子问题
            st.push({l, mid, false}); // 左子问题
        } else {
            int mid = (l+r)/2;
            merge(a, l, mid, r); // 执行合并
        }
    }
}
```

#### 4. 其他方法（特定场景）
- **动态规划（DP）**：将递归式转为递推式（如斐波那契、爬楼梯）
- **记忆化搜索**：用哈希表缓存结果，避免重复递归（本质仍是递归，但效率接近迭代）
- **状态机**：将递归逻辑映射为状态转移（适合复杂控制流）

### 三、方法对比与选择
| 方法 | 适用场景 | 复杂度 | 代码量 | 通用性 |
|:--- |:--- |:--- |:--- |:--- |
| **尾递归→迭代** | 尾递归函数 | O(n) | 极少 | 低 |
| **单向递归→迭代** | 线性递推（斐波那契、阶乘） | O(n) | 少 | 中 |
| **栈模拟** | 所有递归（二叉树、归并、DFS） | O(n log n) / O(n) | 多 | **最高** |
| **动态规划** | 最优子结构、无后效性 | 视问题而定 | 中 | 中 |

### 四、关键技巧
1. **先写递归**：理清基线条件、递归分解、结果合并
2. **识别类型**：尾递归/单向递归优先用迭代；否则用栈
3. **栈节点设计**：只保存**必要信息**（参数、返回标记），减少空间
4. **执行顺序**：栈是**后进先出**，压入子问题时需**逆序**

---

### 五、总结
- **任何递归都可转为非递归**（图灵完备性）
- **简单递归**：直接迭代（尾递归/单向）
- **复杂递归**：**栈模拟**是万能解法
- **核心**：显式栈替代系统调用栈，手动管理执行状态
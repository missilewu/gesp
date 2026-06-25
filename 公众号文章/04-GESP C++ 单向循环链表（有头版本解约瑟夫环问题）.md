
# GESP C++ 单向循环链表（有头版本解约瑟夫环问题）

🚩 **定义：** 在单循环链表中，最后一个节点的 `next` 指针不指向 `nullptr`，而是指向头节点（有头版本），形成一个环。

单向循环链表可以看作是单向链表的一种“变形”，它们的核心区别在于链表的**结尾处理方式**，这直接导致了两者在操作逻辑和应用场景上的不同。在GESP考试中（5级以上），它在处理约瑟夫环（Josephus Problem）等经典问题时非常有用。

简单来说，单向链表是一条有明确起点和终点的“单行道”，而单向循环链表则是一个“环形跑道”。
下面是它们的主要区别：

## 🔗 结构上的根本区别

*   **单向链表 (Singly Linked List):**
    *   它是一条线性的链。
    *   最后一个节点（尾节点）的 `next` 指针指向 `nullptr`，表示链表的结束。
    *   结构示意：`头节点 -> 节点A -> 节点B -> NULL`

*   **单向循环链表 (Singly Circular Linked List):**
    *   它形成一个环。
    *   最后一个节点的 `next` 指针不再指向 `nullptr`，而是指回链表的**头节点**（或第一个有效节点），使整个链表首尾相连。
    *   结构示意：`头节点(head) -> 节点A -> 节点B -> (指回头节点head)`

## ⚙️ 操作逻辑的差异

这个结构上的差异，导致了在具体操作上的一些关键不同：

| 特性         | 单向链表                                                        | 单向循环链表                                      |
| :--------- | :---------------------------------------------------------- | :------------------------------------------ |
| **遍历终止条件** | 当指针遇到 `nullptr` 时停止。                                        | 当指针再次回到**头节点**时停止，否则会变成无限循环。                |
| **空链表判断**  | 头指针为 `nullptr` (不带头结点时) 或 `head->next` 为 `nullptr` (带头结点时)。 | 头结点的 `next` 指针指向它自己 (`head->next == head`)。 |
| **节点可达性**  | 从任意节点出发，只能访问其后续节点，无法回头。                                     | 从**任意节点**出发，都可以遍历到链表中的所有其他节点。               |

## 🎯 应用场景的侧重

*   **单向链表**：适用于数据有明确开始和结束的场景，例如实现一个普通的待办事项列表、一个栈或一个队列。它的逻辑简单，是更基础的数据结构。
*   **单向循环链表**：专为需要循环处理的场景设计，例如操作系统的**时间片轮转调度**、音乐播放器的**循环播放列表**、以及经典的**约瑟夫环**问题。


## 📃代码详解

从上文的差异，单向循环链表可以从单向链表演化过来，通过一个简单的指针修改（将尾节点的 `next` 从 `nullptr` 改为指向头节点`head`），就获得了循环遍历的能力，使其在处理周期性、循环类的算法比单向链表更加高效和方便。😅 重要的事情说了三遍了.....

注意下面代码注释中的【核心差异】

### 1. 单循环链表的定义

```cpp
#include <iostream>
using namespace std;

// --- 0. 定义结构体 ---
struct Node {
    int data;
    Node* next;
    // 构造函数
    Node(int val = 0) : data(val), next(nullptr) {}
};

typedef Node* CircularLinkedList; 

// --- 1. 初始化链表 (创建头结点) ---
CircularLinkedList initList() {
    Node* head = new Node(0); 
    // 【核心差异】循环链表的头结点 next 指向自己，形成空环
    head->next = head; 
    return head;
}
```

---

### 2. 核心操作实现

#### A. 遍历 (Traversal)

由于是循环链表，终止条件不是 `p != nullptr`，而是 `p != head`。

```cpp
// --- 2. 遍历 (Traversal) ---
void traverse(CircularLinkedList head) {
    if (head == nullptr) {
        cout << "Error: List header is missing." << endl;
        return;
    }

    // 【核心差异】如果头结点的下一个是自己，说明链表为空
    if (head->next == head) {
        cout << "List is empty." << endl;
        return;
    }

    Node* curr = head->next; // 从第一个有效节点开始
    
    cout << "List: ";
    // 【核心差异】终止条件是回到头结点，而不是遇到 nullptr
    while (curr != head) {
        cout << curr->data << " -> ";
        curr = curr->next;
    }
    cout << "(Head)" << endl; // 表示回到了头
}
```

#### B. 搜索 (Search)

查找值为 `val` 的节点，返回该节点指针，若未找到返回 `nullptr`。

```cpp
// --- 3. 搜索 (Search) ---
Node* search(CircularLinkedList head, int target) {
    if (head == nullptr) return nullptr;
    
    // 如果是空表，直接返回
    if (head->next == head) return nullptr;

    Node* curr = head->next; 
    // 【核心差异】只要没转回一圈回到头结点，就继续找
    while (curr != head) {
        if (curr->data == target) {
            return curr;
        }
        curr = curr->next;
    }
    return nullptr;
}
```

#### C. 插入 (Insertion)

_注意：在循环链表中，插入操作需要小心维护环的闭合_  

**头插法**

```cpp
// --- 4. 头部插入 (Insert at Head) ---
// 逻辑与单向链表几乎一致，因为 head->next 始终存在（至少指向 head）
void insertAtHead(CircularLinkedList head, int val) {
    if (head == nullptr) return;

    Node* newNode = new Node(val);
    // 1. 新节点指向原来的第一个有效节点（如果是空表，这里指向 head）
    newNode->next = head->next;
    // 2. 【核心差异】新节点必须指回头结点，维持循环
    head->next = newNode;
}
```

**尾插法（推荐，常用于构建队列或约瑟夫环）：**

```cpp
// --- 5. 尾部插入 (Insert at Tail) ---
void insertAtTail(CircularLinkedList head, int val) {
    if (head == nullptr) return;

    Node* newNode = new Node(val);
    Node* curr = head; 
    
    // 【核心差异】找尾部的条件：next 指向 head 的节点是尾节点
    while (curr->next != head) {
        curr = curr->next;
    }
    
    // 插入新节点
    curr->next = newNode;
    // 【核心差异】新节点必须指回头结点，维持循环
    newNode->next = head;
}
```

**指定位置插入（在第 pos 个位置后插入，pos从0开始计数，0表示插在头节点后）：**

```cpp
// --- 6. 指定位置插入 (Insert after a specific node) ---
void insertAfter(Node* prevNode, int val) {
    if (prevNode == nullptr) {
        cout << "Error: Previous node cannot be nullptr." << endl;
        return;
    }
    Node* newNode = new Node(val);
    // 逻辑通用，无论 prevNode 是中间节点还是尾节点都适用
    newNode->next = prevNode->next;
    prevNode->next = newNode;
}
```

#### D. 删除 (Deletion)

删除值为 `val` 的第一个节点。需要找到待删除节点的前驱节点。

```cpp
// --- 7. 删除节点 (Delete by value) ---
bool deleteNode(CircularLinkedList head, int target) {
    if (head == nullptr) return false;
    // 判空
    if (head->next == head) return false;

    Node* curr = head; 
    
    // 寻找前驱节点
    // 【核心差异】循环条件：只要没回到头，且下一个不是目标，就继续找
    while (curr->next != head && curr->next->data != target) {
        curr = curr->next;
    }

    // 如果找到了目标节点
    if (curr->next != head) {
        Node* temp = curr->next;
        curr->next = curr->next->next; // 跳过目标节点
        delete temp;
        return true;
    }

    return false; 
}
```

#### E. 反转 (Reverse)

单循环链表的反转与单链表类似，但最后需要将新的尾节点（原头节点的后继）重新连回头节点。  
**注意**：对于带头节点的循环链表，反转的是数据节点部分，头节点 `head` 保持不变，始终作为入口。

```cpp
// --- 8. 反转链表 (Reverse) ---
// 逻辑基本不变，但要注意最后要把尾节点的 next 重新连回 head
void reverseList(CircularLinkedList head) {
    if (head == nullptr) return;
    if (head->next == head) return; // 空表不用反转

    Node* prev = head;       // 这里的 prev 初始化为 head，方便最后连接
    Node* curr = head->next; 
    Node* nextTemp = nullptr;

    // 注意：这里我们实际上是把 head 当作反转后的“新尾节点”的前驱来处理
    // 或者更简单的理解：把 head->next 这一串摘下来反转，最后再把尾巴连回 head
    
    // 为了逻辑清晰，我们采用标准反转，最后修正头尾关系
    prev = nullptr; 
    Node* tail = head->next; // 记录原来的第一个节点，它反转后会变成最后一个节点
    
    while (curr != head) {
        nextTemp = curr->next;
        curr->next = prev;
        prev = curr;
        curr = nextTemp;
    }
    
    // 此时：
    // prev 指向原来的最后一个节点（现在是新的第一个有效节点）
    // tail 指向原来的第一个节点（现在是新的最后一个有效节点）
    
    head->next = prev;   // 头结点指向新的第一个节点
    tail->next = head;   // 【核心差异】新的最后一个节点必须指回头结点
}
```

#### F. 清空销毁整个链表

```cpp
// --- 9. 释放链表内存 (Clear) ---
void clearList(CircularLinkedList& head) {
    if (head == nullptr) return;
    
    Node* curr = head->next; // 从第一个有效节点开始
    while (curr != head) {   // 遇到头结点就停止
        Node* temp = curr;
        curr = curr->next;
        delete temp;
    }
    
    delete head; // 最后删除头结点
    head = nullptr;
}
```

---

### 3. 完整测试样例 (Main Function)

将上述代码组合在一起进行测试：

```cpp
// ================= 主函数测试 =================
int main() {
    CircularLinkedList myList = initList(); 

    cout << "--- 1. 头部插入测试 (5, 4, 3) ---" << endl;
    insertAtHead(myList, 3);
    insertAtHead(myList, 4);
    insertAtHead(myList, 5);
    traverse(myList); 

    cout << "\n--- 2. 尾部插入测试 (追加 6, 7) ---" << endl;
    insertAtTail(myList, 6);
    insertAtTail(myList, 7);
    traverse(myList); 

    cout << "\n--- 3. 搜索测试 (查找 4) ---" << endl;
    Node* found = search(myList, 4);
    if (found) cout << "Found 4: " << found->data << endl;

    cout << "\n--- 4. 插入测试 (在 4 后面插入 100) ---" << endl;
    Node* node4 = search(myList, 4);
    if (node4) {
        insertAfter(node4, 100);
        traverse(myList); 
    }

    cout << "\n--- 5. 删除测试 (删除头节点 5) ---" << endl;
    if (deleteNode(myList, 5)) {
        cout << "Deleted 5 successfully." << endl;
        traverse(myList); 
    } 

    cout << "\n--- 6. 反转测试 ---" << endl;
    reverseList(myList);
    cout << "After Reverse:  ";
    traverse(myList); 

    cout << "\n--- 7. 清理内存 ---" << endl;
    clearList(myList);
    
    return 0;
}
```

### 4. 编码练习时的注意事项

1. **空表判断**：在循环链表中，`list->head->next == list->head` 是判断空表的唯一标准。
2. **死循环风险**：遍历时的 `do-while` 或 `while` 条件必须严格检查是否回到了 `head`，否则会导致无限循环（TLE）。
3. **内存泄漏**：虽然在线评测系统（OJ）在程序结束后会回收内存，但在删除节点时务必 `delete`，养成良好的编程习惯。
4. **指针操作顺序**：在插入和删除时，一定要先连接新指针，再断开旧指针，防止链表断裂丢失后续节点。这个也是选择和判断题的高频考点。

---
## 应用：经典约瑟夫环问题

约瑟夫环问题（Josephus Problem）是一个著名的数学和计算机科学算法问题，也被称为“约瑟夫斯置换”或“丢手绢问题”。它描述了一个经典的淘汰游戏，并探讨如何在这种规则下成为最后的幸存者。

### 📜 问题描述

问题的标准形式如下：

1.  **N** 个人围成一个圆圈。
2.  从某个人（通常是第1个人）开始，按顺时针方向依次报数，从1报到 **M**。
3.  报到 **M** 的那个人被淘汰出局。
4.  然后从他下一个人开始，重新从1报数，再次报到 **M** 的人被淘汰。
5.  这个过程不断重复，直到圆圈中只剩下最后一个人，这个人就是幸存者。

**核心问题：** 给定总人数 N 和报数 M，如何确定初始位置，才能让自己成为最后的幸存者？

### 💻 解决方法

解决约瑟夫环问题主要有两种思路：模拟法和数学法。其中模拟法 (Simulation)最直观，就是按照游戏规则，一步步地模拟整个淘汰过程。

*   **数据结构：** 通常使用**循环链表**（Circular Linked List）或数组来模拟这个“环”。每个节点代表一个人。
*   **过程：** 从起点开始遍历链表，数到 M 时，将对应节点从链表中删除。然后从下一个节点继续报数，直到链表中只剩下一个节点。
*   **优点：** 逻辑清晰，易于理解和实现。
*   **缺点：** 效率较低。时间复杂度为 O(N×M)，当 N 和 M 很大时，计算会很慢。

数学法我们在算法部分再深入的学习，这里重点是练习单向循环链表的应用。

### 🧩 编码思路

使用单向循环链表解决约瑟夫环问题，其核心优势在于链表的环形结构能够完美模拟“围坐一圈”的场景，而链表的动态删除操作则能高效地模拟“出列”的过程。

1. **构建环**：使用 `insertAtTail` 函数，将 `n` 个人（编号 1 到 n）依次加入链表，形成一个包含 n 个节点的循环链表。
2. **定位起点**：使用 `search` 函数找到编号为 `k` 的节点，作为报数的起始位置。
3. **循环报数与出列**：
    - 从起始节点开始，我们需要找到从当前节点算起的第 `m` 个节点。
    - 由于我们的 `deleteNode` 函数是按“值”删除，所以我们需要先找到这个待删除节点的值。
    - 找到后，调用 `deleteNode` 将其删除，并打印其编号。
    - 重复此过程，直到链表中只剩下一个节点

### 💻 代码示例 (C++)

下面是基于上述思路的 C++ 代码实现：

```cpp
// ================= 约瑟夫环问题求解 =================

/**
 * 使用带头结点的循环链表解决约瑟夫环问题
 * @param n 总人数
 * @param k 从第k个人开始报数
 * @param m 报数到m的人出列
 */
void solveJosephus(int n, int k, int m) {
    if (n <= 0 || k <= 0 || m <= 0 || k > n) {
        cout << "Invalid input parameters." << endl;
        return;
    }

    // 1. 初始化并构建包含 n 个人的循环链表
    CircularLinkedList list = initList();
    for (int i = 1; i <= n; ++i) {
        insertAtTail(list, i);
    }
    
    cout << "初始状态: ";
    traverse(list);

    // 2. 找到报数起点 (第 k 个人)
    Node* current = search(list, k);
    if (!current) {
        cout << "Error finding start node." << endl;
        clearList(list);
        return;
    }

    cout << "从编号 " << k << " 开始报数, 每次数到 " << m << " 出列." << endl;
    cout << "出列顺序: ";

    // 3. 循环报数并删除节点
    // 当链表中不止一个有效节点时循环
    while (list->next != list) { 
        // 从 current 开始，找到第 m 个节点
        // 我们需要找到待删除节点的值
        int target_val = current->data;
        for (int i = 1; i < m; ++i) {
            current = current->next;
            // 如果 current 走到头结点，就绕回到第一个有效节点
            if (current == list) {
                current = list->next;
            }
            target_val = current->data;
        }

        // 打印并删除找到的节点
        cout << target_val << " ";
        deleteNode(list, target_val);

        // 确定下一轮报数的起点
        // 如果被删除的节点是最后一个节点，下一轮起点是链表的第一个节点
        // 否则，起点是被删除节点的下一个节点
        if (list->next == list) { // 链表已空，跳出循环
             break;
        }
        
        // 寻找下一轮的起点：即被删除节点的下一个节点
        // 我们先找到被删除节点的前一个节点
        Node* temp = list;
        while (temp->next != list && temp->next->data != target_val) {
             temp = temp->next;
        }
        
        // 如果找到了前驱节点（说明target_val确实存在），则下一轮起点是其next
        // 否则，说明删除的是最后一个节点，起点重置为第一个节点
        if (temp->next != list) {
             current = temp->next;
        } else {
             current = list->next;
        }
    }
    cout << endl;
    
    // 4. 清理内存
    clearList(list);
}

// ================= 主函数测试 =================
int main() {
    // 示例：5个人，从第1个开始，数到2出列
    // 预期出列顺序: 2 -> 4 -> 1 -> 5 -> 3
    solveJosephus(5, 1, 2);

    cout << "\n--- 另一个测试 ---" << endl;
    // 示例：7个人，从第2个开始，数到3出列
    solveJosephus(7, 2, 3);

    return 0;
}
```

### ⏱️ 复杂度分析

*   **时间复杂度**: O(n × m)。我们需要进行 n 轮删除操作，每一轮都需要移动 m 次指针来找到待删除的节点。
*   **空间复杂度**: O(n)。需要创建 n 个节点来构建链表。




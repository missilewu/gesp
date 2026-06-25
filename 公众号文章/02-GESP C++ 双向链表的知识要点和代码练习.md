# GESP C++ 双向链表的知识要点和代码练习

大家好！今天我们一起学习 **双向链表（Doubly Linked List）**。

在GESP考试中，虽然可以用C++ STL提供的`std::list`，但理解底层原理对于解决复杂问题（如模拟、动态维护序列、内存管理优化），以及回答选择和判断题至关重要。很多时候，我们也需要手写链表（或借鉴其思维模式）来实现特定的功能或优化性能。

---

## 一、什么是双向链表？

### 1. 概念对比
*   **数组（Array）**：内存连续，支持$O(1)$随机访问，但插入/删除需要移动大量元素，复杂度$O(n)$。
*   **单向链表（Singly Linked List）**：每个节点只存数据和**下一个**节点的指针。插入/删除快，但只能向后遍历，无法快速找到前驱节点（删除某节点时需要从头遍历找前驱）。
*   **双向链表（Doubly Linked List）**：每个节点包含三部分：
    1.  **数据域（data）**：存储实际数值。
    2.  **前驱指针（prev）**：指向**上一个**节点。
    3.  **后继指针（next）**：指向**下一个**节点。

### 2. 核心优势
*   **双向遍历**：可以从头到尾，也可以从尾到头。
*   $O(1)$ **删除/插入**：只要拿到了当前节点的指针，就可以在$O(1)$时间内完成插入或删除操作（不需要像单链表那样寻找前驱）。
*   **灵活性**：非常适合需要频繁在序列中间进行增删改查的场景。

---

## 二、原理与内存结构

### 1. 节点结构图示
假设我们有一个节点 `P`：
```text
      prev       data       next
+---------------+-------+---------------+
|  (指向上一节点) |  值   | (指向下一节点) |
+---------------+-------+---------------+
       ^                       ^
       |                       |
    [前一个节点]            [后一个节点]
```

### 2. 特殊处理：哨兵节点（Dummy Head/Tail）

在编程中，为了简化边界判断（比如判断是否为空、处理头插/尾插），我们通常会引入**哨兵节点**（也叫虚拟头节点）。在双向链表中，采用无头（headless）的方式实现更加复杂，一般不推荐使用，但是需要了解其变化，以应对考试。回到哨兵节点上，有以下的结构方式，
#### 哨兵头节点

- **位置**：位于链表的绝对头部。
- **数据**：不存储任何有效业务数据。
- **`prev` 指针**：通常指向 `nullptr`。
- **`next` 指针**：永远指向链表的**第一个实际数据节点**。如果链表为空，则指向哨兵尾节点。

#### 哨兵尾节点

- **位置**：位于链表的绝对尾部。
- **数据**：不存储任何有效业务数据。
- **`prev` 指针**：永远指向链表的**最后一个实际数据节点**。如果链表为空，则指向哨兵头节点。
- **`next` 指针**：通常指向 `nullptr`。

### 哨兵结构的优势

- **统一空链表处理**：
    - 即使链表中没有数据，`Dummy Head` 和 `Dummy Tail` 依然存在且相互连接。
    - 不需要专门判断 `head == nullptr` 的情况。
- **简化插入/删除逻辑**：
    - **插入到头部**：实际上是在 `Dummy Head` 和原来的第一个节点之间插入，不需要修改 `head` 指针本身。
    - **删除最后一个节点**：只需要修改 `Dummy Tail` 的 `prev` 指针，不需要判断是否要把 `head` 置空。

### 3. 核心操作原理（指针变换）

这是考试中最容易出错的地方，请务必记住 **“先连后断”** 的原则，防止链表断裂丢失。

**场景A：在节点 `p` 之后插入新节点 `x`**
原顺序：`p` <-> `p->next`
目标顺序：`p` <-> `x` <-> `p->next`

**步骤（共4步，顺序很重要）：**
1.  `x->next = p->next;`  // x的后继指向p原来的后继
2.  `x->prev = p;`        // x的前驱指向p
3.  `p->next->prev = x;`  // p原来后继的前驱指向x (**注意：如果p是尾节点，这步需小心，但在有哨兵或检查非空时安全**)
4.  `p->next = x;`        // p的后继指向x

**场景B：删除节点 `p`**
原顺序：`p->prev` <-> `p` <-> `p->next`
目标顺序：`p->prev` <-> `p->next`

**步骤（共2步）：**
1.  `p->prev->next = p->next;` // 前驱的后继指向后继
2.  `p->next->prev = p->prev;` // 后继的前驱指向前驱
3.  `delete p;` // 释放内存（竞赛中若使用静态数组模拟则无需此步）

---
## 三、代码练习（有头指针和尾指针）

以下是有头指针和尾指针的 **双向链表（Doubly Linked List）** 的C风格定义及核心操作代码。编码过程中，脑子中要有一个图，能够想象和模拟出插入和删除的变化过程。如果想象不出来，可以在草稿纸上画一画，逐步的在大脑中建立出一个形象的图像。

1.  **空指针异常**：
    *   在使用指针版时，务必检查 `node->next` 或 `node->prev` 是否为 `nullptr`。
    *   使用**哨兵节点**的结构时，可以避免空指针异常，让 `head->prev` 和 `tail->next` 指向合法的哨兵或形成环，这样就不需要判空了。
2.  **指针断开顺序**：
    *   插入时，如果先执行 `p->next = x`，那么原本的 `p->next` 就丢了，再也找不到后面的节点了。**一定要先保存旧连接，再修改新连接。**
    *   口诀：**“新人先认亲，老人再改口”**（新节点先连好前后，旧节点再修改指向新节点）。


### 1. 节点与链表定义

双链表的每个节点包含三个部分：数据域、指向前驱节点的指针、指向后继节点的指针。注意：这个版本是没有哨兵节点的结构。

```cpp
#include <iostream>   // 用于 cout, cerr, endl
#include <cstdlib>    // 用于 exit()

/**
 * @brief 双向链表节点结构体
 * 
 * 每个节点包含数据域和两个指针：指向前驱节点和后继节点。
 */
struct DNode {
    int data;       // 数据域：存储整数值
    DNode* prev;    // 前驱指针：指向链表中的上一个节点
    DNode* next;    // 后继指针：指向链表中的下一个节点
};

/**
 * @brief 双向链表控制结构体
 * 
 * 用于维护链表的元数据：头指针、尾指针和当前节点数量。
 * 维护 tail 指针可以让尾部插入操作的时间复杂度降为 O(1)。
 */
struct DLinkedList {
    DNode* head;    // 头指针：指向第一个节点
    DNode* tail;    // 尾指针：指向最后一个节点
    int size;       // 大小：记录链表中节点的总数
};

/**
 * @brief 初始化链表
 * 
 * 将链表的状态重置为空。
 * @param list 指向链表结构体的指针
 */
void initList(DLinkedList* list) {
    list->head = nullptr; // 头指针置空
    list->tail = nullptr; // 尾指针置空
    list->size = 0;       // 节点数清零
}

/**
 * @brief 创建新节点
 * 
 * 使用 C++ 的 new 操作符在堆上分配内存，并初始化成员变量。
 * @param value 要存储的数据值
 * @return 指向新创建节点的指针
 */
DNode* createNode(int value) {
    // 使用 new(std::nothrow) 分配内存，自动调用构造函数（如果有）且返回正确类型的指针
    DNode* newNode = new(std::nothrow) DNode;
    
    // 检查内存分配是否成功（注意前面使用的是 nothrow 版本的new）
    // new 默认失败会抛异常，若采用new DNode 需严格检查可捕获 std::bad_alloc
    if (!newNode) {
        cerr << "Memory allocation failed" << endl; // 使用 cerr 输出错误信息
        exit(1); // 异常退出
    }
    
    newNode->data = value;
    newNode->prev = nullptr; // 初始化前驱指针为空
    newNode->next = nullptr; // 初始化后继指针为空
    
    return newNode;
}
```

---

### 2. 遍历操作 (Traversal)

从头到尾打印链表内容。

```cpp
/**
 * @brief 正向遍历链表
 * 
 * 从头节点开始，沿着 next 指针逐个访问直到末尾。
 * @param list 指向链表结构体的指针
 */
void traverseForward(DLinkedList* list) {
    DNode* curr = list->head; // 从头节点开始
    cout << "List (Forward): ";
    
    // 当当前节点不为空时循环
    while (curr != nullptr) {
        cout << curr->data << " "; // 输出数据，后跟空格
        curr = curr->next;         // 移动到下一个节点
    }
    cout << endl; // 换行
}

/**
 * @brief 反向遍历链表
 * 
 * 双链表的优势：可以从尾节点开始，沿着 prev 指针向前访问。
 * @param list 指向链表结构体的指针
 */
void traverseBackward(DLinkedList* list) {
    DNode* curr = list->tail; // 从尾节点开始
    cout << "List (Backward): ";
    
    // 当当前节点不为空时循环
    while (curr != nullptr) {
        cout << curr->data << " ";
        curr = curr->prev;    // 移动到前一个节点
    }
    cout << endl;
}
```

---

### 3. 搜索操作 (Search)

查找特定值的节点，返回节点指针，若未找到返回 `nullptr`。

```cpp
/**
 * @brief 搜索节点
 * 
 * 线性查找值为 value 的节点。
 * @param list 指向链表结构体的指针
 * @param value 要查找的值
 * @return 找到则返回节点指针，否则返回 nullptr
 */
DNode* search(DLinkedList* list, int value) {
    DNode* curr = list->head;
    
    while (curr != nullptr) {
        if (curr->data == value) {
            return curr; // 找到目标，返回指针
        }
        curr = curr->next;
    }
    
    return nullptr; // 遍历结束未找到
}
```

---

### 4. 插入操作 (Insertion)

这是竞赛中最常考的细节，需要处理头插、尾插和中间插入，并正确维护 `prev` 和 `next` 指针。

#### A. 头部插入

```cpp
/**
 * @brief 头插法插入节点
 * 
 * 在链表头部插入新节点。如果链表为空，新节点既是头也是尾。
 * 时间复杂度: O(1)
 * @param list 指向链表结构体的指针
 * @param value 要插入的值
 */
void insertAtHead(DLinkedList* list, int value) {
    DNode* newNode = createNode(value);
    
    if (list->head == nullptr) {
        // 情况 1: 链表为空
        // 新节点既是头也是尾
        list->head = newNode;
        list->tail = newNode;
    } else {
        // 情况 2: 链表非空
        // 1. 新节点的 next 指向原来的头节点
        newNode->next = list->head;
        // 2. 原来头节点的 prev 指向新节点
        list->head->prev = newNode;
        // 3. 更新头指针指向新节点
        list->head = newNode;
    }
    
    list->size++; // 更新大小
}
```

#### B. 尾部插入

```cpp
/**
 * @brief 尾插法插入节点
 * 
 * 在链表尾部插入新节点。利用 tail 指针实现 O(1) 插入。
 * @param list 指向链表结构体的指针
 * @param value 要插入的值
 */
void insertAtTail(DLinkedList* list, int value) {
    DNode* newNode = createNode(value);
    
    if (list->tail == nullptr) {
        // 情况 1: 链表为空
        // 新节点既是头也是尾
        list->head = newNode;
        list->tail = newNode;
    } else {
        // 情况 2: 链表非空
        // 1. 新节点的 prev 指向原来的尾节点
        newNode->prev = list->tail;
        // 2. 原来尾节点的 next 指向新节点
        list->tail->next = newNode;
        // 3. 更新尾指针指向新节点
        list->tail = newNode;
    }
    
    list->size++;
}
```

#### C. 在指定节点后插入 (通用插入)

_注意：在竞赛中，通常是在找到某个节点 `pos` 后，在其后面插入新节点。_

```cpp
/**
 * @brief 在指定节点后插入
 * 
 * 在节点 pos 之后插入新节点。
 * @param list 指向链表结构体的指针
 * @param pos 参考节点指针，新节点将插在它后面
 * @param value 要插入的值
 */
void insertAfter(DLinkedList* list, DNode* pos, int value) {
    // 边界检查：如果参考节点为空，无法执行插入
    if (pos == nullptr) {
        return; 
    } 
    
    DNode* newNode = createNode(value);
    
	// 注意：核心指针操作 (先连新节点，再断旧连接)
    
    // 1. 新节点的 next 指向 pos 的下一个节点
    newNode->next = pos->next;
    // 2. 新节点的 prev 指向 pos
    newNode->prev = pos;
    
    // 3. 如果 pos 不是最后一个节点，需要修改 pos 原后继节点的 prev 指针
    if (pos->next != nullptr) {
        pos->next->prev = newNode;
    } else {
        // 特殊情况：如果 pos 是尾节点，新节点将成为新的尾节点
        list->tail = newNode;
    }
    
    // 4. 最后将 pos 的 next 指向新节点
    pos->next = newNode;
    
    list->size++;
}
```

---

### 5. 删除操作 (Deletion)

删除节点时需要特别注意断开前后指针的连接，并释放内存。

#### A. 删除指定值的第一个匹配节点

```c
/**
 * @brief 删除指定值的节点
 * 
 * 查找并删除第一个匹配值的节点，同时修复前后指针连接。
 * @param list 指向链表结构体的指针
 * @param value 要删除的值
 * @return 删除成功返回 true，未找到返回 false
 */
bool deleteNode(DLinkedList* list, int value) {
    // 先搜索节点
    DNode* curr = search(list, value);
    
    // 如果没找到，直接返回 false
    if (curr == nullptr) {
        return false; 
    }
    
    // --- 修复前驱链接 ---
    if (curr->prev != nullptr) {
        // 如果不是头节点，让前驱节点的 next 跳过 curr，指向 curr 的后继
        curr->prev->next = curr->next;
    } else {
        // 如果是头节点，更新链表的 head 指针
        list->head = curr->next;
    }
    
    // --- 修复后继链接 ---
    if (curr->next != nullptr) {
        // 如果不是尾节点，让后继节点的 prev 跳过 curr，指向 curr 的前驱
        curr->next->prev = curr->prev;
    } else {
        // 如果是尾节点，更新链表的 tail 指针
        list->tail = curr->prev;
    }
    
    // 释放内存 (使用 delete 对应之前的 new)
    delete curr;
    
    list->size--;
    return true;
}
```

#### B. 销毁整个链表

```c
/**
 * @brief 清空链表
 * 
 * 遍历并删除所有节点，释放内存，重置链表状态。
 * @param list 指向链表结构体的指针
 */
void clearList(DLinkedList* list) {
    DNode* curr = list->head;
    
    while (curr != nullptr) {
        DNode* next = curr->next; // 先保存下一个节点的地址，防止丢失
        delete curr;              // 删除当前节点
        curr = next;              // 移动到下一个
    }
    
    // 重置元数据
    list->head = nullptr;
    list->tail = nullptr;
    list->size = 0;
}
```

---

### 6. 链表反转 (Reverse)

双链表的反转可以通过交换每个节点的 `prev` 和 `next` 指针来实现，最后交换头尾指针。时间复杂度 $O(N)$，空间复杂度 $O(1)$。

```cpp
/**
 * @brief 反转链表
 * 
 * 原地反转双向链表。交换每个节点的 prev 和 next 指针，并交换头尾指针。
 * @param list 指向链表结构体的指针
 */
void reverseList(DLinkedList* list) {
    // 空链表无需反转
    if (list->head == nullptr) {
        return;
    }
    
    DNode* curr = list->head;
    DNode* temp = nullptr;
    
    // 1. 交换链表的头尾指针记录
    temp = list->head;
    list->head = list->tail;
    list->tail = temp;
    
    // 2. 遍历链表，交换每个节点的指针方向
    while (curr != nullptr) {
        // 交换 prev 和 next
        temp = curr->prev;
        curr->prev = curr->next;
        curr->next = temp;
        
        // 移动到下一个节点
        // 注意：因为刚才交换了指针，现在的 curr->prev 实际上是指向原来的“下一个”节点
        curr = curr->prev; 
    }
}

```

---

### 7. 完整测试示例 (Main Function)

```cpp
int main() {
    // 声明链表对象
    DLinkedList myList;
    initList(&myList); // 初始化
    
    cout << "--- Testing Insertions ---" << endl;
    // 测试尾插: 10, 20
    insertAtTail(&myList, 10);
    insertAtTail(&myList, 20);
    // 测试头插: 5 -> 10 -> 20
    insertAtHead(&myList, 5);
    
    cout << "Initial List:" << endl;
    traverseForward(&myList); // 预期: 5 10 20
    traverseBackward(&myList); // 预期: 20 10 5
    
    cout << "\n--- Testing Search and InsertAfter ---" << endl;
    // 测试搜索
    DNode* found = search(&myList, 10);
    if (found) {
        cout << "Found 10, inserting 15 after it." << endl;
        insertAfter(&myList, found, 15); 
        // 预期链表: 5 <-> 10 <-> 15 <-> 20
    }
    
    traverseForward(&myList);
    
    cout << "\n--- Testing Deletion ---" << endl;
    cout << "Deleting 5 (Head)..." << endl;
    deleteNode(&myList, 5); // 删除头节点
    traverseForward(&myList); // 预期: 10 15 20
    
    cout << "Deleting 15 (Middle node)..." << endl;
    deleteNode(&myList, 15);
    traverseForward(&myList);  // 预期: 5 10 20

    cout << "Inserting 15 after 10 again." << endl;
    insertAfter(&myList, found, 15);  
	traverseForward(&myList); // 预期链表: 5 <-> 10 <-> 15 <-> 20
    
    cout << "Deleting 20 (Tail)..." << endl;
    deleteNode(&myList, 20); // 删除尾节点
    traverseForward(&myList); // 预期: 10 15
    
    cout << "\n--- Testing Reverse ---" << endl;
    cout << "Reversing list..." << endl;
    reverseList(&myList);
    
    cout << "Forward traversal after reverse:" << endl;
    traverseForward(&myList);   // 预期: 15 10
    
    cout << "Backward traversal after reverse:" << endl;
    traverseBackward(&myList);  // 预期: 10 15
    
    cout << "\n--- Cleaning Up ---" << endl;
    // 清理内存，防止泄漏
    clearList(&myList);
    cout << "List cleared successfully." << endl;
    
    return 0;
}
```


---
## 四、代码练习（有头指针、无尾指针）

作为对比和学习，下面是一个去掉了 `tail` 指针及相关逻辑的**无尾双向链表**实现（由千问改写）。同样的，通过不断地编码练习，熟练掌握细节的逻辑，可以轻松的应对GESP中的选择和判断题。

### 主要变更说明：
1.  **结构体变更**：`DLinkedList` 中移除了 `DNode* tail` 成员。
2.  **初始化**：`initList` 不再初始化 `tail`。
3.  **遍历反向**：`traverseBackward` 无法再直接从 `tail` 开始，必须先遍历到链表末尾找到最后一个节点，然后从后往前遍历。**时间复杂度从 O(n) 变为 O(n)（寻找尾部）+ O(n)（遍历），整体仍为 O(n)，但常数系数变大。**
4.  **插入操作**：
    *   `insertAtTail`：不再直接访问 `tail`，必须从头遍历找到最后一个节点，然后进行插入。**时间复杂度从 O(1) 降为 O(n)**。
    *   `insertAfter`：如果插入位置是最后一个节点，不再更新 `tail`（因为不存在）。
5.  **删除操作**：
    *   `deleteNode`：如果删除的是尾节点，不再更新 `tail`。
6.  **清空操作**：`clearList` 不再重置 `tail`。
7.  **反转操作**：`reverseList` 不再交换 `head` 和 `tail`，只需交换节点内部的指针。由于没有 `tail` 记录，反转后如果需要访问尾部，仍需遍历。



```cpp
#include <iostream>   // 用于 cout, cerr, endl
#include <cstdlib>    // 用于 exit()

using namespace std;

/**
 * @brief 双向链表节点结构体
 */
struct DNode {
    int data;       // 数据域
    DNode* prev;    // 前驱指针
    DNode* next;    // 后继指针
};

/**
 * @brief 双向链表控制结构体 (无尾指针版本)
 * 
 * 仅维护头指针和大小。
 * 注意：由于没有 tail 指针，尾插法和反向遍历的效率会降低至 O(n)。
 */
struct DLinkedList {
    DNode* head;    // 头指针：指向第一个节点
    // DNode* tail; // 已移除
    int size;       // 大小：记录链表中节点的总数
};

/**
 * @brief 初始化链表
 */
void initList(DLinkedList* list) {
    list->head = nullptr; 
    // list->tail = nullptr; // 已移除
    list->size = 0;       
}

/**
 * @brief 创建新节点
 */
DNode* createNode(int value) {
    DNode* newNode = new(std::nothrow) DNode;
    
    if (!newNode) {
        cerr << "Memory allocation failed" << endl;
        exit(1);
    }
    
    newNode->data = value;
    newNode->prev = nullptr;
    newNode->next = nullptr;
    
    return newNode;
}

/**
 * @brief 正向遍历链表
 * 
 * 从头节点开始，沿着 next 指针逐个访问。
 */
void traverseForward(DLinkedList* list) {
    DNode* curr = list->head;
    cout << "List (Forward): ";
    
    while (curr != nullptr) {
        cout << curr->data << " ";
        curr = curr->next;
    }
    cout << endl;
}

/**
 * @brief 反向遍历链表 (无 tail 指针版本)
 * 
 * 由于没有 tail 指针，必须先从头遍历找到最后一个节点，然后再向前遍历。
 * 时间复杂度: O(n) 用于找尾 + O(n) 用于遍历 = O(n)
 */
void traverseBackward(DLinkedList* list) {
    cout << "List (Backward): ";
    
    if (list->head == nullptr) {
        cout << endl;
        return;
    }

    // 1. 先找到尾节点
    DNode* curr = list->head;
    while (curr->next != nullptr) {
        curr = curr->next;
    }
    
    // 2. 从尾节点向前遍历
    while (curr != nullptr) {
        cout << curr->data << " ";
        curr = curr->prev;
    }
    cout << endl;
}

/**
 * @brief 搜索节点
 */
DNode* search(DLinkedList* list, int value) {
    DNode* curr = list->head;
    
    while (curr != nullptr) {
        if (curr->data == value) {
            return curr;
        }
        curr = curr->next;
    }
    
    return nullptr;
}

/**
 * @brief 头插法插入节点
 * 
 * 时间复杂度: O(1)
 */
void insertAtHead(DLinkedList* list, int value) {
    DNode* newNode = createNode(value);
    
    if (list->head == nullptr) {
        // 链表为空，新节点即为头节点（也是唯一的节点）
        list->head = newNode;
        // 无 tail 指针，无需更新
    } else {
        // 链表非空
        newNode->next = list->head;
        list->head->prev = newNode;
        list->head = newNode;
    }
    
    list->size++;
}

/**
 * @brief 尾插法插入节点 (无 tail 指针版本)
 * 
 * 由于没有 tail 指针，必须从头遍历找到最后一个节点。
 * 时间复杂度: O(n)
 */
void insertAtTail(DLinkedList* list, int value) {
    DNode* newNode = createNode(value);
    
    if (list->head == nullptr) {
        // 链表为空
        list->head = newNode;
    } else {
        // 1. 遍历找到最后一个节点
        DNode* curr = list->head;
        while (curr->next != nullptr) {
            curr = curr->next;
        }
        
        // 2. 执行插入
        newNode->prev = curr;
        curr->next = newNode;
        // 无 tail 指针，无需更新
    }
    
    list->size++;
}

/**
 * @brief 在指定节点后插入
 */
void insertAfter(DLinkedList* list, DNode* pos, int value) {
    if (pos == nullptr) {
        return; 
    } 
    
    DNode* newNode = createNode(value);
    
    newNode->next = pos->next;
    newNode->prev = pos;
    
    if (pos->next != nullptr) {
        pos->next->prev = newNode;
    } 
    // 如果 pos->next 为 nullptr，说明 pos 原本是尾节点。
    // 在有 tail 的版本中需要更新 tail，但在无 tail 版本中无需操作。
    
    pos->next = newNode;
    
    list->size++;
}

/**
 * @brief 删除指定值的节点
 */
bool deleteNode(DLinkedList* list, int value) {
    DNode* curr = search(list, value);
    
    if (curr == nullptr) {
        return false; 
    }
    
    // 修复前驱链接
    if (curr->prev != nullptr) {
        curr->prev->next = curr->next;
    } else {
        // 删除的是头节点
        list->head = curr->next;
    }
    
    // 修复后继链接
    if (curr->next != nullptr) {
        curr->next->prev = curr->prev;
    } 
    // 如果 curr->next 为 nullptr，说明删除的是尾节点。
    // 在有 tail 的版本中需要更新 tail，但在无 tail 版本中无需操作。
    
    delete curr;
    list->size--;
    return true;
}

/**
 * @brief 清空链表
 */
void clearList(DLinkedList* list) {
    DNode* curr = list->head;
    
    while (curr != nullptr) {
        DNode* next = curr->next;
        delete curr;
        curr = next;
    }
    
    list->head = nullptr;
    // list->tail = nullptr; // 已移除
    list->size = 0;
}

/**
 * @brief 反转链表 (无 tail 指针版本)
 * 
 * 只需交换每个节点的指针方向。
 * 由于没有 tail 变量，不需要交换 head 和 tail 的值。
 * 但是，反转后原来的 head 变成了尾，原来的尾变成了 head。
 * 我们需要将 list->head 指向原来的尾节点（即现在的头节点）。
 */
void reverseList(DLinkedList* list) {
    if (list->head == nullptr) {
        return;
    }
    
    DNode* curr = list->head;
    DNode* temp = nullptr;
    
    // 记录新的头节点（即当前的尾节点），以便最后更新 list->head
    // 我们需要先找到当前的尾节点，或者在遍历过程中记录最后一个处理的节点
    DNode* newHead = nullptr;

    // 遍历交换每个节点的指针方向
    while (curr != nullptr) {
        // 记录当前节点作为潜在的新头（因为它将是遍历到的最后一个节点）
        newHead = curr;

        // 交换 prev 和 next
        temp = curr->prev;
        curr->prev = curr->next;
        curr->next = temp;
        
        // 移动到下一个节点（注意：next 已经变成了原来的 prev）
        curr = curr->prev; 
    }
    
    // 更新链表的头指针为新的头节点
    list->head = newHead;
}

int main() {
    // 声明链表对象
    DLinkedList myList;
    initList(&myList); // 初始化
    
    cout << "--- Testing Insertions ---" << endl;
    // 测试尾插: 10, 20 (注意：现在是 O(n) 操作)
    insertAtTail(&myList, 10);
    insertAtTail(&myList, 20);
    // 测试头插: 5 -> 10 -> 20
    insertAtHead(&myList, 5);
    
    cout << "Initial List:" << endl;
    traverseForward(&myList); // 预期: 5 10 20
    traverseBackward(&myList); // 预期: 20 10 5
    
    cout << "\n--- Testing Search and InsertAfter ---" << endl;
    // 测试搜索
    DNode* found = search(&myList, 10);
    if (found) {
        cout << "Found 10, inserting 15 after it." << endl;
        insertAfter(&myList, found, 15); 
        // 预期链表: 5 <-> 10 <-> 15 <-> 20
    }
    
    traverseForward(&myList);
    traverseBackward(&myList);
    
    cout << "\n--- Testing Deletion ---" << endl;
    cout << "Deleting 5 (Head)..." << endl;
    deleteNode(&myList, 5); // 删除头节点
    traverseForward(&myList); // 预期: 10 15 20
    
    cout << "Deleting 15 (Middle node)..." << endl;
    deleteNode(&myList, 15);
    traverseForward(&myList);  // 预期: 10 20

    // 重新查找 10，因为之前的 found 指针可能仍然有效，但为了逻辑严谨重新搜一次
    found = search(&myList, 10);
    if(found) {
        cout << "Inserting 15 after 10 again." << endl;
        insertAfter(&myList, found, 15);  
        traverseForward(&myList); // 预期: 10 15 20
    }
    
    cout << "Deleting 20 (Tail)..." << endl;
    deleteNode(&myList, 20); // 删除尾节点
    traverseForward(&myList); // 预期: 10 15
    traverseBackward(&myList); // 预期: 15 10
    
    cout << "\n--- Testing Reverse ---" << endl;
    cout << "Reversing list..." << endl;
    reverseList(&myList);
    
    cout << "Forward traversal after reverse:" << endl;
    traverseForward(&myList);   // 预期: 15 10
    
    cout << "Backward traversal after reverse:" << endl;
    traverseBackward(&myList);  // 预期: 10 15
    
    cout << "\n--- Cleaning Up ---" << endl;
    // 清理内存
    clearList(&myList);
    cout << "List cleared successfully." << endl;
    
    return 0;
}
```

### 代码差异点总结

| 操作             | 有 `tail` 指针版本          | 无 `tail` 指针版本  | 备注              |
| :------------- | :--------------------- | :------------- | :-------------- |
| 结构体            | `head`, `tail`, `size` | `head`, `size` | 节省 8 字节 (64位系统) |
| `insertAtTail` | O(1)                   | O(N)           | 最大痛点，需遍历全表      |
| 删除逻辑           | 需判断并更新 `list.tail`     | 无需更新全局指针       | 代码逻辑略微简化        |
| 反向遍历           | O(N)                   | O(N)           | 总体耗时差不多，但常数略大   |

再次强调，这个无尾的版本仅仅供学习对比用，平时解编程题目，尽量使用有头有尾的模式。下一步，我们会学习一个采用数组来“池化”内存的实现方式，避免使用 new，delete的动态内存操作，提高性能，适合竞赛的场景使用。

---
## 五、有头哨兵，无尾指针结构的实现

1.  **哨兵节点（Dummy Head）**：
    -   `initList`：现在分配一个实际的节点给 `list->head`。该节点的 `data` 无意义（通常设为0），`prev` 和 `next` 初始化为 `nullptr`。
    -   **核心逻辑变化**：`head` 不再代表第一个数据节点，而是代表链表的“入口”。第一个实际数据节点现在是 `head->next`。

2.  **遍历逻辑调整**：
    -   `traverseForward`：从 `list->head->next` 开始遍历，跳过哨兵节点。

3.  **插入逻辑调整**：
    -   `insertAtHead`：不再判断链表是否为空（因为哨兵节点永远存在）。新节点总是插入在 `head` 和 `head->next` 之间。
    -   `insertAtTail`：从 `head` 开始遍历寻找尾部，或者在空表时直接挂在 `head` 后面。

4.  **删除逻辑调整**：
    -   `deleteNode`：由于有了哨兵节点，即使删除的是第一个数据节点，其前驱（`prev`）也永远不为空（指向 `head`）。这简化了删除头节点的代码逻辑，不再需要单独处理“更新 `list->head`”的情况。


```cpp
#include <iostream>   // 用于 cout, cerr, endl
#include <cstdlib>    // 用于 exit()

using namespace std;

/**
 * @brief 双向链表节点结构体
 */
struct DNode {
    int data;       // 数据域
    DNode* prev;    // 前驱指针
    DNode* next;    // 后继指针
};

/**
 * @brief 双向链表控制结构体
 */
struct DLinkedList {
    DNode* head;    // 头指针：指向哨兵节点（Dummy Node）
    int size;       // 大小：记录链表中实际数据节点的总数
};

/**
 * @brief 初始化链表
 * 
 * 关键修改：分配一个哨兵节点，而不是将 head 设为 nullptr。
 */
void initList(DLinkedList* list) {
    // 1. 创建哨兵节点
    list->head = new(std::nothrow) DNode;
    if (!list->head) {
        cerr << "Memory allocation failed for head node" << endl;
        exit(1);
    }

    // 2. 初始化哨兵节点
    list->head->data = 0;       // 哨兵节点的数据通常不使用
    list->head->prev = nullptr; // 哨兵节点的前驱始终为空
    list->head->next = nullptr; // 初始时后继为空（链表为空）
    
    list->size = 0;       
}

/**
 * @brief 创建新节点
 */
DNode* createNode(int value) {
    DNode* newNode = new(std::nothrow) DNode;
    
    if (!newNode) {
        cerr << "Memory allocation failed" << endl;
        exit(1);
    }
    
    newNode->data = value;
    newNode->prev = nullptr;
    newNode->next = nullptr;
    
    return newNode;
}

/**
 * @brief 正向遍历链表
 * 
 * 修改：从 head->next 开始遍历，跳过哨兵节点。
 */
void traverseForward(DLinkedList* list) {
    // 从第一个实际数据节点开始
    DNode* curr = list->head->next; 
    
    cout << "List (Forward): ";
    
    while (curr != nullptr) {
        cout << curr->data << " ";
        curr = curr->next;
    }
    cout << endl;
}

/**
 * @brief 反向遍历链表
 * 
 * 逻辑：先找到尾节点，再向前遍历。
 */
void traverseBackward(DLinkedList* list) {
    cout << "List (Backward): ";
    
    // 如果 head->next 为空，说明链表无数据
    if (list->head->next == nullptr) {
        cout << endl;
        return;
    }

    // 1. 先找到尾节点
    DNode* curr = list->head->next; // 从头节点的下一个开始找
    while (curr->next != nullptr) {
        curr = curr->next;
    }
    
    // 2. 从尾节点向前遍历
    while (curr != nullptr) {
        cout << curr->data << " ";
        curr = curr->prev;
    }
    cout << endl;
}

/**
 * @brief 搜索节点
 */
DNode* search(DLinkedList* list, int value) {
    // 从第一个实际数据节点开始搜索
    DNode* curr = list->head->next;
    
    while (curr != nullptr) {
        if (curr->data == value) {
            return curr;
        }
        curr = curr->next;
    }
    
    return nullptr;
}

/**
 * @brief 头插法插入节点
 * 
 * 修改：始终插入在 head 和 head->next 之间。无需判断空表。
 */
void insertAtHead(DLinkedList* list, int value) {
    DNode* newNode = createNode(value);
    
    // 1. 新节点指向原第一个节点 (head->next)
    newNode->next = list->head->next;
    
    // 2. 新节点的前驱指向哨兵节点
    newNode->prev = list->head;
    
    // 3. 如果原链表不为空，原第一个节点的前驱指向新节点
    if (list->head->next != nullptr) {
        list->head->next->prev = newNode;
    }
    
    // 4. 哨兵节点的 next 指向新节点
    list->head->next = newNode;
    
    list->size++;
}

/**
 * @brief 尾插法插入节点
 */
void insertAtTail(DLinkedList* list, int value) {
    DNode* newNode = createNode(value);
    
    // 寻找尾节点。注意：从 head 开始找，这样即使链表为空（head->next 为 null）也能处理
    DNode* curr = list->head;
    while (curr->next != nullptr) {
        curr = curr->next;
    }
    
    // 此时 curr 是尾节点（或者是哨兵节点，如果链表为空）
    // 执行插入
    newNode->prev = curr;
    curr->next = newNode;
    
    list->size++;
}

/**
 * @brief 在指定节点后插入
 */
void insertAfter(DLinkedList* list, DNode* pos, int value) {
    if (pos == nullptr) {
        return; 
    } 
    
    DNode* newNode = createNode(value);
    
    newNode->next = pos->next;
    newNode->prev = pos;
    
    if (pos->next != nullptr) {
        pos->next->prev = newNode;
    } 
    
    pos->next = newNode;
    
    list->size++;
}

/**
 * @brief 删除指定值的节点
 * 
 * 修改：利用哨兵节点特性，删除第一个节点时逻辑更统一。
 */
bool deleteNode(DLinkedList* list, int value) {
    DNode* curr = search(list, value);
    
    if (curr == nullptr) {
        return false; 
    }
    
    // 修复前驱链接
    // 注意：因为有哨兵节点，curr->prev 永远不会是 nullptr（至少指向 head）
    curr->prev->next = curr->next;
    
    // 修复后继链接
    if (curr->next != nullptr) {
        curr->next->prev = curr->prev;
    }
    
    delete curr;
    list->size--;
    return true;
}

/**
 * @brief 清空链表
 */
void clearList(DLinkedList* list) {
    DNode* curr = list->head->next; // 从第一个数据节点开始删除
    
    while (curr != nullptr) {
        DNode* next = curr->next;
        delete curr;
        curr = next;
    }
    
    // 重置哨兵节点的指针
    list->head->next = nullptr;
    list->head->prev = nullptr;
    list->size = 0;
}

/**
 * @brief 反转链表（不动 Dummy）
 */
void reverseList(DLinkedList* list) {
    DNode* curr = list->head->next;
    if (!curr) return;  // 空链表

    DNode* prev = nullptr;
    DNode* next = nullptr;

    // 反转有效节点（不包括 Dummy）
    while (curr != nullptr) {
        next = curr->next;
        curr->next = prev;
        curr->prev = next;
        prev = curr;
        curr = next;
    }

    // prev 现在是新的头节点
    list->head->next = prev;
    prev->prev = list->head;
}

/**
 * @brief 反转链表
 * 
 * 算法逻辑：
 * 1. 保存原链表的第一个数据节点。
 * 2. 断开哨兵节点(head)与原链表的连接。
 * 3. 遍历原链表，依次将节点使用“头插法”插入到哨兵节点之后。
 */
void reverseList2(DLinkedList* list) {
    // 1. 如果链表为空或只有一个节点，无需反转
    if (list->head->next == nullptr || list->head->next->next == nullptr) {
        return;
    }
    
    // 2. 保存原链表的第一个数据节点（后续遍历的游标）
    DNode* curr = list->head->next;
    
    // 3. 关键步骤：断开哨兵节点与原链表的连接
    // 将 head->next 置空，相当于初始化一个新的空链表（保留哨兵）
    list->head->next = nullptr;
    
    // 4. 遍历原链表，使用头插法重建
    while (curr != nullptr) {
        // 保存下一个节点，防止断链后丢失
        DNode* nextTemp = curr->next;
        
        // --- 头插法核心逻辑 ---
        
        // 1. 新节点(当前curr)的next指向原第一个节点(head->next)
        curr->next = list->head->next;
        
        // 2. 如果原链表不为空（即head->next不为空），原第一个节点的prev指向curr
        if (list->head->next != nullptr) {
            list->head->next->prev = curr;
        }
        
        // 3. curr的prev指向哨兵节点
        curr->prev = list->head;
        
        // 4. 哨兵节点的next指向curr，确立curr为新的头
        list->head->next = curr;
        
        // --- 头插法结束 ---
        
        // 移动到下一个待处理节点
        curr = nextTemp;
    }
}

int main() {
    DLinkedList myList;
    initList(&myList); // 初始化（创建哨兵节点）
    
    cout << "--- Testing Insertions ---" << endl;
    // 测试尾插
    insertAtTail(&myList, 10);
    insertAtTail(&myList, 20);
    // 测试头插
    insertAtHead(&myList, 5);
    
    cout << "Initial List:" << endl;
    traverseForward(&myList); // 预期: 5 10 20
    traverseBackward(&myList); // 预期: 20 10 5
    
    cout << "\n--- Testing Search and InsertAfter ---" << endl;
    DNode* found = search(&myList, 10);
    if (found) {
        cout << "Found 10, inserting 15 after it." << endl;
        insertAfter(&myList, found, 15); 
    }
    
    traverseForward(&myList); // 预期: 5 10 15 20
    traverseBackward(&myList); // 预期: 20 15 10 5
    
    cout << "\n--- Testing Deletion ---" << endl;
    cout << "Deleting 5 (Head)..." << endl;
    deleteNode(&myList, 5); 
    traverseForward(&myList); // 预期: 10 15 20
    
    cout << "Deleting 15 (Middle node)..." << endl;
    deleteNode(&myList, 15);
    traverseForward(&myList);  // 预期: 10 20

    found = search(&myList, 10);
    if(found) {
        cout << "Inserting 15 after 10 again." << endl;
        insertAfter(&myList, found, 15);  
        traverseForward(&myList); // 预期: 10 15 20
    }
    
    cout << "Deleting 20 (Tail)..." << endl;
    deleteNode(&myList, 20); 
    traverseForward(&myList); // 预期: 10 15
    traverseBackward(&myList); // 预期: 15 10
    
    cout << "\n--- Testing Reverse ---" << endl;
    cout << "Reversing list..." << endl;
    reverseList(&myList);
    
    cout << "Forward traversal after reverse:" << endl;
    traverseForward(&myList);   // 预期: 15 10
    
    cout << "Backward traversal after reverse:" << endl;
    traverseBackward(&myList);  // 预期: 10 15
    
    cout << "\n--- Cleaning Up ---" << endl;
    clearList(&myList);
    cout << "List cleared successfully." << endl;
    
    // 注意：通常不需要 delete list->head，除非你要销毁整个链表结构体本身
    // delete myList.head; 
    
    return 0;
}
```


我今天学到这里，还有一些场景组合我没有整理，一是自己先思考，写写代码，同时可以利用AI工具让他们去写，跟他们对比思路上的差异，编程技巧的不同，重点是理解在发生变化时，如何先保留好旧连接关系，然后再建立新连接关系，以及头尾的边界处理。
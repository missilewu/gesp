# GESP C++ 单向链表、双向链表的数组实现方式

前面介绍过单向链表和双向链表的常规实现，采用的是动态分配内存的方式实现的“动态链表”或“真链表”， 是一个**通用型**的数据结构，对我们应对GESP考试足够用了。但是在竞赛的场景下，追求极致的运行效率时，往往会采用一种基于数组的“池化”内存管理技术实现，不少竞赛的教材中也推荐使用。下面的内容就是把前面的文章中的代码转换为了一个 **“高性能/工程型”** 的数据结构。虽然牺牲了灵活性（固定大小），但换取了更高的运行效率和更稳定的内存表现。因为内存是静态的，也称作“静态链表”

## 单向链表（数组版本，静态链表）

### 核心变化说明

1.  **内存管理**：不再使用 `new` 和 `delete`，对应实现了 (`allocateNode` / `freeNode`)。我们预先定义了一个固定大小的数组 `pool` 来充当内存池。
2.  **节点定义**：`next` 指针变成了 `int` 类型的下标（`next` index）。`NIL` 即 -1` 代表 `nullptr`（空）。
3.  **空闲链表**：为了模拟 `new` 的操作，我们维护一个“空闲链表”（freehead是空闲链表的头）。当需要插入数据时，从空闲链表取一个节点；当删除数据时，将节点还给空闲链表。
4.  **头结点**：依然保留头结点的概念，头结点的下标固定为 `0`。

```cpp
#include <iostream>
using namespace std;

// --- 0. 定义常量与结构体 ---
const int MAX_SIZE = 100; // 定义链表最大容量
const int NIL = -1;       // 用 -1 表示空指针 (nullptr)

struct Node {
    int data;
    int next; // 【关键修改】这里不再是 Node* 指针，而是数组下标 (int)
    
    Node() : data(0), next(NIL) {} // -1 代表 nullptr
};

// 全局内存池（模拟堆内存）
// pool[0] 固定作为头结点
Node pool[MAX_SIZE]; 

// 全局变量：空闲链表的头下标
// 初始时，所有节点连成一条空闲链：1->2->3...->MAX_SIZE-1
int freeHead = 1; 

// 初始化内存池（将空闲节点串联起来）
void initPool() {
    for (int i = 1; i < MAX_SIZE - 1; i++) {
        pool[i].next = i + 1;
    }
    pool[MAX_SIZE - 1].next = NIL; // 最后一个节点指向空
    freeHead = 1; // 重置空闲链表头
}

// 模拟 malloc/new：从空闲链表获取一个节点
int allocateNode() {
    if (freeHead == NIL) {
        cout << "Error: Memory Pool Overflow!" << endl;
        return NIL; // 内存已满
    }
    int newNodeIndex = freeHead;      // 取出当前空闲节点
    freeHead = pool[freeHead].next;   // 移动空闲链表头
    return newNodeIndex;
}

// 模拟 free/delete：将节点归还给空闲链表
void freeNode(int index) {
    if (index < 0 || index >= MAX_SIZE) return;
    pool[index].next = freeHead; // 将节点插回空闲链表头部
    freeHead = index;
}

// --- 1. 初始化链表 ---
// 返回头结点的下标（固定为0）
int initList() {
    initPool(); // 初始化内存池
    pool[0].data = 0; // 头结点数据
    pool[0].next = NIL; // 头结点指向空
    return 0; // 返回头结点下标
}

// --- 2. 遍历 (Traversal) ---
void traverse(int head) {
    int curr = pool[head].next; // 从第一个有效节点开始
    
    if (curr == NIL) {
        cout << "List is empty." << endl;
        return;
    }

    cout << "List: ";
    while (curr != NIL) {
        cout << pool[curr].data << " -> ";
        curr = pool[curr].next;
    }
    cout << "NULL" << endl;
}

// --- 3. 搜索 (Search) ---
// 返回找到的节点下标，若未找到返回 NIL
int search(int head, int target) {
    int curr = pool[head].next;
    while (curr != NIL) {
        if (pool[curr].data == target) {
            return curr;
        }
        curr = pool[curr].next;
    }
    return NIL;
}

// --- 4. 头部插入 (Insert at Head) ---
void insertAtHead(int head, int val) {
    int newIndex = allocateNode(); // 申请新节点
    if (newIndex == NIL) return;

    pool[newIndex].data = val;
    
    // 1. 新节点指向原来的第一个有效节点
    pool[newIndex].next = pool[head].next;
    // 2. 头结点指向新节点
    pool[head].next = newIndex;
}

// --- 5. 尾部插入 (Insert at Tail) ---
void insertAtTail(int head, int val) {
    int newIndex = allocateNode();
    if (newIndex == NIL) return;

    pool[newIndex].data = val;
    pool[newIndex].next = NIL;

    int curr = head;
    // 找到最后一个节点
    while (pool[curr].next != NIL) {
        curr = pool[curr].next;
    }
    pool[curr].next = newIndex;
}

// --- 6. 指定位置插入 (Insert after a specific node) ---
void insertAfter(int prevIndex, int val) {
    if (prevIndex == NIL) {
        cout << "Error: Previous node cannot be nullptr." << endl;
        return;
    }

    int newIndex = allocateNode();
    if (newIndex == NIL) return;

    pool[newIndex].data = val;
    pool[newIndex].next = pool[prevIndex].next;
    pool[prevIndex].next = newIndex;
}

// --- 7. 删除节点 (Delete by value) ---
bool deleteNode(int head, int target) {
    if (pool[head].next == NIL) return false;

    int curr = head; // 从头结点开始找前驱
    
    // 寻找前驱节点
    while (pool[curr].next != NIL && pool[pool[curr].next].data != target) {
        curr = pool[curr].next;
    }

    // 如果找到了目标节点
    if (pool[curr].next != NIL) {
        int temp = pool[curr].next;       // 记录要删除的节点下标
        pool[curr].next = pool[temp].next; // 跳过目标节点
        freeNode(temp);                // 【关键】归还内存
        return true;
    }

    return false;
}

// --- 8. 反转链表 (Reverse) ---
void reverseList(int head) {
    int prev = NIL;
    int curr = pool[head].next;
    int nextTemp = NIL;

    while (curr != NIL) {
        nextTemp = pool[curr].next; // 暂存下一个
        pool[curr].next = prev;     // 反转指向
        prev = curr;                // 指针后移
        curr = nextTemp;
    }
    // 更新头结点的 next 指向新的首节点
    pool[head].next = prev;
}

// --- 9. 清理链表 (Clear) ---
// 将所有节点归还给空闲链表
void clearList(int head) {
    int curr = pool[head].next;
    while (curr != NIL) {
        int temp = curr;
        curr = pool[curr].next;
        freeNode(temp); // 归还节点
    }
    pool[head].next = NIL; // 重置头结点
}

// ================= 主函数测试 =================
int main() {
    // 初始化，返回头结点下标 0
    int myList = initList(); 

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
    int foundIndex = search(myList, 4);
    if (foundIndex != NIL) cout << "Found 4 at index: " << foundIndex << ", Value: " << pool[foundIndex].data << endl;
    else cout << "4 not found." << endl;

    cout << "\n--- 4. 插入测试 (在 4 后面插入 100) ---" << endl;
    if (foundIndex != NIL) {
        insertAfter(foundIndex, 100);
        traverse(myList); 
    }

    cout << "\n--- 5. 删除测试 (删除头节点 5) ---" << endl;
    if (deleteNode(myList, 5)) {
        cout << "Deleted 5 successfully." << endl;
        traverse(myList); 
    } 

    cout << "\n--- 6. 反转测试 ---" << endl;
    cout << "Before Reverse: ";
    traverse(myList);
    reverseList(myList);
    cout << "After Reverse:  ";
    traverse(myList); 

    cout << "\n--- 7. 清理内存 ---" << endl;
    clearList(myList);
    traverse(myList); 

    return 0;
}
```

## 双向链表（数组版本，静态链表）

### 核心变化说明

1.  **移除指针**：不再使用 `DNode*`，而是使用 `int` 类型的**下标**（Index）来表示“指针”。
2.  **静态内存池**：定义一个固定大小的全局数组 `pool` 作为内存池，代替 `new`。
3.  **空指针表示**：使用常量 `NIL`（定义为 -1）来代替 `nullptr`。
4.  **空闲链表管理**：为了模拟 `new` 和 `delete` 的效果，对应实现了 (`allocateNode` / `freeNode`)，我们需要维护一个“空闲链表” （freehead是头），记录数组中哪些位置是空的，可以被复用。当我们“删除”一个节点时，我们并没有真正擦除数据，而是将该节点的下标重新链接到 `freeHead` 链表中，这样下次 `allocateNode` 时就可以复用这个位置。这完全模拟了动态内存分配的行为。


```cpp
#include <iostream>
#include <cstdlib>

using namespace std;

// --- 常量定义 ---
const int MAX_SIZE = 100; // 静态数组的最大容量
const int NIL = -1;       // 用 -1 表示空指针 (nullptr)

/**
 * @brief 静态节点结构体
 * 
 * 将指针替换为整型下标。
 */
struct DNode {
    int data;       // 数据域
    int prev;       // 前驱下标：指向数组中的索引，-1 表示无
    int next;       // 后继下标：指向数组中的索引，-1 表示无
};

/**
 * @brief 静态双向链表控制结构体
 */
struct DLinkedList {
    int head;       // 头节点下标
    int tail;       // 尾节点下标
    int size;       // 当前节点数量
    int freeHead;   // 空闲链表的头下标（用于管理可用空间）
};

// --- 全局静态内存池 ---
// 相当于堆内存，但在编译时分配
DNode pool[MAX_SIZE]; 

/**
 * @brief 初始化空闲链表
 * 
 * 将所有数组位置串联起来，形成一个空闲链表，以便后续分配。
 * 0 -> 1 -> 2 -> ... -> MAX_SIZE-1 -> NIL
 */
void initFreeList(DLinkedList* list) {
    for (int i = 0; i < MAX_SIZE - 1; i++) {
        pool[i].next = i + 1; // 每个节点的 next 指向下一个空闲位置
    }
    pool[MAX_SIZE - 1].next = NIL; // 最后一个节点指向 NIL
    list->freeHead = 0; // 空闲链表从头开始
}

/**
 * @brief 初始化链表
 * 
 * @param list 指向链表结构体的指针
 */
void initList(DLinkedList* list) {
    list->head = NIL;
    list->tail = NIL;
    list->size = 0;
    initFreeList(list); // 初始化内存池管理
}

/**
 * @brief 分配节点 (模拟 new)
 * 
 * 从空闲链表中取出一个节点。
 * @param list 指向链表结构体的指针
 * @return 可用的数组下标，如果内存满则返回 NIL
 */
int allocateNode(DLinkedList* list) {
    if (list->freeHead == NIL) {
        cerr << "Error: Memory Pool Overflow (Array Full)" << endl;
        exit(1);
    }
    
    // 取出空闲链表的第一个节点
    int newNodeIndex = list->freeHead;
    
    // 更新空闲链表头指针，指向下一个空闲节点
    list->freeHead = pool[newNodeIndex].next;
    
    // 初始化新节点的数据（可选，为了安全）
    pool[newNodeIndex].prev = NIL;
    pool[newNodeIndex].next = NIL;
    
    return newNodeIndex;
}

/**
 * @brief 回收节点 (模拟 delete)
 * 
 * 将不用的节点归还到空闲链表中。
 * @param list 指向链表结构体的指针
 * @param index 要回收的节点下标
 */
void freeNode(DLinkedList* list, int index) {
    if (index == NIL) return;
    
    // 将该节点插入到空闲链表的头部
    pool[index].next = list->freeHead;
    list->freeHead = index;
}

/**
 * @brief 正向遍历链表
 * 
 * @param list 指向链表结构体的指针
 */
void traverseForward(DLinkedList* list) {
    int curr = list->head; // 从 head 下标开始
    cout << "List (Forward): ";
    
    while (curr != NIL) {
        cout << pool[curr].data << " ";
        curr = pool[curr].next; // 移动到下一个下标
    }
    cout << endl;
}

/**
 * @brief 反向遍历链表
 * 
 * @param list 指向链表结构体的指针
 */
void traverseBackward(DLinkedList* list) {
    int curr = list->tail; // 从 tail 下标开始
    cout << "List (Backward): ";
    
    while (curr != NIL) {
        cout << pool[curr].data << " ";
        curr = pool[curr].prev; // 移动到前一个下标
    }
    cout << endl;
}

/**
 * @brief 搜索节点
 * 
 * @param list 指向链表结构体的指针
 * @param value 要查找的值
 * @return 找到则返回节点下标，否则返回 NIL
 */
int search(DLinkedList* list, int value) {
    int curr = list->head;
    
    while (curr != NIL) {
        if (pool[curr].data == value) {
            return curr;
        }
        curr = pool[curr].next;
    }
    
    return NIL;
}

/**
 * @brief 头插法插入节点
 * 
 * @param list 指向链表结构体的指针
 * @param value 要插入的值
 */
void insertAtHead(DLinkedList* list, int value) {
    int newNode = allocateNode(list); // 模拟 new
    pool[newNode].data = value;
    
    if (list->head == NIL) {
        // 链表为空
        list->head = newNode;
        list->tail = newNode;
    } else {
        // 链表非空
        pool[newNode].next = list->head;
        pool[list->head].prev = newNode;
        list->head = newNode;
    }
    
    list->size++;
}

/**
 * @brief 尾插法插入节点
 * 
 * @param list 指向链表结构体的指针
 * @param value 要插入的值
 */
void insertAtTail(DLinkedList* list, int value) {
    int newNode = allocateNode(list); // 模拟 new
    pool[newNode].data = value;
    
    if (list->tail == NIL) {
        // 链表为空
        list->head = newNode;
        list->tail = newNode;
    } else {
        // 链表非空
        pool[newNode].prev = list->tail;
        pool[list->tail].next = newNode;
        list->tail = newNode;
    }
    
    list->size++;
}

/**
 * @brief 在指定节点后插入
 * 
 * @param list 指向链表结构体的指针
 * @param posIndex 参考节点的下标
 * @param value 要插入的值
 */
void insertAfter(DLinkedList* list, int posIndex, int value) {
    if (posIndex == NIL) {
        return; 
    } 
    
    int newNode = allocateNode(list);
    pool[newNode].data = value;
    
    // 1. 新节点的 next 指向 pos 的下一个节点
    pool[newNode].next = pool[posIndex].next;
    // 2. 新节点的 prev 指向 pos
    pool[newNode].prev = posIndex;
    
    // 3. 如果 pos 不是最后一个节点
    if (pool[posIndex].next != NIL) {
        pool[pool[posIndex].next].prev = newNode;
    } else {
        // 特殊情况：pos 是尾节点
        list->tail = newNode;
    }
    
    // 4. pos 的 next 指向新节点
    pool[posIndex].next = newNode;
    
    list->size++;
}

/**
 * @brief 删除指定值的节点
 * 
 * @param list 指向链表结构体的指针
 * @param value 要删除的值
 * @return 删除成功返回 true，未找到返回 false
 */
bool deleteNode(DLinkedList* list, int value) {
    int curr = search(list, value);
    
    if (curr == NIL) {
        return false; 
    }
    
    // --- 修复前驱链接 ---
    if (pool[curr].prev != NIL) {
        pool[pool[curr].prev].next = pool[curr].next;
    } else {
        list->head = pool[curr].next;
    }
    
    // --- 修复后继链接 ---
    if (pool[curr].next != NIL) {
        pool[pool[curr].next].prev = pool[curr].prev;
    } else {
        list->tail = pool[curr].prev;
    }
    
    // 释放内存 (模拟 delete)
    freeNode(list, curr);
    
    list->size--;
    return true;
}

/**
 * @brief 清空链表
 * 
 * @param list 指向链表结构体的指针
 */
void clearList(DLinkedList* list) {
    int curr = list->head;
    
    while (curr != NIL) {
        int next = pool[curr].next; // 保存下一个
        freeNode(list, curr);       // 归还到空闲池
        curr = next;
    }
    
    list->head = NIL;
    list->tail = NIL;
    list->size = 0;
    // 注意：这里不需要重新 initFreeList，因为 freeNode 已经把节点加回去了
}

/**
 * @brief 反转链表
 * 
 * @param list 指向链表结构体的指针
 */
void reverseList(DLinkedList* list) {
    if (list->head == NIL) {
        return;
    }
    
    int curr = list->head;
    int temp;
    
    // 1. 交换头尾记录
    temp = list->head;
    list->head = list->tail;
    list->tail = temp;
    
    // 2. 交换每个节点的指针
    while (curr != NIL) {
        temp = pool[curr].prev;
        pool[curr].prev = pool[curr].next;
        pool[curr].next = temp;
        
        // 移动：因为指针交换了，prev 指向原来的 next
        curr = pool[curr].prev; 
    }
}

int main() {
    DLinkedList myList;
    initList(&myList);
    
    cout << "--- Testing Static Array List ---" << endl;
    
    // 测试插入
    insertAtTail(&myList, 10);
    insertAtTail(&myList, 20);
    insertAtHead(&myList, 5);
    
    cout << "Initial List:" << endl;
    traverseForward(&myList); // 5 10 20
    traverseBackward(&myList); // 20 10 5
    
    cout << "\n--- Testing Search and InsertAfter ---" << endl;
    int found = search(&myList, 10);
    if (found != NIL) {
        cout << "Found 10 at index " << found << ", inserting 15." << endl;
        insertAfter(&myList, found, 15); 
    }
    traverseForward(&myList); // 5 10 15 20
    
    cout << "\n--- Testing Deletion ---" << endl;
    cout << "Deleting 5 (Head)..." << endl;
    deleteNode(&myList, 5);
    traverseForward(&myList); // 10 15 20
    
    cout << "Deleting 20 (Tail)..." << endl;
    deleteNode(&myList, 20);
    traverseForward(&myList); // 10 15
    
    cout << "\n--- Testing Reverse ---" << endl;
    reverseList(&myList);
    traverseForward(&myList); // 15 10
    traverseBackward(&myList); // 10 15
    
    cout << "\n--- Cleaning Up ---" << endl;
    clearList(&myList);
    cout << "List cleared." << endl;
    
    return 0;
}
```

## 两种方案的详细对比分析

利用AI工具对比整理的优缺点分析，虽然精简，但是较为准确，了解即可。

### 📊 核心优缺点对比表

| 特性          | 方案一：指针实现 (原代码)                     | 方案二：数组实现 (新代码)               |
| :---------- | :--------------------------------- | :--------------------------- |
| **内存分配**    | **动态分配** (`new`/`delete`)，按需申请     | **静态预分配** (全局数组)，固定上限        |
| **内存位置**    | **不连续** (分散在堆内存中)                  | **连续** (在数组内存块中)             |
| **插入/删除速度** | **快** (仅修改指针，O(1))                 | **快** (仅修改游标下标，O(1))         |
| **缓存友好性**   | **差** (节点分散，易导致 CPU Cache Miss)    | **好** (数据连续，CPU 预取效率高)       |
| **灵活性**     | **高** (受限于系统总内存，理论上无限)             | **低** (受限于 `MAX_SIZE`，需预先估算) |
| **内存开销**    | **较高** (每个节点需额外存储指针，且 `new` 有管理开销) | **较低** (仅存储整数下标，无额外管理开销)     |
| **调试难度**    | **难** (容易出现内存泄漏、野指针、段错误)           | **易** (无内存泄漏风险，越界容易检查)       |
| **适用场景**    | 通用软件开发、数据量未知或变化巨大的场景               | 算法竞赛、嵌入式系统、高频交易系统            |

---

### 🔍 深度解析

#### 1. 内存管理与安全性
*   **指针方案**：
    *   **优点**：非常灵活。只要系统内存足够，链表就可以一直增长。
    *   **缺点**：`new` 和 `delete` 是昂贵的系统调用，频繁调用会导致**内存碎片**。如果忘记 `delete`，会导致**内存泄漏**；如果访问已释放的内存，会导致程序崩溃（段错误）。
*   **数组方案**：
    *   **优点**：**绝对安全**。没有内存泄漏，因为内存是全局静态分配的。所有的“分配”和“释放”只是整数的赋值操作，速度极快。
    *   **缺点**：必须预先定义 `MAX_SIZE`。如果数据量超过这个值，程序会报错或无法插入（溢出）。

#### 2. 性能与 CPU 缓存 (关键点)
这是数组实现最大的隐藏优势。
*   **指针方案**：节点散落在内存的各个角落。当你遍历链表时，CPU 无法有效利用 **L1/L2 Cache**（因为读取一个节点可能触发一次昂贵的内存读取），这被称为“缓存未命中” (Cache Miss)。
*   **数组方案**：所有节点存储在连续的内存块中。当你访问 `pool[i]` 时，CPU 往往会把周围的数据（如 `pool[i+1]`）也加载到缓存中。因此，**数组实现的链表在遍历和查找时，通常比指针实现快得多**（有时甚至快几倍）。

#### 3. 空间效率
*   **指针方案**：在 64 位系统中，一个指针占用 **8 字节**，两个指针就是 16 字节；
*   **数组方案**：一个数组下标（`int`）通常只占用 **4 字节**。
*   **结论**：如果链表节点本身的数据很小（例如只存一个 `int`），数组方案能节省近一半的存储空间（4字节数据 + 4字节下标 vs 4字节数据 + 8字节指针 + 内存管理头开销）。

#### 4. 代码实现复杂度
*   **指针方案**：逻辑直观，符合人类对“链”的直觉，但需要小心处理空指针和边界情况。
*   **数组方案**：引入了“游标”的概念，逻辑上稍微绕一点（例如“下标 -1 代表空”），且无法使用标准的 C++ 容器或智能指针，移植性较差。

---

### 💡 什么时候该用哪种？

#### 选择 **指针实现** (方案一) 当：
1.  你正在开发通用的软件库或应用程序。
2.  你无法预估数据量的大小，或者数据量可能非常大。
3.  你需要频繁地改变链表结构，且不想受到固定大小的限制。
4.  代码的可读性和标准性比极致的性能优化更重要。

#### 选择 **数组实现** (方案二) 当：
1.  **算法竞赛 (ACM/OI)**：这是最常见的用法。因为不需要处理内存泄漏，且运行速度极快，能卡时间复杂度。
2.  **嵌入式开发**：内存资源极其有限，且不允许动态内存分配（为了防止内存碎片导致系统崩溃）。
3.  **高频交易/实时系统**：要求操作具有确定性的时间复杂度，不能容忍 `new` 操作带来的不确定的系统延迟。
4.  **数据恢复/持久化**：因为数组结构可以直接 `memcpy` 写入硬盘，下次读取时直接读回内存，指针会失效，但数组下标不会。

# 05-GESP C++ 双向循环链表的两种实现（有头+哨兵）

**双向循环链表**（Doubly Circular Linked List），由于其头尾相连且双向可遍历的特性，在处理约瑟夫环、滑块谜题或需要频繁首尾插入删除的题目时非常高效。虽然GESP考试中常用`std::list`或`std::vector`解编程题，但**手写链表**是考察指针操作、内存管理和算法逻辑的核心考点，也是五级以上选择和判断题的高频考点。

有了前面单向循环链表的学习和联系，理解双向循环链表会更容易一些，重点针对头（head）的操作。基于前面学习的有头无尾（没有哨兵节点）的双向链表`DLinkedList` ，我们可以实现一版双向循环链表`DCircularList`。它们之间的差异和变化主要有：

1. **循环特性**：
    - **尾节点的 `next`** 指向 **头节点**。
    - **头节点的 `prev`** 指向 **尾节点**。
    - 空链表时，`head` 为 `nullptr`。
2. **遍历逻辑**：
    - 正向遍历时，终止条件从 `curr != nullptr` 变为检查是否回到了 `head`。
    - 反向遍历时，不再需要 O(n) 查找尾节点，直接通过 `head->prev` 获取尾节点，效率提升为 O(n)。
3. **插入/删除逻辑**：
    - 在操作头尾节点时，必须维护循环链接（即更新 `head->prev` 和 `tail->next`）。
4. **反转逻辑**：
    - 交换每个节点的 `prev` 和 `next` 指针。
    - 由于是循环结构，反转后原来的**尾节点**实际上变成了新的**头节点**。
    - 因此，在遍历交换完所有指针后，必须更新 `list->head` 指向原来的尾节点（即 `list->head->prev`），以保持头指针指向正确的位置。

## 定义和初始化
### 1. 结构体定义与节点创建

双向循环链表的核心在于：

- 每个节点有 `prev` (前驱) 和 `next` (后继)。
- 头节点的 `prev` 指向尾节点。
- 尾节点的 `next` 指向头节点。
- 空表时，头节点的 `next` 和 `prev` 均指向自己。

```cpp
#include <iostream>   // 用于 cout, cerr, endl
#include <cstdlib>    // 用于 exit()

using namespace std;

/**
 * @brief 双向链表节点结构体 (保持不变)
 */
struct DNode {
    int data;       // 数据域
    DNode* prev;    // 前驱指针
    DNode* next;    // 后继指针
};

/**
 * @brief 双向循环链表控制结构体
 * 
 * 修改点：
 * 1. 结构体名称改为 DCircularList。
 * 2. 逻辑上，尾节点的 next 指向头节点，头节点的 prev 指向尾节点。
 */
struct DCircularList {
    DNode* head;    // 头指针：指向第一个节点 (如果为空则为 nullptr)
    int size;       // 大小：记录链表中节点的总数
};

/**
 * @brief 初始化链表
 */
void initList(DCircularList* list) {
    list->head = nullptr; // 注意这里是空指针，不是空节点
    list->size = 0;       
}

/**
 * @brief 创建新节点 (保持不变)
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
```

## 遍历和搜索操作
### 2. 遍历操作（Traversal）

遍历时需注意终止条件：当指针回到 `head` 时停止。

```cpp

/**
 * @brief 正向遍历链表
 * 
 * 修改点：循环条件改变。从 head 开始，直到再次回到 head 停止。
 */
void traverseForward(DCircularList* list) {
    if (list->head == nullptr) {
        cout << "List (Forward): (Empty)" << endl;
        return;
    }

    DNode* curr = list->head;
    cout << "List (Forward): ";
    
    do {
        cout << curr->data << " ";
        curr = curr->next;
    } while (curr != list->head);
    
    cout << endl;
}

/**
 * @brief 反向遍历链表 (循环链表版本)
 * 
 * 修改点：
 * 1. 效率提升。不再需要从头遍历找尾。
 * 2. 直接通过 head->prev 找到尾节点。
 */
void traverseBackward(DCircularList* list) {
    if (list->head == nullptr) {
        cout << "List (Backward): (Empty)" << endl;
        return;
    }

    cout << "List (Backward): ";
    
    // 1. 直接通过 head->prev 获取尾节点
    DNode* curr = list->head->prev;
    
    // 2. 向前遍历，直到回到 head
    do {
        cout << curr->data << " ";
        curr = curr->prev;
    } while (curr != list->head->prev); // 当 curr 再次等于尾节点时停止
    
    cout << endl;
}
```


### 5. 搜索操作（Search）

返回找到的节点指针，若未找到返回 `nullptr`。

```cpp
/**
 * @brief 搜索节点 (保持不变)
 */
DNode* search(DCircularList* list, int value) {
    if (list->head == nullptr) return nullptr;

    DNode* curr = list->head;
    
    do {
        if (curr->data == value) {
            return curr;
        }
        curr = curr->next;
    } while (curr != list->head);
    
    return nullptr;
}

```


## 插入操作

### 3. 头部插入（有教材称为：Prepend）

在循环链表的**头前面**插入一个节点（pre-pend），因为是一个环，所以逻辑上跟在尾部后面插入一个节点的逻辑类似，区别是把head设置为这个新的节点（移动了头节点）。

```c
/**
 * @brief 头插法插入节点
 * 
 * 修改点：
 * 1. 处理空链表时的自循环。
 * 2. 处理非空链表时，新节点与旧头节点、尾节点的链接关系。
 */
void insertAtHead(DCircularList* list, int value) {
    DNode* newNode = createNode(value);
    
    if (list->head == nullptr) {
        // 链表为空：新节点指向自己
        list->head = newNode;
        newNode->next = newNode;
        newNode->prev = newNode;
    } else {
        // 链表非空
        // 1. 找到当前的尾节点 (head->prev)
        // 注意：head 的前驱就是尾结点(tail)
        DNode* tail = list->head->prev;
        
        // 2. 插入新节点
        newNode->next = list->head;
        newNode->prev = tail;
        
        // 3. 更新旧尾和旧头的链接
        tail->next = newNode;
        list->head->prev = newNode;
        
        // 4. 更新头指针
        list->head = newNode;
    }
    
    list->size++;
}
```

### 4. 尾部插入（有教材称为：Append）

在循环链表的**尾后面**插入一个节点（ap-pend），逻辑与头插法类似，只是头指针不移动（注意跟头插法的区别，脑子里面有个对应的图像，后面写代码的时候就会比较顺利）。

```cpp
/**
 * @brief 尾插法插入节点
 * 
 * 修改点：
 * 1. 利用循环特性，尾插法现在也是 O(1) 操作（不需要遍历找尾）。
 * 2. 逻辑与头插法类似，只是头指针不移动。
 */
void insertAtTail(DCircularList* list, int value) {
    DNode* newNode = createNode(value);
    
    if (list->head == nullptr) {
        // 链表为空
        list->head = newNode;
        newNode->next = newNode;
        newNode->prev = newNode;
    } else {
        // 链表非空
        DNode* tail = list->head->prev;
        
        newNode->prev = tail;
        newNode->next = list->head;
        
        tail->next = newNode;
        list->head->prev = newNode;
        
        // 注意：尾插法不改变 head 指针
    }
    
    list->size++;
}

```


### 6. 插入指定位置（Insert After）

在某个已知节点后插入，通常需要结合搜索操作使用（如：找到某个期望的节点后，进行插入、删除等一系列的操作）。

```cpp
/**
 * @brief 在指定节点后插入
 * 
 * 修改点：
 * 1. 处理 pos 是尾节点的情况（此时 pos->next 是 head）。
 */
void insertAfter(DCircularList* list, DNode* pos, int value) {
    if (pos == nullptr) {
        return; 
    } 
    
    DNode* newNode = createNode(value);
    
    DNode* nextNode = pos->next;
    
    // 插入新节点
    newNode->prev = pos;
    newNode->next = nextNode;
    
    // 修复周围节点
    pos->next = newNode;
    nextNode->prev = newNode;
    
    // 如果是在尾节点后插入（即在 head 之前插入），
    // 不需要改变 head 指针，除非题目要求 insertAfter 尾节点等于 insertAtHead，
    // 但通常 insertAfter 只是链接，不改变 list->head 指向。
    
    list->size++;
}
```

## 删除操作
### 7. 删除操作（Delete）

删除节点时需要小心处理前后指针的连接，并释放内存。

```cpp
/**
 * @brief 删除指定值的节点
 * 
 * 修改点：
 * 1. 处理删除后链表变空的情况。
 * 2. 维护循环链接。
 */
bool deleteNode(DCircularList* list, int value) {
    DNode* curr = search(list, value);
    
    if (curr == nullptr) {
        return false; 
    }
    
    // 情况1: 链表只有一个节点
    if (curr->next == curr && curr->prev == curr) {
        delete curr;
        list->head = nullptr;
        list->size--;
        return true;
    }
    
    // 情况2: 链表有多个节点
    // 1. 修复前驱和后继的链接
    curr->prev->next = curr->next;
    curr->next->prev = curr->prev;
    
    // 2. 如果删除的是头节点，更新 head 指针指向下一个节点
    if (curr == list->head) {
        list->head = curr->next;
    }
    
    delete curr;
    list->size--;
    return true;
}
```


### 8. 清空操作（Clear）

清空链表，并释放内存。

```cpp
/**
 * @brief 清空链表
 */
void clearList(DCircularList* list) {
    if (list->head == nullptr) return;

    DNode* curr = list->head;
    DNode* tail = list->head->prev; // 记录尾节点，作为结束标志
    
    // 遍历删除所有节点
    // 注意：这里不能简单地用 do-while 删除，因为删除 curr 后 curr->next 可能失效
    // 但逻辑上我们需要遍历一圈。
    
    while (list->head != nullptr) {
        DNode* toDelete = list->head;
        
        // 如果只剩一个节点
        if (toDelete->next == toDelete) {
            delete toDelete;
            list->head = nullptr;
        } else {
            // 断开链接并删除头节点
            DNode* newHead = toDelete->next;
            DNode* tailNode = toDelete->prev;
            
            tailNode->next = newHead;
            newHead->prev = tailNode;
            
            delete toDelete;
            list->head = newHead;
        }
    }
    
    list->size = 0;
}
```

## 反转操作
### 9. 链表反转（Reverse）

这是高频考点。对于双向链表，反转只需要交换每个节点的 `prev` 和 `next` 指针，最后还需要交换头节点的指向（或者简单地交换遍历方向，但在物理结构上通常要求彻底反转）。

**注意**：在双向循环链表中，如果仅仅交换每个节点的 `prev` 和 `next`，链表的逻辑方向就反了。由于是循环的，原来的 `head->next` 变成了 `head->prev`。为了保持 `head` 依然作为入口且 `head->next` 指向新的第一个元素，我们需要做特殊处理，或者直接交换整个链表的 `next` 和 `prev` 逻辑（即遍历时走 `prev` 指针）。

下面给出**物理结构完全反转**的实现：交换所有节点的指针，并调整头节点的连接关系。

```c
/**
 * @brief 反转链表
 * 
 * 修改点：
 * 1. 交换每个节点的 prev 和 next。
 * 2. 由于是循环链表，反转后原来的 head 变成了 tail，原来的 tail 变成了 head。
 * 3. 需要更新 list->head 指向原来的 tail。
 */
void reverseList(DCircularList* list) {
    if (list->head == nullptr) {
        return;
    }
    
    DNode* curr = list->head;
    DNode* temp = nullptr;
    
    // 记录当前的尾节点，反转后它将变成新的头节点
    DNode* oldTail = list->head->prev;

    do {
        // 交换 prev 和 next
        temp = curr->prev;
        curr->prev = curr->next;
        curr->next = temp;
        
        // 移动到下一个节点 (原来的 prev，现在的 next)
        curr = curr->next;
        
    } while (curr != list->head);
    
    // 更新头指针：原来的尾节点变成了现在的头节点
    list->head = oldTail;
}
```

## 测试样例
### 10. 主函数测试样例

```cpp
int main() {
    // 声明链表对象
    DCircularList myList;
    initList(&myList); // 初始化
    
    cout << "--- Testing Insertions ---" << endl;
    // 测试尾插
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

    // 重新查找 10
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

## 编程过程时，推荐使用有哨兵的版本，简单不易出错

下面是一个带有**哨兵头节点**的双向循环链表实现，对比和体会哨兵节点带来的简化（尤其是节省了大量的`head==nullptr`的空链表的判断。主要这些主要修改点。

1.  **哨兵节点**：在 `initList` 中，`list->head` 不再指向 `nullptr`，而是指向一个专门创建的“哨兵”节点。
2.  **空链表状态**：当链表为空时，哨兵节点的 `next` 和 `prev` 都指向它自己。
3.  **遍历逻辑**：
    -   正向遍历从 `head->next` 开始，直到再次遇到 `head` 停止。
    -   反向遍历从 `head->prev` 开始，直到再次遇到 `head` 停止。
4.  **插入/删除简化**：由于有了哨兵节点，链表永远不会真正“为空”（始终有一个节点），因此不需要专门处理 `head == nullptr` 的特殊情况，所有插入和删除操作逻辑统一。

---


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
 * @brief 双向循环链表控制结构体 (带哨兵头节点版本)
 * 
 * 修改点：
 * 1. list->head 始终指向一个哨兵节点，永不指向 nullptr。
 * 2. 哨兵节点不存储有效数据。
 */
struct DCircularList {
    DNode* head;    // 哨兵头指针
    int size;       // 大小：记录链表中有效节点的总数
};

/**
 * @brief 创建新节点 (保持不变)
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
 * @brief 初始化链表
 * 
 * 修改点：创建一个哨兵节点，并使其首尾相接指向自己。
 */
void initList(DCircularList* list) {
    list->head = createNode(0); // 创建哨兵节点，数据域任意（设为0）
    list->head->next = list->head; // 注意要自己指向自己
    list->head->prev = list->head; // 注意要自己指向自己
    list->size = 0;       
}

/**
 * @brief 正向遍历链表
 * 
 * 修改点：从 head->next 开始，直到回到 head 停止。
 */
void traverseForward(DCircularList* list) {
    DNode* curr = list->head->next; // 从第一个有效节点开始
    cout << "List (Forward): ";
    
    while (curr != list->head) {
        cout << curr->data << " ";
        curr = curr->next;
    }
    
    if (list->size == 0) {
        cout << "(Empty)";
    }
    cout << endl;
}

/**
 * @brief 反向遍历链表
 * 
 * 修改点：从 head->prev (尾节点) 开始，直到回到 head 停止。
 */
void traverseBackward(DCircularList* list) {
    cout << "List (Backward): ";
    
    DNode* curr = list->head->prev; // 从尾节点开始
    
    while (curr != list->head) {
        cout << curr->data << " ";
        curr = curr->prev;
    }

    if (list->size == 0) {
        cout << "(Empty)";
    }
    cout << endl;
}

/**
 * @brief 搜索节点
 * 
 * 修改点：循环条件改为 curr != list->head。
 */
DNode* search(DCircularList* list, int value) {
    DNode* curr = list->head->next;
    
    while (curr != list->head) {
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
 * 修改点：逻辑简化，统一在 head 之后插入。
 */
void insertAtHead(DCircularList* list, int value) {
    DNode* newNode = createNode(value);
    
    // 1. 找到当前的第一个有效节点 (oldFirst)
    DNode* oldFirst = list->head->next;
    
    // 2. 插入新节点到 head 和 oldFirst 之间
    newNode->prev = list->head;
    newNode->next = oldFirst;
    
    // 3. 更新周围节点的链接
    list->head->next = newNode;
    oldFirst->prev = newNode;
    
    list->size++;
}

/**
 * @brief 尾插法插入节点
 * 
 * 修改点：逻辑简化，统一在 head 之前插入（即 tail 之后）。
 */
void insertAtTail(DCircularList* list, int value) {
    DNode* newNode = createNode(value);
    
    // 1. 找到当前的尾节点
    DNode* tail = list->head->prev;
    
    // 2. 插入新节点到 tail 和 head 之间
    newNode->prev = tail;
    newNode->next = list->head;
    
    // 3. 更新周围节点的链接
    tail->next = newNode;
    list->head->prev = newNode;
    
    list->size++;
}

/**
 * @brief 在指定节点后插入
 * 
 * 修改点：逻辑保持不变，因为 pos 不会是 nullptr（除非用户传错）。
 */
void insertAfter(DCircularList* list, DNode* pos, int value) {
    if (pos == nullptr) {
        return; 
    } 
    
    DNode* newNode = createNode(value);
    
    DNode* nextNode = pos->next;
    
    // 插入新节点
    newNode->prev = pos;
    newNode->next = nextNode;
    
    // 修复周围节点
    pos->next = newNode;
    nextNode->prev = newNode;
    
    list->size++;
}

/**
 * @brief 删除指定值的节点
 * 
 * 修改点：不需要检查删除后链表是否为空，因为始终有 head 哨兵。
 */
bool deleteNode(DCircularList* list, int value) {
    DNode* curr = search(list, value);
    
    if (curr == nullptr) {
        return false; 
    }
    
    // 1. 修复前驱和后继的链接
    curr->prev->next = curr->next;
    curr->next->prev = curr->prev;
    
    // 2. 删除节点
    delete curr;
    list->size--;
    return true;
}

/**
 * @brief 清空链表
 * 
 * 修改点：遍历删除所有有效节点，保留 head 哨兵。
 */
void clearList(DCircularList* list) {
    DNode* curr = list->head->next;
    
    // 遍历直到回到 head
    while (curr != list->head) {
        DNode* next = curr->next;
        delete curr;
        curr = next;
    }
    
    // 重置 head 的链接指向自己
    list->head->next = list->head;
    list->head->prev = list->head;
    list->size = 0;
}

/**
 * @brief 反转链表 (修正版)
 * 
 * 修正点：
 * 1. 在交换指针前，先保存下一个节点的引用。
 * 2. 正确处理哨兵节点的 next 和 prev 指向。
 */
void reverseList(DCircularList* list) {
    if (list->size == 0) {
        return;
    }
    
    // 记录原来的首节点，反转后它将变成尾节点
    DNode* oldFirst = list->head->next;
    
    DNode* curr = oldFirst;
    DNode* temp = nullptr;
    
    // 遍历所有有效节点
    // 注意：我们需要遍历一圈回到 oldFirst
    do {
        // 1. 【关键修正】在交换之前，先保存原来的 prev。
        // 因为交换后 curr->next 会变成原来的 prev，我们需要用它来移动指针。
        DNode* nextNode = curr->prev;
        
        // 2. 交换 prev 和 next 指针
        temp = curr->prev;
        curr->prev = curr->next;
        curr->next = temp;
        
        // 3. 移动到下一个节点
        curr = nextNode;
        
    } while (curr != oldFirst);
    
    // 4. 更新哨兵节点的链接
    // 反转后，oldFirst 变成了尾节点，所以 head->prev 指向它
    list->head->prev = oldFirst;
    // 反转后，oldFirst->prev (即原来的头节点的前一个，也就是原来的尾) 变成了首节点
    // 所以 head->next 指向 oldFirst->prev
    list->head->next = oldFirst->prev;
}

int main() {
    // 声明链表对象
    DCircularList myList;
    initList(&myList); // 初始化 (创建哨兵节点)
    
    cout << "--- Testing Insertions ---" << endl;
    // 测试尾插
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

    // 重新查找 10
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
    // 清理内存 (保留哨兵节点)
    clearList(&myList);
    cout << "List cleared successfully." << endl;
    
    return 0;
}
```


我今天学到这里，对链表的理解，有头，有哨兵的概念有了新的理解，我们一起加油操练吧，把基础打好。
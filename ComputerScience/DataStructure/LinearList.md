# 线性表知识点

## 一、线性表概述

### 定义

由 `n` 个相同类型元素构成的有限序列，逻辑上属于线性结构。

### 存储表示形式

- **顺序表**：物理地址连续，通常使用数组存储。

- 链表

  ：链式存储结构，数据存储离散，通过指针相连。链表可根据不同维度进行分类：

  - 单向 / 双向：
    - 单向链表：仅含后继指针（如单链表）。
    - 双向链表：含前驱和后继指针（如双向链表）。
  - 带头 / 不带头：
    - 无头链表：直接通过首节点操作（如单链表）。
    - 带头链表：增设头节点（哑节点），简化边界条件处理（如双向链表）。
  - 循环 / 非循环：
    - 非循环链表：尾节点指针为`NULL`（如单链表）。
    - 循环链表：尾节点指针指向头节点，形成环（如双向链表通常为循环结构）。

常见链表结构：

- **无头单向非循环链表**：结构简单，常用于子结构（如哈希表拉链）。
- **带头双向循环链表**：操作高效，常用于独立数据存储（如 STL 的`list`）。

---



## 二、顺序表

### 1. 结构定义

顺序表是用一段物理地址连续的存储单元依次存储数据元素，分为静态和动态。

#### 动态顺序表结构 C 语言表示

```c
typedef struct SeqList 
{
    int* array;    // 动态数组指针
    size_t size;   // 有效数据个数
    size_t capacity; // 容量
} SeqList;
```

### 2. 顺序表核心操作实现

#### 初始化顺序表

```c
void SeqListInit(SeqList* psl) {
    assert(psl);
    psl->array = NULL;
    psl->size = psl->capacity = 0;
}
```

#### 检查容量并增容（2 倍扩容）

```c
void CheckCapacity(SeqList* psl) {
    assert(psl);
    if (psl->size == psl->capacity) {
        size_t new_capacity = psl->capacity == 0 ? 4 : psl->capacity * 2;
        int* new_array = (int*)realloc(psl->array, new_capacity * sizeof(int));
        if (new_array == NULL) {
            perror("realloc failed");
            return;
        }
        psl->array = new_array;
        psl->capacity = new_capacity;
    }
}
```

#### 尾插

```c
void SeqListPushBack(SeqList* psl, int x) {
    CheckCapacity(psl);
    psl->array[psl->size++] = x;
}
```

#### 任意位置插入（pos 从 0 开始）

```c
void SeqListInsert(SeqList* psl, size_t pos, int x) {
    assert(psl);
    assert(pos <= psl->size); // 位置合法性检查
    CheckCapacity(psl);
    // 后移元素
    memmove(psl->array + pos + 1, psl->array + pos, (psl->size - pos) * sizeof(int));
    psl->array[pos] = x;
    psl->size++;
}
```

#### 任意位置删除（pos 从 0 开始）

```c
void SeqListErase(SeqList* psl, size_t pos) {
    assert(psl);
    assert(pos < psl->size); // 位置合法性检查
    // 前移元素
    memmove(psl->array + pos, psl->array + pos + 1, (psl->size - pos - 1) * sizeof(int));
    psl->size--;
}
```

#### 销毁顺序表

```c
void SeqListDestroy(SeqList* psl) {
    assert(psl);
    free(psl->array);
    psl->array = NULL;
    psl->size = psl->capacity = 0;
}
```

### 3. 顺序表的优缺点

#### 插入 / 删除效率

中间和头部插入删除需要移动数据，时间复杂度 O (N)。

#### 优点

- 支持随机访问，访问效率 O (1)。
- 缓存命中率高，符合局部性原理。

#### 缺点

- 需要预先分配空间，可能存在空间浪费（如申请 200 个，只用 120 个，浪费 80 个）。
- 数据量不稳定时，扩容频繁，且扩容可能导致内存碎片。

---



## 三、单链表

### 1. 结构定义

单链表是链式存储结构，数据存储离散，通过指针相连。每个节点包含数据域和指针域，指针指向下一个节点。

#### 单链表结构 C 语言表示

```c
typedef int SLNDataType;
struct SLNode
{
    SLNDataType val;
    struct SLNode* next;
};
```

### 2. 常见操作实现

#### （1）尾插操作

**功能**：在链表尾部插入新节点。
**代码实现**：

```c
void SLTPushBack(struct SLNode** pphead, SLNDataType x)
{
    struct SLNode* newnode = (struct SLNode*)malloc(sizeof(struct SLNode));
    assert(newnode);
    newnode->val = x;
    newnode->next = NULL;

    if (*pphead == NULL)
    {
        *pphead = newnode;
    }
    else
    {
        struct SLNode* tail = *pphead;
        while (tail->next != NULL)
        {
            tail = tail->next;
        }
        tail->next = newnode;
    }
}
```

**逻辑说明**：

- 若链表为空，直接让头指针指向新节点。
- 若链表非空，遍历找到尾节点，将尾节点的 `next` 指向新节点。

#### （2）头插操作

**功能**：在链表头部插入新节点。
**代码实现**：

```c
void SLTPushFront(struct SLNode** pphead, SLNDataType x)
{
    struct SLNode* newnode = (struct SLNode*)malloc(sizeof(struct SLNode));
    assert(newnode);
    newnode->val = x;
    newnode->next = *pphead;
    *pphead = newnode;
}
```

**逻辑说明**：

- 新节点的 `next` 指向原头节点，更新头指针指向新节点，完成头插。

#### （3）尾删操作

**功能**：删除链表尾部节点。
**代码实现**

```c
void SLTPopBack(struct SLNode** pphead)
{
    if (*pphead == NULL)
    {
        return;
    }
    if ((*pphead)->next == NULL)
    {
        free(*pphead);
        *pphead = NULL;
    }
    else
    {
        struct SLNode* prev = NULL;
        struct SLNode* tail = *pphead;
        while (tail->next != NULL)
        {
            prev = tail;
            tail = tail->next;
        }
        free(tail);
        prev->next = NULL;
    }
}
```

**逻辑说明**：

- 若链表只有一个节点，直接释放头节点并置空。
- 若链表有多个节点，找到尾节点的前驱节点，释放尾节点，前驱节点的 `next` 置为 `NULL`。

#### （4）头删操作

**功能**：删除链表头部节点。
**代码实现**：

```c
void SLTPopFront(struct SLNode** pphead)
{
    if (*pphead == NULL)
    {
        return;
    }
    struct SLNode* tmp = *pphead;
    *pphead = (*pphead)->next;
    free(tmp);
}
```

**逻辑说明**：

- 保存原头节点，头指针后移指向下一个节点，释放原头节点内存。

#### （5）打印链表

**功能**：遍历打印链表所有节点值。
**代码实现**：

```c
void SLTPrint(struct SLNode* phead)
{
    struct SLNode* cur = phead;
    while (cur != NULL)
    {
        printf("%d ", cur->val);
        cur = cur->next;
    }
    printf("\n");
}
```

#### （6）查找值为 x 的节点（返回首个匹配节点）

```c
struct SLNode* SLTFind(struct SLNode* phead, SLNDataType x) {
    struct SLNode* cur = phead;
    while (cur != NULL) {
        if (cur->val == x) {
            return cur;
        }
        cur = cur->next;
    }
    return NULL; // 未找到
}
```

#### （7）在 pos 节点之后插入（单链表只能后插，前插需遍历找前驱）

```c
void SLTInsertAfter(struct SLNode* pos, SLNDataType x) {
    assert(pos);
    struct SLNode* newnode = (struct SLNode*)malloc(sizeof(struct SLNode));
    assert(newnode);
    newnode->val = x;
    newnode->next = pos->next;
    pos->next = newnode;
}
```

#### （8）删除 pos 节点之后的节点

```c
void SLTEraseAfter(struct SLNode* pos) {
    assert(pos);
    assert(pos->next != NULL); // 确保pos后有节点
    struct SLNode* del = pos->next;
    pos->next = del->next;
    free(del);
}
```

### 3. 单链表操作注意事项

- **指针传递**：涉及头指针修改的操作（如头插、头删、尾插空链表），需使用二级指针 `struct SLNode** pphead`，确保函数内外头指针同步修改。
- **空指针检查**：操作前检查链表是否为空，避免空指针解引用导致程序崩溃（如尾删、头删前判断 `*pphead == NULL`）。
- **内存管理**：动态分配内存的节点，删除时必须调用 `free` 释放内存，防止内存泄漏（如尾删释放 `tail`，头删释放 `tmp`）。

### 4. 单链表的优缺点

- 优点：
  - 插入和删除操作效率高，已知位置时时间复杂度为 O (1)（仅需修改指针）。
  - 动态分配内存，无需预先分配大量空间，不会造成空间浪费。
- 缺点：
  - 不支持随机访问，查找元素需从头遍历，时间复杂度为 O (N)。
  - 每个节点需额外存储指针域，增加空间开销。

---



## 四、双向链表

### 1. 双向链表结构定义

```c
typedef int LTDataType;
struct LTNode {
    struct LTNode* prev;  // 指向前驱节点
    struct LTNode* next;  // 指向后继节点
    LTDataType val;       // 存储数据
};
```

### 2. 核心操作实现

#### （1）初始化（头节点初始化）

```c
void LTInit(LTNode** pphead) 
{
    *pphead = (LTNode*)malloc(sizeof(LTNode));
    if (*pphead == NULL) 
    {
        perror("malloc fail");
        return;
    }
    (*pphead)->prev = *pphead;
    (*pphead)->next = *pphead;
}
```

#### （2）头插法

```c
void LTPushFront(LTNode* phead, LTDataType x) 
{
    assert(phead);
    LTNode* newnode = (LTNode*)malloc(sizeof(LTNode));
    assert(newnode);
    newnode->val = x;

    LTNode* first = phead->next;
    // 调整指针
    phead->next = newnode;
    newnode->prev = phead;
    newnode->next = first;
    first->prev = newnode;
}
```

#### （3）头删操作

```c
void LTPopFront(LTNode* phead) 
{
    assert(phead);
    assert(phead->next != phead); // 确保非空

    LTNode* first = phead->next;
    LTNode* second = first->next;

    phead->next = second;
    second->prev = phead;

    free(first);
    first = NULL;
}
```

#### （4）尾插法

```c
void LTPushBack(LTNode* phead, LTDataType x) 
{
    assert(phead);
    LTNode* newnode = (LTNode*)malloc(sizeof(LTNode));
    assert(newnode);
    newnode->val = x;

    LTNode* tail = phead->prev;
    // 调整指针
    tail->next = newnode;
    newnode->prev = tail;
    newnode->next = phead;
    phead->prev = newnode;
}
```

#### （5）尾删操作

```c
void LTPopBack(LTNode* phead) 
{
    assert(phead);
    assert(phead->next != phead);

    LTNode* tail = phead->prev;
    LTNode* tailPrev = tail->prev;

    tailPrev->next = phead;
    phead->prev = tailPrev;

    free(tail);
    tail = NULL;
}
```

#### （6）任意位置前插入

```c
void LTInsert(LTNode* pos, LTDataType x) 
{
    assert(pos);
    LTNode* newnode = (LTNode*)malloc(sizeof(LTNode));
    assert(newnode);
    newnode->val = x;

    LTNode* posPrev = pos->prev;
    // 调整指针
    posPrev->next = newnode;
    newnode->prev = posPrev;
    newnode->next = pos;
    pos->prev = newnode;
}
```

#### （7）任意位置删除

```c
void LTErase(LTNode* pos) 
{
    assert(pos);
    assert(pos != phead); // 防止删除头节点
    LTNode* posPrev = pos->prev;
    LTNode* posNext = pos->next;

    posPrev->next = posNext;
    posNext->prev = posPrev;

    free(pos);
    pos = NULL;
}
```

#### （8）链表销毁

```c
void LTDestroy(LTNode** pphead) 
{
    assert(pphead);
    LTNode* phead = *pphead;
    LTNode* cur = phead->next;
    while (cur != phead) {
        LTNode* next = cur->next;
        free(cur);
        cur = next;
    }
    free(phead);
    *pphead = NULL;
}
```

#### （9）查找值为 x 的节点（返回首个匹配节点）

```c
struct LTNode* LTFind(struct LTNode* phead, LTDataType x) {
    assert(phead);
    struct LTNode* cur = phead->next; // 从首节点开始遍历
    while (cur != phead) { // 循环链表，cur != phead时终止
        if (cur->val == x) {
            return cur;
        }
        cur = cur->next;
    }
    return NULL; // 未找到
}
```

### 3. 双向链表特点总结

- **双向遍历**：可通过 `prev` 和 `next` 指针实现前后双向遍历。
- **操作高效**：插入删除无需遍历找前驱，直接通过指针操作，时间复杂度低。
- **结构对称**：节点结构对称，代码逻辑在处理前后指针时具有对称性，便于理解和维护。

---



## 五、顺序表与链表对比

### 顺序表与单链表对比

| **对比项**      | **顺序表**                     | **单链表**                   |
| --------------- | ------------------------------ | ---------------------------- |
| **存储形式**    | 物理地址连续                   | 链式存储离散                 |
| **访问效率**    | 随机访问 O (1)                 | 顺序访问 O (N)               |
| **插入 / 删除** | 中间和头部 O (N)（需移动数据） | 已知位置 O (1)（仅改指针）   |
| **空间占用**    | 预先分配，可能浪费空间         | 按需分配，无空间浪费         |
| **缓存友好**    | 高，局部性原理                 | 低                           |
| **适用场景**    | 数据量稳定，频繁访问           | 数据量动态变化，频繁插入删除 |
| **缓存利用率**  | 高（CPU 高速缓存命中率比较高） | 低                           |

### 顺序表与双向链表对比

| **对比维度**    | **顺序表**                      | **双向链表**                        |
| --------------- | ------------------------------- | ----------------------------------- |
| **存储方式**    | 连续内存空间存储                | 非连续内存空间，通过指针连接        |
| **随机访问**    | 支持随机访问，时间复杂度 O (1)  | 不支持随机访问，需遍历              |
| **插入 / 删除** | 中间或头部操作需移动元素，O (N) | 任意位置插入 / 删除时间复杂度 O (1) |
| **空间分配**    | 需预分配空间，可能浪费或溢出    | 动态申请空间，灵活度高              |
| **内存碎片**    | 无内存碎片问题                  | 可能产生内存碎片                    |
| **适用场景**    | 频繁随机访问，少插入删除场景    | 频繁插入 / 删除，无需随机访问场景   |

## 六、栈

### 一、栈的概念及结构

#### 定义

栈是一种特殊的线性表，其只允许在固定的一端进行插入和删除元素操作。进行数据插入和删除操作的一端称为栈顶，另一端称为栈底。栈中的数据元素遵守**后进先出（LIFO，Last In First Out）** 的原则。

#### 核心特性

- **压栈（入栈）**：在栈顶插入元素，时间复杂度为 \(O(1)\)（不考虑扩容）。
- **出栈（弹栈）**：在栈顶删除元素，时间复杂度为 \(O(1)\)。
- **栈顶元素**：始终是最后入栈的元素，支持 \(O(1)\) 时间访问。

#### 存储形式

栈的实现通常有两种方式：

1. **顺序栈（数组实现）**：物理地址连续，通过动态数组实现，尾插尾删效率高，缓存命中率高，是主流实现方式。
2. **链栈（链表实现）**：链式存储，无需预分配空间，但存在指针开销，实际应用较少。

### 二、顺序栈（动态数组实现）

#### 1. 结构定义

```c
typedef int STDataType;

typedef struct Stack {
    STDataType* a;       // 动态数组指针
    size_t top;         // 栈顶位置（指向栈顶元素的下一个位置）
    size_t capacity;    // 栈的容量
} ST;
```

#### 2. 核心操作实现

##### （1）初始化栈

```c
void STInit(ST* pst) 
{
    assert(pst);
    pst->a = NULL;
    pst->top = 0;
    pst->capacity = 0;
}
```

- **功能**：初始化栈结构，指针置空，栈顶和容量设为 0。
- **注意**：使用 `assert` 确保指针有效性。

##### （2）销毁栈

```c
void STDestroy(ST* pst) 
{
    assert(pst);
    free(pst->a);       // 释放动态数组内存
    pst->a = NULL;
    pst->top = pst->capacity = 0;
}
```

- **功能**：释放栈的内存空间，重置成员变量。

##### （3）入栈操作

```c
void STPush(ST* pst, STDataType x) 
{
    assert(pst);
    // 检查容量，按需扩容（2倍扩容）
    if (pst->top == pst->capacity) {
        size_t new_capacity = pst->capacity == 0 ? 4 : pst->capacity * 2;
        STDataType* tmp = (STDataType*)realloc(pst->a, sizeof(STDataType) * new_capacity);
        if (tmp == NULL) {
            perror("realloc failed");
            return;
        }
        pst->a = tmp;
        pst->capacity = new_capacity;
    }
    pst->a[pst->top] = x;  // 元素存入栈顶位置
    pst->top++;            // 栈顶指针后移
}
```

- **功能**：在栈顶插入元素，支持自动扩容。
- **扩容策略**：初始容量为 0 时申请 4 个空间，否则扩容为原来的 2 倍。

##### （4）出栈操作

```c
void STPop(ST* pst) 
{
    assert(pst);
    assert(pst->top > 0);  // 确保栈非空
    pst->top--;            // 栈顶指针前移，等效删除栈顶元素
}
```

- **功能**：删除栈顶元素，需先检查栈是否为空。

##### （5）获取栈顶元素

```c
STDataType STTop(ST* pst) 
{
    assert(pst);
    assert(pst->top > 0);  // 确保栈非空
    return pst->a[pst->top - 1];  // 栈顶指针前移一位获取元素
}
```

- **功能**：返回栈顶元素，不删除元素。

##### （6）获取栈中有效元素个数

```c
size_t STSize(ST* pst) 
{
    assert(pst);
    return pst->top;  // top 即为有效元素个数
}
```

##### （7）判断栈是否为空

```c
bool STEmpty(ST* pst) 
{
    assert(pst);
    return pst->top == 0;  // top为0时栈空
}
```

#### 3. 优缺点

| **优点**                                         | **缺点**                                                  |
| ------------------------------------------------ | --------------------------------------------------------- |
| 1. 尾插尾删效率高，均为 \(O(1)\)（不考虑扩容）。 | 1. 扩容时可能产生内存碎片，且扩容操作耗时 \(O(N)\)。      |
| 2. 支持随机访问栈顶元素，时间复杂度 \(O(1)\)。   | 2. 预分配空间可能导致空间浪费（如容量大于实际元素个数）。 |
| 3. 数组存储缓存命中率高，符合局部性原理。        |                                                           |

#### 4.概念对比

##### （1）栈溢出的概念（操作系统 / 内存管理范畴）

栈溢出（Stack Overflow）是指程序运行时，由于函数调用层级过深或局部变量占用空间过大，导致**线程栈空间**超过操作系统分配的内存上限，从而引发的内存错误。

- **本质**：操作系统为每个线程分配固定大小的栈空间（如 Linux 默认为 8MB），用于存储函数调用信息（如参数、局部变量、返回地址等）。当递归调用无终止、局部数组 / 结构体过大，或循环调用深层函数时，栈空间被耗尽，程序会因访问非法内存而崩溃（通常触发 `Segmentation Fault`）。
- 典型场景：
  1. 无限递归（如未设置终止条件的递归函数）。
  2. 局部变量占用空间超过栈容量（如 `int arr[1000000];` 在栈中定义超大数组）。

##### （2）与上文 “栈”（数据结构）的区别

| **对比项**      | **栈溢出（操作系统）**                                       | **上文的栈（数据结构）**                                     |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **定义范畴**    | 程序运行时的内存管理问题，属于操作系统层面概念。             | 一种遵循后进先出（LIFO）原则的线性数据结构，属于算法与数据结构范畴。 |
| **核心作用**    | 存储函数调用状态，保障程序执行流程。                         | 管理数据的插入和删除（仅在栈顶操作），实现特定逻辑（如表达式求值、括号匹配）。 |
| **“溢出” 原因** | 线程栈空间不足（由函数调用深度或局部变量大小导致）。         | 数据插入时超过栈的容量（如顺序栈未扩容时 `top == capacity`）。 |
| **处理方式**    | 调整栈空间大小（危险，可能引发内存泄漏）或优化代码（避免深层递归 / 超大局部变量）。 | 通过扩容机制（如动态数组翻倍扩容）避免溢出，或在操作前检查栈状态（如 `STPush` 前检查容量）。 |
| **影响范围**    | 导致程序崩溃或未定义行为（如覆盖其他内存数据）。             | 可能导致越界访问（如未处理扩容时的数组越界），但可通过代码逻辑控制（如断言检查）。 |

##### （3）深刻理解两者的关键

1. **区分 “逻辑栈” 与 “内存栈”**：
   - 上文的栈是**逻辑结构**，关注数据操作（入栈、出栈、取栈顶），通过代码实现（如数组 / 链表），溢出是可预期并通过扩容处理的逻辑错误（如 `STPush` 中的 `realloc`）。
   - 栈溢出是**内存管理问题**，涉及操作系统为线程分配的物理内存空间，溢出后程序无法自行恢复，需通过代码设计避免（如限制递归深度、合理分配局部变量）。
2. **数据结构栈的 “溢出” 本质是边界问题**：
   在顺序栈实现中，若未处理容量不足就插入元素，会导致数组越界（如用户之前代码中 `STTop` 误将 `top` 作为索引，实际应为 `top-1`），这是代码逻辑错误，可通过断言和扩容修复。
3. **操作系统栈溢出的核心是资源限制**：
   操作系统对栈空间的限制是为了防止单个线程占用过多内存，保障系统稳定性。理解这一点，就能明白为何深层递归（如斐波那契递归实现）会崩溃 —— 每次递归调用都在栈中创建新的函数帧，最终突破空间上限。
4. **共同的 “后进先出” 特性**：
   尽管范畴不同，两者均遵循 LIFO 原则：数据结构的栈通过代码实现逻辑顺序；操作系统的栈通过函数调用栈帧的压入 / 弹出实现执行顺序（最新调用的函数先返回）。

##### 总结

上文的 “栈” 是算法层面的抽象数据结构，通过代码控制其行为和边界；而 “栈溢出” 是操作系统层面的内存错误，源于线程栈空间不足。理解两者的关键在于区分**逻辑操作**与**内存管理**，前者通过合理设计数据结构（如动态扩容）避免溢出，后者通过优化代码逻辑（如避免无限递归）防止内存滥用。

---



## 七、队列

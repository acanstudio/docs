线性表
=============

线性表定义
-------------

线性表（List）：零个或多个数据元素的有限序列。元素之间是由顺序的，若存在多个元素，则第一个元素无前驱，最后一个元素无后继，其他每个元素都有且只有一个前驱后后继。线性表是有限的，线性表元素的个数称为线性表的长度，长度为0时，称为空表。

抽象数据类型
----------------------

<pre>
ADT 线性表(List)

Data
    线性表的数据对象集合为{a1, a2, ... , an}，每个元素的类型均为DataType，其中，除第一个元素a1外，每一个元素有且只有一个直接前驱元素，除了最后一个元素之外，每个元素有且只有一个直接后继元素，数据元素之间的关系是一对一的关系。

Operation 
    InitList(*L): 初始化操作，建立一个空的线性表L
    ListEmpty(L): 若线性表为空，返回true，否则返回false
    ClearList(*): 清空线性表
    GetElem(L, i, *e): 将L中第i个元素值返回给e
    LocateElem(L, e): 在线性表L中查找与给定值e相等的元素，若存在返回元素的在表中的序号，否则返回false
    ListInsert(*L, i, e): 在线性表L中的第i个位置插入元素e
    ListDelete(*L, i, *e): 删除线性表L中第i个元素，并用e返回其值
    ListLength(L): 返回表的元素个数
endADT
</pre>

基于线性表的基本操作可以实现更复杂的线性表操作，如实现两个线性表的合并
<pre>
// 将所有在Lb中但不在La中的元素插入到La中
void union(List *La, List Lb)
{
  int La_len, Lb_len, i;
  ElemType e;
  La_len = ListLength(La);
  Lb_len = ListLength(Lb);
  for (i = 1, i <= Lb_len; i++) {
    GetElem(Lb, i, e);
    if (!LocateElem(La, e, equal)) {
      ListInsert(La, ++La_len, e);
    }
  }
}
</pre>

顺序存储结构
-------------

线性表的顺序存储结构，指的是用一段地址连续的存储单元依次存储线性表的数据元素。具体实现通常使用一位数组

<pre>
#define MAXSIZE = 20 // 存储空间初始分配值
typedef int ElemType; // ElemType类型根据实际情况而定
typedef struct
{
  ElemType data[MAXSIZE]; // 数组存储数据元素，最大值为MAXSIZE
  int length; // 线性表当前长度
} SqList;
</pre>

三个关键属性
* 存储空间的起始位置：数组data的存储位置；
* 线性表的最大存储容量：数组长度MAXSIZE；
* 线性表的当前长度：length；

数据长度（数组的存储空间长度）通常大于或等于线性表的长度（线性表中数据元素的个数）；

对于顺序存储结构，对每个线性表位置的存入或者取出数据，对于计算机来说都是相等的时间，时间复杂度为O(1)，这种存储结构又称随机存取结构。

顺序存储结构的插入和删除
-------------------------------

获取元素的操作
<pre>
#define OK 1;
#define ERROR 0;
#define TRUE 1
#define FALSE 0
typedef int Status; // 函数类型，其值是函数结果状态代码

// 初始条件：顺序线性表L已存在，1<=i<=ListLength(L)
// 操作结果：用e返回L中第i个元素的值
Status GetElem(SqList L, int i, ElemType *e)
{
  if (L.length == 0 || i < 1 || i > L.length) {
    return false;
  }
  *e = L.data[i-1];
  return true;
}
</pre>

插入元素思路
1. 如果插入位置不合理，抛出异常；
2. 如果线性表长度大于等于数组长度，抛出异常或动态增加容量
3. 从最后一个元素开始向前遍历到第i个位置，分别将他们都向后移动一个位置；
4. 将要插入元素填入位置i处；
5. 表长加1

<pre>
// 初始条件：顺序线性表L已存在，1<=i<=ListLength(L)
// 操作结果：在L中第i个位置之前插入新的数据元素e，L的长度加1
Status ListInsert(SqList *L, int i, ElemType e)
{
  int k;
  if (L->length == MAXSIZE) {
    return false;
  }
  if (i < 1 || i > L->length + 1) {
    return false;
  }
  if (i <= L->length) { // 若插入位置不在表尾
    for (k = L->length - 1; k >= i - 1; k--) { // 将要插入位置后数据元素向后移动一位
      L->data[k+1] = L->data[k];
    }
  }
  L->data[i - 1] = e; 
  L->length++;

  return true;
}
</pre>

删除算法思路
1. 如果删除位置不合理，抛出异常；
2. 取出删除元素；
3. 从删除元素位置开始遍历到最后位置，分别将他们都向前移动一个位置；
5. 表长减1

<pre>
// 初始条件：顺序线性表L已存在，1<=i<=ListLength(L)
// 操作结果：删除L第i个位置数据元素，并用e返回其值，L的长度减1
Status ListDelete(SqList *L, int i, ElemType *e)
{
  int k;
  if (L->length == 0) {
    return false;
  }
  if (i < 1 || i > L->length) {
    return false;
  }
  *e = L->data[i - 1];
  if (i < L->length) { // 若插入位置不在表尾
    for (k = 1; k < L->length; k++) { // 将要删除位置后数据元素向前移动一位
      L->data[k-1] = L->data[k];
    }
  }
  L->length--;

  return true;
}
</pre>

顺序存储结构的优点
* 无须为表示表中元素之间的逻辑关系增加额外存储空间；
* 可以快速地存取表中任一位置元素

缺点：
* 插入和删除操作需要移动大量元素
* 当线性表长度变化较大时，难以确定存储空间的容量
* 造成存储空间的碎片


线性表的链式存储结构
----------------------

为了表示每一个数据元素ai与其直接后继元素ai+1之间的逻辑关系，对数据元素ai来说，除了存储器自身的信息外，还需存储一个指示其直接后继的信息（即直接后继的存储位置）。把存储数据元素信息的域称为数据域，存储直接后继位置的域称为指针域。指针域中的信息称为指针或链。这两部分信息组成的数据元素ai的存储映象称为结点（Node）。

n个结点链接成一个链表，即为线性表的连式存储结构，结点只包含一个指针域的又叫单链表。
链表中第一个结点的存储位置叫做头指针，最后一个结点指针为空；为了更加方便，在第一个结点前附设一个结点称为头结点，头结点数据域不存放信息或存放线性表的长度，头结点的指针域存储指向第一个结点的指针。

头指针与头结点

* 头指针是指链表指向第一个结点的指针，若链表偶头结点，则是指向头结点的指针；头指针具有标识作用，所以常用头指针管理链表的名字；无论链表是否为空，头指针均不为空，头指针是链表的必要元素；
* 头结点是为了操作同一和方便而设立的，放在第一元素结点之前，数据域一般无意义或存放链表长度；头结点不是必需的；方便处理第一结点的操作；

链式存储结构的描述
<pre>
// 线性表的单链表存储结构
typedef struct Node
{
  ElemType data;
  struct Node *next;
} Node;
typedef struct Node *LinkList; // 定义LinkList
</pre>

单链表的读取
---------------

获取第i个数据的算法思路
1. 声明一个结点p指向链表第一个结点，初始化j从1开始；
2. 当j<i时，遍历链表，让p的指针向后移动，不断质量下一个结点，j累加1；
3. 若到链表末尾p为空，则说明第i个元素不存在；
4. 否则查找成功，返回结点p的数据；

<pre>
// 读取单链表元素
// 初始条件，顺序线性表L已存在 1<=i<=ListLength(L)
// 操作结果，用e返回L中第i个元素的值
Status GetElem(LinkList L, int i, ElemType *e)
{
  int j;
  LinkList p; // 声明一个结点
  p = L->next; // 昂p指向链表L的第一个结点
  j = 1; 
  while (p && j < i) { // p不为空或计数器j还没有等于i，循环继续
    p = p->next; // 指向下一个结点
    ++j;
  }
  if (!p || j > i) {
    return false;
  }
  *e = p->data;
  return true;
}
</pre>

单链表的插入和删除
---------------------

单链表第i个数据插入结点的算法思路
1. 声明一个结点p指向链表第一个结点，初始化j从1开始；
2. 当j<i时，就遍历链表，让p的指针向后移动，不断指向下一个结点，j累加1；
3. 若到链表末尾p为空，则说明第i个元素不存在；
4. 否则查找成功，在系统中生成一个空节点s；
5. 将数据元素e赋值给s->data；
6. 单链表的插入标准语句s->next=p->next; p->next=s;
7. 返回成功；

<pre>
// 初始条件：顺序线性表L已存在，1<=i<=ListLength(L)
// 操作结果：在L中第i位置之前插入新的数据元素e，L的长度加1
Status ListInsert(LinkList *L, int i, ElemType e)
{
  int j;
  LinkList p, s;
  p = *L;
  j = 1;
  while (p && j < i) { // 寻找第i个结点
    p = p->next;
    ++j;
  }
  if (!p || j > i) {
    return false; // 第i个元素不存在
  }

  s = (LinkList)malloc(sizeof(Node)); // 生成新结点
  s->data = e;
  s->next = p->next; // 将p的后继结点赋值给s的后继
  p->next = s; // 将s赋值给p的后继
  return true;
}
</pre>

单链表第i个数据删除结点的算法思路
1. 声明一个结点p指向链表第一个结点，初始化j从1开始；
2. 当j<i时，遍历链表，让p的指针后移，不断指向下一个结点，j累加1；
3. 若到链表末尾p为空，则说明第i个元素不存在；
4. 否则查找成功，将欲删除的结点p->next赋值给q；
5. 单链表的删除标准语句p->next=q->next;
6. 将q结点中的数据赋值给e，作为返回
7. 释放q结点；
8. 返回成功

<pre>
// 初始条件：顺序线性表L已存在，1<=i<=ListLength(L)
// 操作结果：删除L第i数据元素，并用e返回其值，L的长度减1
Status ListInsert(LinkList *L, int i, ElemType e)
{
  int j;
  LinkList p, q;
  p = *L;
  j = 1;
  while (p->next && j < i) { // 寻找第i个结点
    p = p->next;
    ++j;
  }
  if (!(p->next) || j > i) {
    return false; // 第i个元素不存在
  }

  q = p->next;
  p->net = q->next; // 将q的后继赋值给p的后继
  *e = q->data;
  free(q);

  return true;
}
</pre>

对插入和删除操作比较频繁的操作，单链表的效率明显高于顺序存储结构。

单链表的整表创建
----------------------

1. 声明一结点p和计数器遍历i;
2. 初始化一空链表L；
3. 让L的头结点的指针指向null，即建立一个带头结点的单链表；
4. 循环
    * 生成一个新结点赋值给p；
    * 随机生成一数字赋值给p的数据域p->data；
    * 将p插入到头结点与前一新结点之间；

<pre>
// 随机产生n个元素的值，建立带表头结点的单链表L
// 头插入法
void CreateListHead(LinkList *L, int n)
{
  LinkList p;
  int i;
  srand(time(0));
  *L = (LinkList) malloc(sizeof(Node));
  (*L)->next = null; // 建立一个带头结点的单链表
  for (i = 0; i < n; i++) {
    p = (LinkList) malloc(sizeof(Node)); // 生成新结点
    p->data = rand() * 100 + 1; // 随机生成100以内的数字
    p->next = (*L)->next;
    (*L)->next = p;
  }
}

// 尾插入法
void CreateListTail(LinkList *L, int n)
{
  LinkList p, r;
  int i;
  srand(time(0));
  *L = (LinkList) malloc(sizeof(Node)); 
  r = *L; // r为指向尾部的结点
  for (i = 0; i < n; i++) {
    p = (Node *) malloc(sizeof(Node)); // 生成新结点
    p->data = rand() % 100 + 1; 
    r->next = p;
    r = p;
  }
  r->next = null;
}
</pre>

单链表的整表删除
------------------

1. 声明一结点p和q；
2. 将第一个结点赋值给p；
3. 循环
    * 将下一个结点赋值给q；
    * 释放p；
    * 将q赋值给p；

<pre>
// 顺序表L已存在，操作结果：将L重置为空表
Status ClearList(LinkList *L)
{
  LinkList p, q;
  p = (*L)->next; // p指向第一个结点
  while (p) {
    q = p->next;
    free(p);
    p = q;
  }
  (*L)->next = null;
  return true;
}
</pre>

单链表结构与顺序存储结构优缺点
---------------------------------

存储分配方式
* 顺序存储结构用一段连续的存储单元依次存储线性表数据元素
* 单链表采用链式存储结构，用一组任意的存储单元存放线性表的元素

时间性能：
* 查找：顺序存储结构为O(1)，单链表为O(n);
* 插入和删除：顺序存储结构需要平均移动表长一半的元素，时间为O(n)，链式存储在找出指定位置的指针后，插入和删除的时间为O(1)；

空间性能
* 顺序存储结构需要预分配存储空间，不好把握大小
* 链式存储不需要分配存储空间，只要有就可以分配，元素个数也不受限制。

经验结论
* 需要频繁查找，很少进行插入和删除时，宜采用顺序存储结构

静态链表
-----------

使用数组描述单链表，首先数组的元素由数据域(存放数据信息)和游标(指针域，存放后继在数组中的下标)组成，这种用数组描述的链表叫做静态链表
<pre>
// 线性表的静态链表存储结构
#define MAXSIZE 1000;
typedef struct
{
  ElemType data;
  int cur; // 游标(Cursor)，为0时表示无指向
} Component, StaticLinkList[MAXSIZE];
</pre>

对数组第一个和最后一个元素做特殊处理，不存数据，未被使用的数组元素称为备用链表；第一个元素的cur存放备用链表第一个结点的下标；最后一个元素的cur存放第一个有数组的元素的下标；相对于单链表中的头结点。

<pre>
// 将一维数组space中各分量链成一备用链表
// space[0].cur为头指针，0表示空指针
Status InitList(StaticLinkList space)
{
  int i;
  for (i = 0; i < MAXSIZE - 1; i++) {
    space[i].cur = i + 1;
  }
  space[MAXSIZE - 1].cur = 0; // 当前静态链表为空，最后一个元素的cur为0

  return true;
}
</pre>

模拟存储空间的分配和和数据插入
<pre>
// 若备用空间链表非空，则返回分配的结点下标；否则返回false
int Malloc_SLL(StaticLinkList space)
{
  int i = space[0].cur; // 备用链的下标
  if (space[0].cur) {
    space[0].cur = space[i].cur; // 重新分配备用链下标
  }

  return i;
}

// 在L中第i个元素之前插入新的元素e
Status ListInsert(StaticLinkList L, int i, ElemType e)
{
  int j, k, l;
  k = MAX_SIZE = 1; // k首先是最后一个元素的下标
  if (i < 1 || i > ListLength(L) + 1) {
    return false;
  }
  j = Malloc_SSL(L); // 获取空闲分量下标
  if (j) {
    L[j].data = e; 
    for (l = 1; l <= i - 1; l++) { 
      k = L[k].cur;
    }
    L[j].cur = L[k].cur;
    L[k].cur = j;
    return true;
  }

  return false;
}
</pre>

循环链表
----------

将单链表中终端结点的指针端有空指针改为指向头结点，就使整个单链表形成一个环，这种头尾相接的单链表成为单循环链表，简称循环链表（circular linked list），可以从当中一个结点出发，访问链表的全部结点。

双向链表
----------

在单链表的每个结点中，再设置一个指向其前驱结点的指针域。双向链表也可以是循环表。用空间换时间。

<pre>
// 线性表的双向链表存储结构
typedef structDolNode
{
  ElemType data;
  struct DoLNode *prior;
  struct DoLNode *next;
} DulNode, *DuLinkList;

//p->next->prior = p = p->prior->next;

// 双向链表的插入
s->prior = p; // 把p赋值给s的前驱
s->next = p->next; // 把p->next赋值给s的后继
p->next->prior = s; // 把s赋值给p->next的前驱
p->next = s; // 把s赋值给p的后继

// 双向链表的删除
p->prior->next = p->next; // 把p->next赋值给p->prior的后继
p->next->prior = p->prior; // 把p->prior赋值给p->next的前驱
free(p); // 释放结点
</pre>

总结
----------

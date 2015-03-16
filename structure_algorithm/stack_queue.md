栈与队列
=========

栈的相关概念
------------

栈（stack）是限定只在表尾进行插入和删除操作的线性表，又成后进先出（Last In First Out，LIFO）线性表。

    * 运行插入和删除的一端成为栈顶（top），另一端成为栈底（bottom）；
    * 不含任何元素的栈成为空栈；
    * 栈的插入操作叫进栈、压栈或入栈，栈的删除操作叫出栈或弹栈； 


栈的抽象数据类型
-------------------

<pre>
ADT stack

Data
    同线性表，元素具有相同的类型，相邻元素具有前驱和后继关系

Operation
    InitStack (*S): 初始化操作，建立一个空栈S
    DestroyStack (*S): 若栈存在，则销毁它
    ClearStack (*S): 清空栈
    StackEmpty (*S): 若栈为空，返回true，否则返回false
    GetTop (S, *e): 若栈存在且非空，用e返回S的栈顶元素
    Push (*S, e): 若栈存在，插入新元素e到栈中，e为新的栈顶元素
    Pop (*S, *e): 删除栈顶元素，用e返回栈顶元素
    StackLength (S): 返回栈S的元素个数
endADT
</pre>

栈的顺序存储结构接实现
-------------------------

顺序栈的结构定义
<pre>
typedef int SElemType; // SElemType 类型根据实际情况而定，这里暂定为int
typedef struct
(
    SElemType data[MAXSIZE];
    int top;
) SqStack;
</pre>

顺序栈的进栈操作
<pre>
// 插入元素e为新的栈顶元素
Status Push(SqStack *S, SelemType e)
{
    if (S->top == MAXSIZE - 1) { // 栈满
        return false;
    }
    S->top++;
    S->data[S->top] = e;
    return true;
}
</pre>

顺序栈的出栈操作
<pre>
// 若栈不空，则删除S的栈顶元素，用e返回站定元素，同时返回true，否则返回false
Status Pop(SqStack *S, SElemType *e)
{
    if (S->top == -1) {
        return false;
    }

    *e = S->data[S->top];
    S->top--;
    return true;
}
</pre>

两栈共享空间
---------------

共享栈的结构
<pre>
typedef struct
{
  SElemTpe data[MAXSIZE];
  int top1; // 栈1的栈顶指针
  int top2; // 栈2的栈顶指针
} SqDoubleStack;
</pre>

共享栈的入栈操作
<pre>
// 插入元素额为新的栈顶元素
Status Push(SqDoubleStack *S, SElemType e, int stackNumber)
{
  if (S->top1 + 1 == S->top2) { // 栈满
    return false;
  }
  if (stackNumber == 1) {
    S->data[++S->top] = e; // 栈1入栈
  } elseif (stackNumber == 2) {
    S->data[--S->top2] = e;
  }

  return true;
}
</pre>

共享栈的出栈操作
<pre>
// 若栈不空，则删除S的栈顶元素，用e返回其值
Status Pop(SqDoubleStack *S, SElemType *e, int stackNmeber)
{
  if (stackNumber == 1) {
    if (S->top1 == -1) {
      return false; // 栈1为空
    }
    *e = S->data[S->top--]; 
  } elseif (stackNumber == 2) {
    if (S->top2 == MAXSIZE) {
      return false; // 栈2为空
    }
    *e = S->dataS->top2++];
  }

  return true;
}
</pre>

栈的链式存储结构及实现
--------------------------

栈的链式存储结构是在普通链表的基础上，链表的头部作为栈顶(链栈通常不需要头结点)，增加一些栈操作，简称链栈。
链栈的结构定义
<pre>
typedef struct StackNode
{
  SElemType data;
  struct StackNode *next;
} StackNode, *LinkStackPtr;

typedef struct LinkStack
{
  LinkStackPtr top;
  int count;
} LinkStack;
</pre>

链栈的入栈操作
<pre>
// 插入元素e作为新的栈顶元素
Status Push(LinkStack *S, SElemType e)
{
  LinkStackPtr s = (LinkStackPtr) malloc(sizeof(StackNode));
  s->data = e;
  s->next = S->top; // s的指针域指向当前栈顶元素
  s->top = s; // 栈顶指针指向新元素
  S->count++；
  return true;
}
</pre>

链栈的出栈操作
<pre>
// 若栈不空，则删除S的栈顶元素，用e返回其值
Status Pop(LinkStack *S, SElemType *e)
{
  LinkStackPtr p;
  if (StackEmpty(*S) {
    return false;
  }
  p = S->top;
  *e = p->data;
  S->top = S->top->next;
  free(p);
  S->count--;
  return true;
}
</pre>

栈的作用
--------------

栈的引入简化了程序设计的问题，划分了不同关注层次，使得思考范围缩小，可以更集中于要解决的问题核心。因此许多高级语言都有对栈结构封装的功能模块，可以直接以栈的形式处理复杂的问题，同时不用考虑栈的实现细节。

栈的应用——递归
---------------

斐波那契数列常规迭代实现
<pre>
int main()
{
  int i;
  int a[40];
  a[0] = 0; 
  a[1] = 1;
  printf("%d ", a[0]);
  printf("$d ", a[1]);
  for (i = 2; i < 40; i++) {
    a[i] = a[i - 1] + a[i - 2];
    printf("%d ", a[i]);
  }

  return ;
}
</pre>

斐波那契数列的递归实现
<pre>
int Fbi(int i)
{
  if (i < 2) {
    return i == 0 ? 0 : 1;
  }
  return Fbi(i - 1) + Fbi(i - 2); // 自己调用自己
}

int main()
{
  int i;
  for (int i = 0; i < 40; i++) {
    printf("%d ", Fbi(i));
  }
  return ;
}
</pre>

把一个直接调用自己或通过一系列的调用语句间接地调用自己的函数称为递归函数。递归定义至少有一个条件，满足时不再进行递归。递归使用不当很容易造成时间和内存的损耗。

递归执行的前行过程中，产生一系列的数据，回归过程则有不断消耗这些数据。这些数据通常都是在栈中进行管理和维护的。

栈的应用——四则运算表达式求值
-------------------------------

后缀(逆波兰)表示法定义：四则运算表达式中，所有的符号都是在要运算数字的后面出现，并且没有了括号。

后缀表达式计算规则：从左到右遍历表达式的每个数字和符号，遇到数字就进栈，遇到符号，就将栈顶两个数字出栈进行运算，然后将预算结果进栈，直至预算结束。

中缀表达式(标准四则运算表达式)转换为后缀表达式的规则：从左到右遍历中缀表达式的每个数字和符号，若是数字就输出，成为后缀表达式的一部分，若是符号，则判断其与栈顶符号的优先级，是右括号或优先级低于栈顶符号(乘除优先加减)则栈顶元素依次出栈并输出，并将当前符号进栈，一直到最终输出后缀表达式。

基于栈实现的四则运算表达式求值的步骤
* 将中缀表达式转换为后缀表达式(栈用来进程运算的符号)；
* 将后缀表达式进行运算得出结果(栈用来进出运算的数子)；

队列的定义
------------

队列(Queue)是只允许在一端进行插入操作，在另一端进行删除操作的线性表，是一种先进先出(First In First Out)的线性表，简称FIFO，允许插入的一端叫队尾，允许删除的一端叫队头。

队列的抽象数据类型
--------------------

<pre>
ADT 队列(Queue)

Data
    同线性表，元素具有相同的类型，相邻元素具有前驱和后继关系

Operation
    InitQueue(*Q); // 初始化操作，建立一个空队列Q
    DestroyQueue(*Q); // 若队列Q存在，则销毁Q
    ClearQueue(*Q); // 将队列情况
    QueueEmpty(*Q); // 队列是否为空，为空返回true否则返回false
    GetHead(Q, *e); // 若Q存在且非空，用e返回Q的对头元素
    EnQueue(*Q, e); // 若Q存在，插入新元素e到队列Q中并成为队尾元素
    DeQueue(*Q, *e); // 删除Q中对头元素，并用e返回其值
    QueueLength(Q); // 返回队列的元素个数
endADT
</pre>

循环队列
--------------

同普通线性表一样，队列也可以使用顺序存储结构。使用普通的顺序存储结构，入队的时间复杂度为O(1)，出队的为O(n)；可以通过附加两个指针，用来指向对头(front)和队尾(rear)，但这样对存储空间的分配和使用依然不够灵活，如假溢出问题。

循环队列，把队列头尾连接的顺序存储结构成为循环队列。如何标识循环队列为空或已满？
* 设置一个标识变量，front=rear，且flag=0时循环队列为空，front=rear,flag=1时队列已满；
* 对头和队尾元素之间保留一个元素空间，即杜绝front=rear的情况，队列长度计算公式：(rear-front+QueueSize)%QueueSize；

<pre>
// 循环队列的顺序结构代码
typedef int QElemType; // 队列元素数据类型
typedef struct 
{
  QElemType data[MAXSIZE];
  int front; // 头指针
  int rear; // 尾指针
} SqQueue;

// 循环队列初始化
Status InitQueue(SqQueue *Q)
{
  Q->front = 0;
  Q->rear = 0;
  return true;
}

// 队列长度
int QueueLength(SqQueue Q)
{
  return (Q.rear - Q.front + MAXSIZE) % MAXSIZE;
}

// 循环队列的入队操作
Status EnQueue(SqQueue *Q, QElemType e)
{
  if ((Q->rear + 1) % MAXSIZE == Q->front) { // 队列已满
    return false;
  }
  Q->data[Q->rear] = e; // 将元素e赋值给队尾
  Q->rear = (Q->rear + 1) % MAXSIZE; // rear指针后移一位，若到最后，则转到数组头部

  return true;
}

// 循环队列出队操作
Status DeQueue(SqQueue *Q, QElemType *e)
{
  if (Q->front == Q->rear) { // 是否为空
    return false;
  }
  *e = Q->data[Q->front]; // 对头赋值给e
  Q->front = (Q->front + 1) % MAXSIZE; // front指针后移，若到最后，则转到数组头部

  return true;
}
</pre>

队列的链式存储结构及实现
---------------------------

队列的链式存储结构，就是线性表的单链表，只是只能尾进头出，为了方便头结点或头指针指向对头，增加一个尾指针指向队尾。为了区别普通链表，称为链队列。

链队列的结构
<pre>
typedef int QElemType; // 队列数据的类型

typedef struct QNode // 结点结构
{
  QElemType data;
  struct QNode *next;
} QNode, *QueuePtr;

typedef struct // 链队列结构
{ 
  QueuePtr front, rear; // 队头队尾指针
} LinkQueue;
</pre>

入队操作
<pre>
// 插入元素e作为Q的新的队尾元素
Status EnQueue(LinkQueue *Q, QElemType e)
{
  QueuePtr s = (QueuePtr)malloc(sizeof(QNode));
  if (!s) { // 存储分配失败
    return false;
  }

  s->data = e;
  s->next = null;
  Q->rear->next = s; // 之前的队尾元素的指针域指向s
  Q->rear = s; // 队尾指针指向s
  return true;
}
</pre>

出队操作
<pre>
// 若队列不空，删除Q的队头元素，用e返回其值
Status DeQueue(LinkQueue *Q, QElemType *e)
{
  QueuePtr p;
  if (Q->front == Q->rear) {
    return false;
  }
  p = Q->front->next; // 获取队头元素
  *e = p->data;
  Q->front->next = p->next; // 队头指针指向队头的下一个元素

  if (Q->rear == p) { // 若删除的元素同时是最后一个元素，将rear指向头结点
    Q->rear = Q->front;
  }
  free(p);
  return true;
}
</pre>

队列长度可以基本确定的环境下，考虑使用循环队列，否则使用链队列。

总结
-----

栈和队列是对插入和删除做了限制的线性表
栈（stack）是限定仅在表位进行插入和删除操作的线性表；
队列（queue）是只允许在一端进行插入，另一端进行删除的线性表；

使用顺序存储时
对于栈来说，如何分配栈空间的大小是个关键。如果是两个相同数据类型的栈，可以通过

栈与队列
=========

栈的相关概念
------------

栈（stack）是限定尽在表位进行插入和删除操作的线性表，又成后进先出（Last In First Out，LIFO）线性表。

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

栈的链式存储结构及实现
--------------------------

栈的作用
--------------

栈的引入简化了程序设计的问题，划分了不同关注层次，使得思考范围缩小，可以更集中于要解决的问题核心。因此许多高级语言都有对栈结构封装的功能模块，可以直接以栈的形式处理复杂的问题，同时不用考虑栈的实现细节。

栈的应用——递归
---------------

栈的应用——四则运算表达式求值
-------------------------------

队列的定义
------------

队列(Queue)是只允许在一端进行插入操作，在另一端进行删除操作的线性表，是一种先进先出(First In First Out)的线性表，简称FIFO，允许插入的一端叫队尾，允许删除的一端叫对头。

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

同普通线性表一样，队列也可以使用顺序存储结构。使用普通的顺序存储结构，入队的时间复杂度为O(1)，出队的为O(n)；可以通过附加连个指针，用来指向对头(front)和队尾(rear)，但这样对存储空间的分配和使用依然不够灵活，如假溢出问题。

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

总结
-----

栈和队列是对插入和删除做了限制的线性表
栈（stack）是限定仅在表位进行插入和删除操作的线性表；
队列（queue）是只允许在一端进行插入，另一端进行删除的线性表；

使用顺序存储时
对于栈来说，如何分配栈空间的大小是个关键。如果是两个相同数据类型的栈，可以通过

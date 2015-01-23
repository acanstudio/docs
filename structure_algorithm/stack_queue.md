栈与队列
=========

栈的相关概念
------------

栈（stack）是限定尽在表位进行插入和删除操作的线性表，又成后进先出（Last In First Out，FIFO）线性表。

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

栈的应用——递归
---------------

栈的应用——四则运算表达式求值
-------------------------------

队列的定义
------------

队列的抽象数据类型
--------------------

循环队列
--------------

队列的链式存储结构及实现
---------------------------

总结
-----

栈和队列是对插入和删除做了限制的线性表
栈（stack）是限定仅在表位进行插入和删除操作的线性表；
队列（queue）是只允许在一端进行插入，另一端进行删除的线性表；

使用顺序存储时
对于栈来说，如何分配栈空间的大小是个关键。如果是两个相同数据类型的栈，可以通过

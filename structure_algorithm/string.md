串
===

串的相关概念
----------

* 串（string）是由零个或多个字符组成的有限序列，又叫字符串；
* 串中字符数目n称为串的长度；
* 零个字符的串称为空城（null string）；
* 串的相邻字符之间具有前驱和后继的关系；
* 子串和主串：串中任意个数的连续字符组成的子序列称为该串的子串，包含子串的串即为主串；
* 子串在主串中的位置就是子串第一个字符在主串中的序号；
* 串的比较：串是基于组成串的字符的编码来进行比较的，字符的编码指的是字符在对应字符集中的符号；

串的抽象数据类型
-----------------

<pre>
ADT 串(string)

Data
    串中元素仅由一个字符组成，相邻元素具有前驱和后继关系

Operation
    StrAssign(T, *chars): 生成一个其值等于字符串常量chars的串
    StrCopy(T, S): 串S存在，由串S复制得串T
    ClearString(S): 串S存在，清空串S
    StringEmpty(S): 若串S为空，返回true，否则返回false
    StrLength(S): 返回串的元素个数，即串的长度
    StrCompare(S, T): 若S>T，返回值大于0，若相等返回0，若S<T，返回值小于0
    Concat(T, S1, S2): 用T返回由S1和S2连接而成的新串
    SubString(Sub, S, pos, len): 串S存在， 1<=pos<=StrLength(S), 0<=len<=StrLength(S) - pos + 1，用Sub返回串S的第pos个字符起长度为len的子串
    Index(S, T, pos): 串S和T存在，T是非空串，1<=pos<=StrLength(S)，若主串S中存在和串T值相同的子串，则返回他在主串S中的第pos个字符之后第一次出现的位置，否则返回false
    Replace(S, T, V): 串S、T、V存在，T非空，用V串替换S中出现的所有与T串相等的不重叠的子串
    StrInsert(S, pos, T): 串S和T存在，1<=pos<=StrLength(S) + 1，在串S的第pos个字符之前插入串T
    StrDelete(S, pos, len): 串S存在，1<=pos<=StrLength(S) - len + 1， 从串S中删除第pos个字符起长度为len的子串

endADT
</pre>

检索主串中指定子串的位置的算法

<pre>
// T为非空串，若主串S中第pos个字符之后存在与T串相等的子串
// 则返回pos，否则返回false
int Index(String S, String T, int pos)
{
  int n, m, i;
  String sub;
  if (pos > 0) {
    n = StrLength(S);
    m = StrLength(T);
    i = pos;
    while (i <= n-m+1) {
      SubString(sub, S, i, m);
      if (StrCompare(sub, T) != 0) {
        ++i;
      } else {
        return i;
      }
    }
  }
  return false;
}
</pre>

串的存储结构
-------------

串的存储结构与普通的线性表类似，也分顺序存储和链式存储两种。

朴素的模式匹配算法
-------------------

* 字串的定位操作称作串的模式匹配

<pre>
// 返回字串T在主串S中第pos个字符之后的位置，若不存在，返回false
// T非空，1<=pos<=StrLength(S)
int Index(String S, String T, int pos)
{
    int i = pos; // i用于主串S中当前位置下标
    int j = 1; // j用于字串T中当前位置下标

    while (i <= S[0] && j <= T[0]) { // 若i小于S长度且j小于T的长度是开始循环
        if (S[i] == T[j]) {
            ++i;
            ++j;
        } else {
            i = i - j + 2; // i 退回到上次匹配首位的下一位
            j = 1; // j退回到字串的首位
        }
    }

    if (j > T[0]) {
        return i - T[0];
    } else {
        return false;
    }
}
</pre>

模式匹配算法PHP版

<pre>
<?php
$string = '00000000000000000000001';
$find = '0002';
$result = search($string, $find);
var_dump($result);

function search($string, $find, $position = 0)
{
    $len = strlen($string);
    $len1 = strlen($find);
    echo $len . '--' . $len1 . '<br />';
    $i = $position;
    $j = 0;
    while ($i <= $len - 1 && $j <= $len1 - 1 && $len - $i >= $len1) { // 若i小于S长度且j小于T的长度是开始循环
        echo $i . '===' . $j;

        if ($string{$i} == $find{$j}) {
            echo '--yes<br />';
            ++$i;
            ++$j;
        } else {
            echo '--no<br />';
            $i = $i - $j + 1; // i 退回到上次匹配首位的下一位
            $j = 0; // j退回到字串的首位
        }
    }
    echo $i . 'i<br />';
    echo $j . 'j<br />';
    if ($j >= $len1) {
        return $i - $len1;
    } else {
        return false;
    }
}
</pre?

KMP模式匹配算法
----------------

<pre>
// 通过计算返回子串T的next数组
void get_next(String T, int *next)
{
  int i, j;
  i = 1;
  j = 0;
  nxt[1] = 0;
  while (i < T[0]) { // T[0]表示串T的长度
    if (j == 0 || T[i] == T[j]) { // T[i]后缀的单个字符，T[j]前缀的单个字符
      ++i;
      ++j;
      next[i] = j;
    } else {
      j = next[j]; // 若字符不相同，则j值回溯
    }
  }
}

// 返回子串T在主串S中第pos个字符之后的位置，若不存在返回false
// T非空， 1<=pos<=StrLength(S)
int Index_KMP(String S, String T, int pos)
{
  int i = pos; // i用于主串S当前位置下标值，若pos不为1，则从pos位置开始匹配
  int j = 1; // j用于子串T中当前位置下标值
  int next[255]; 
  get_next(T, next);

  while (i <= S[0] && j <= T[0]) { // 若i小于S的长度且j小于T的长度
    if (j == 0 || S[i] == T[j]) { // 相等则继续，增加了j==0的判断
      ++i;
      ++j;
    } else {
      j = next[j]; // j退回合适的位置，i值不变
    }
  }

  if (j > T[0]) {
    return i - T[0];
  } else {
    return false;
  }
}
</pre>

总结
--------

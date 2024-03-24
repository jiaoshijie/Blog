---
title: "阅读 Linux 核心源代码 lib/list_sort.c"
date: 2024-03-23T14:33:19+08:00
tags: ["linux", "c"]
categories: ""
author: "Jiao shijie"
draft: true
hidemeta: true
math: true
---

Linux 核心代码中，对链表的排序算法是一种归并排序的变种。采用的排序方式为自下而上
的排序，这种方式可以避免 [cache thrashing](https://en.wikipedia.org/wiki/Thrashing_(computer_science)#Cache_thrashing)。

## 合并时机

Linux 中的对归并排序的改进主要是改变了两个链表的合并时机。该算法会有一个 `pending`
链表用来记录排序好的子链表，且保证每个子链表的长度都为 $2^k$。当 `pending` 中
即将出现第三个 $2^k$ 长度的链表时，就会合并已存在的两个 $2^k$ 长度的链表使其
变为一个 $2^{k+1}$ 长度的链表，所以在 `pending` 链表中永远不会存在超过 2 个
长度为 $2^k$ 的链表。

使用一个变量 `count` 来追踪 `pending` 中 $2^k$ 长度链表的个数，可以通过 `count`
中 `k+1` `k` `k-1` 这 3 个 bits 来确定 `pending` 中 $2^k$ 长度链表的个数。下表
中的 3 个 bits 分别表示 `k+1` `k` `k-1`，当 `k` 为 0 时，认为 `-1` bit 为 1。

| count     | $2^k$ 长度链表个数 |
| ----      | ----               |
| ...000... | 0                  |
| ...001... | 0                  |
| ...010... | 0                  |
| ...011... | 1                  |
| ...100... | 1                  |
| ...101... | 2                  |

## 源码分析

```c
__attribute__((nonnull(2,3)))
void list_sort(void *priv, struct list_head *head, list_cmp_func_t cmp)
{
  struct list_head *list = head->next, *pending = NULL;
  size_t count = 0;  /* Count of pending */

  if (list == head->prev)  /* Zero or one elements */
    return;

  /* Convert to a null-terminated singly-linked list. */
  head->prev->next = NULL;

  // ....
}
```

函数的参数：

- `void *priv` 最终会传入到 `cmp` 中，其它地方用不到。
- `struct list_head *head` 是一个双向循环链表
- `list_cmp_func_t cmp` 是一个函数指针，合并时用来比较的函数。

这段代码主要是初始化了 `list` 和 `pending` 两个变量，然后将 `head` 转化成了
单链表，在后续的排序过程中，遍历链表时只会用到 `next` 指针，不需要 `prev`指针。

```c
void list_sort(/* ... */) {
  // ....

  do {
    size_t bits;
    struct list_head **tail = &pending;

    /* Find the least-significant clear bit in count */
    for (bits = count; bits & 1; bits >>= 1)
      tail = &(*tail)->prev;
    /* Do the indicated merge */
    if (likely(bits)) {
      struct list_head *a = *tail, *b = a->prev;

      a = merge(priv, cmp, b, a);
      /* Install the merged result in place of the inputs */
      a->prev = b->prev;
      *tail = a;
    }

    /* Move one element from input list to pending */
    list->prev = pending;
    pending = list;
    list = list->next;
    pending->next = NULL;
    count++;
  } while (list);

  // ....
}
```

代码中的 `likely` 为宏，定义如下：

```c
#define likely(x)    (__builtin_expect(!!(x), 1))
```

> [__builtin_expect](https://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html)
>
> You may use __builtin_expect to provide the compiler with branch prediction
> information.

这段代码是将 `list` 中的每一个结点都移到 `pending` 中，并实时对 `pending` 中
链表进行合并来保证不会出现两个以上的长度为 $2^k$ 的链表。

下面以 16 个结点的 `list` 举例，来观察 `pending` 中每个链表的变化情况。

| count | binary | 由上一步到这是否需要合并 | pending                       |
| ----  | ----   | ----                     | ----                          |
| 0     | 0000   | 否                       | NULL                          |
| 1     | 0001   | 否                       | NULL <- 1                     |
| 2     | 0010   | 否                       | NULL <- 1 <- 1                |
| 3     | 0011   | 是                       | NULL <- 2 <- 1                |
| 4     | 0100   | 否                       | NULL <- 2 <- 1 <- 1           |
| 5     | 0101   | 是                       | NULL <- 2 <- 2 <- 1           |
| 6     | 0110   | 是                       | NULL <- 4 <- 1 <- 1           |
| 7     | 0111   | 是                       | NULL <- 4 <- 2 <- 1           |
| 8     | 1000   | 否                       | NULL <- 4 <- 2 <- 1 <- 1      |
| 9     | 1001   | 是                       | NULL <- 4 <- 2 <- 2 <- 1      |
| 10    | 1010   | 是                       | NULL <- 4 <- 4 <- 1 <- 1      |
| 11    | 1011   | 是                       | NULL <- 4 <- 4 <- 2 <- 1      |
| 12    | 1100   | 是                       | NULL <- 8 <- 2 <- 1 <- 1      |
| 13    | 1101   | 是                       | NULL <- 8 <- 2 <- 2 <- 1      |
| 14    | 1110   | 是                       | NULL <- 8 <- 4 <- 1 <- 1      |
| 15    | 1111   | 是                       | NULL <- 8 <- 4 <- 2 <- 1      |
| 16    | 10000  | 否                       | NULL <- 8 <- 4 <- 2 <- 1 <- 1 |

只有当 `count` 为 0 或二进制只有一个 1 时才不会合并否则都会进行合并。当 `list`
为 `NULL` 时 `do-while` 循环会结束。进入下面这段代码：

```c
void list_sort(/* ... */) {
  // ....

  /* End of input; merge together all the pending lists. */
  list = pending;
  pending = pending->prev;
  for (;;) {
    struct list_head *next = pending->prev;

    if (!next)
      break;
    list = merge(priv, cmp, pending, list);
    pending = next;
  }

  // ....
}
```

这段代码会对 `pending` 中的链表长度由小到大进行合并，最终会得到两个链表 `list`
和 `pending`，且长度之比不会大于 $2:1$。

接下来就是执行最终的合并，并将其再转换回双向循环链表。

```c
void list_sort(/* ... */) {
  // ....

  /* The final merge, rebuilding prev links */
  merge_final(priv, cmp, head, pending, list);

  // ....
}

__attribute__((nonnull(2,3,4,5)))
static void merge_final(void *priv, list_cmp_func_t cmp, struct list_head *head,
      struct list_head *a, struct list_head *b)
{
  struct list_head *tail = head;
  u8 count = 0;

  for (;;) {
    /* if equal, take 'a' -- important for sort stability */
    if (cmp(priv, a, b) <= 0) {
      tail->next = a;
      a->prev = tail;
      tail = a;
      a = a->next;
      if (!a)
        break;
    } else {
      tail->next = b;
      b->prev = tail;
      tail = b;
      b = b->next;
      if (!b) {
        b = a;
        break;
      }
    }
  }

  /* Finish linking remainder of list b on to tail */
  tail->next = b;
  do {
    if (unlikely(!++count))
      cmp(priv, b, b);
    b->prev = tail;
    tail = b;
    b = b->next;
  } while (b);

  /* And the final links to make a circular doubly-linked list */
  tail->next = head;
  head->prev = tail;
}
```

`merge_final` 中的代码 `for-loop` 是对两个链表进行合并，并同时将其设置为
双向链表。`do-while` 是当链表中有一个为空后，将另一个不为空的链表转换成上
双向链表。最后的两行代码为将其转化为双向循环链表。其中的 `unlikely` 宏定义如下，
是用来告诉编译器这个条件大概率为 0，可以对其进行优化。

```c
#define unlikely(x)    (__builtin_expect(!!(x), 0))
```

在 `do-while` 中其实完全没有必要进行比较，只是如果合并的这两个链表非常的不平衡
(就是一个链表值都很大，一个链表值都很小，但都是排好序的)，这样 `do-while` 会跑
很多轮，而这个比较函数为用户传递的，可以在其中使用 `cond_resched()` 宏来通知
调度器来将自己调度出去，避免总是占着 CPU 的执行。而 `count` 是一个 `u8` 的类型，
这个 `cmp` 只有当 `count` 溢出时才会执行一次，因此使用 `unlikely` 来告诉编译器
可以尝试对其进行优化。

## 性能分析

- [ ] TODO

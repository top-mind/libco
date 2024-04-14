# Notes

This project is a part of Operating System (Lab) lectured by JYY (jyy@jyywiki.cn)

see [jyywiki.cn/OS/2024](https://jyywiki.cn/OS/2024) for more information

## 创建 coroutine 和栈切换时机两种思路，以及 main 函数协程化 hack

1. 像代码中展示的, 把栈和协程绑定在一起, 一旦分配了协程,
就切换到新的栈上保存上下文再回来.

2. 栈和协程解耦，等到协程被调度器选中再切换栈。这样 main 函数可以看作
CO\_RUN 状态的协程，而不是看作特殊的执行流。

## co\_wait 的三种实现思路

1. 自旋直到 waitee 退出

2. 建议调度器忽视自己，waitee 退出时唤醒自己

3. 让调度器忽视自己，waitee 退出时唤醒自己

解法 2 是最通用的，它把这个过程抽象成了 co\_spin 和 co\_signal。

## 公平高效的调度器

一个调度器是指下面的抽象数据结构，支持：

1. 插入 删除

2. 查找

3. 随机访问

索引跳表 ([Indexable skiplist](https://en.wikipedia.org/wiki/Skip_list#Indexable_skiplist)) 是不错的选择.

## 死锁

你有没有想过，下面的测试代码会有怎样的行为，如果用 pthread 实现 coroutine (重量级协程库:)
pthread\_join 会怎么处理这种情况？

```cpp
  struct co *p;

  p = co_start(
      "deadlock",
      [](void *p) {
        co_wait(*(struct co **)p);
      },
      &p);

  co_wait(p);
```

你能解决这个问题吗？这个问题的答案就藏在前一个问题中哦。

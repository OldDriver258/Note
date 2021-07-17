# 一个实际的 schedule() 函数

## counter 的作用：时间片
在时钟的中断里做 counter--
``` C
void do_timer(long cpl)
{
    if ((--current->counter)>0) return;
	current->counter=0;
	if (!cpl) return;                       // 内核态程序不依赖counter值进行调度
	schedule();
}
```
每经过一个固定的时间，当前任务的 counter 都会在定时器计数中--
当当前任务的时间片用完之后，会调用调度函数。

## 调度
Linux 0.11的调度函数 schedule() 函数
``` C
while (1) {
    c = -1;
    next = 0;
    i = NR_TASKS;
    p = &task[NR_TASKS];
    // 这段代码也是从任务数组的最后一个任务开始循环处理，并跳过不含任务的数组槽。比较
    // 每个就绪状态任务的counter(任务运行时间的递减滴答计数)值，哪一个值大，运行时间还
    // 不长，next就值向哪个的任务号。
    while (--i) {
        if (!*--p) //跳过空的 PCB
            continue;
        if ((*p)->state == TASK_RUNNING && (*p)->counter > c)
            c = (*p)->counter, next = i;
    }
    // 如果比较得出有counter值不等于0的结果，或者系统中没有一个可运行的任务存在(此时c
    // 仍然为-1，next=0),则退出while(1)_的循环，执行switch任务切换操作。否则就根据每个
    // 任务的优先权值，更新每一个任务的counter值，然后回到while(1)循环。counter值的计算
    // 方式counter＝counter/2 + priority.注意：这里计算过程不考虑进程的状态。
    if (c) break;
    for(p = &LAST_TASK ; p > &FIRST_TASK ; --p)
        if (*p)
            (*p)->counter = ((*p)->counter >> 1) +
                    (*p)->priority;
}
// 用下面的宏把当前任务指针current指向任务号Next的任务，并切换到该任务中运行。上面Next
// 被初始化为0。此时任务0仅执行pause()系统调用，并又会调用本函数。
switch_to(next);     // 切换到Next任务并运行。
```
运行通过第一个while()之后。c 保存了当前就绪可运行的任务的 counter 最大值，if(c) 判断过后，通过的情况只有所有的就绪态任务 counter 都是0。如果有 counter > 0 表示还有就绪态的任务的时间片没有用完，将 break 退出调度函数。如果 counter == -1 表示当前没有处于就绪态的任务，将 break 退出调度函数。
break 退出之后如果是就绪态任务时间片没有用完，这时候 next 中是就绪态任务中 counter 最大的任务，切换到最大 counter 任务中执行。break 退出之后如果是没有就绪态任务，此时 next == 0，就会切换到0 号进程中去。

下面的循环就是重新分配每个任务的 counter 值，如果任务是处于 counter == 0 的任务，得到的新的时间片就是设置的优先级的大小。如果是处于阻塞状态的任务 counter＝counter/2 + priority ，保证了阻塞的进程在回来就绪的时候，counter一定比较大，优先级更高。那么越是进程阻塞的进程就导致 counter 越大，优先级也越高，一般经常阻塞的程序都是前台程序，这样就会动态调高前台程序的优先级。
时间片重新分配之后，会通过循环从上面重新开始检查就绪态和counter 值，这是优先级搞得任务会首先放到next 中，进行进程切换。

## counter 作用总结
### 保证了响应时间的届
$$
\begin{cases}
  c(t)=\frac{c(t-1)}{2}+p  \\
  c(0)=p  \\  
\end{cases}
$$
如果一个进程永远的阻塞下去, 时间片最长也只有 2p
$$
c(0)=p \\
c(1)=\frac{c(0)}{2}+p = p + \frac{p}{2} \\
c(2)=\frac{c(1)}{2}+p = p + \frac{p}{2} + \frac{p}{4} \\
\vdots \\
c(\infty) = p + \frac{p}{2} + \frac{p}{4} + \cdots \leq 2p \\
$$

## 照顾前台进程
经过 IO 操作后  ，counter会变大，IO 时间越长 counter 越大。

## 后台进程
基于 counter 进行轮转，短的进程一定会先完成，近似了 SJF 调度
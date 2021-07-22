# 实验内容
在 lunux0.11 中实现信号量，用信号量解决生产者和消费者问题
## 用信号量解决生产者消费者问题
Ubuntu 上编写"pc.c", 解决生产者消费者问题：
1. 建立一个生产者进程， N个消费者进程
2. 用文件建立一个共享缓冲区
3. 生产者进程依次向缓冲区写入整数 0， 1， 2， ...， M （M > 500）
4. 消费者进程从缓冲区读数，每次读一个，并将读出的数字从缓冲区删除， 然后将本进程 ID 和 读出数字到标准输出
5. 缓冲区同时最多只能保存 10 个数。

pc.c 中将会用到 sem_open()、sem_close()、sem_wait() 和 sem_post() 等信号量相关的系统调用，请查阅相关文档。
> 《UNIX 环境高级编程》是一本关于 Unix/Linux 系统级编程的相当经典的教程。如果你对 POSIX 编程感兴趣，建议买一本常备手边

## 实现信号量
实现一套符合 POSIX 规范的信号量的缩水版， 它的函数原型和标准并不完全相同， 只包含如下系统调用
``` C
sem_t *sem_open(const char *name, unsigned int value);
int    sem_wait(sem_t *sem);
int    sem_post(sem_t *sem);
int    sem_unlink(const char *name);
```
* sem_open() 创建一个信号量， 或者打开一个已经存在的信号量
  * sem_t 是信号量类型
  * name 是信号量的名字。不同的进程提供同样的 name 共享同一个信号量
  * value 是信号量的初值， 只有在新建信号量的时候才会有效， 其他情况下参数忽略
  * 返回信号量的唯一标识， 如果失败返回 NULL
* sem_wait() 就是信号量 P 原子操作。如果条件不满足，一直等待信号量。
  * 返回 0 表示成功， -1 表示失败
* sem_post() 是信号量 V 原子操作。如果有等待信号量的进程， 它会唤醒其中的一个
  * 返回 0 表示成功， -1 表示失败
* sem_unlink() 删除名称为 name 的信号量
  * 返回 0 表示成功， -1 表示失败
  
实现的代码在 kernel/sem.c 新建文件上实现。 通过 ubuntu 移植来的 pc.c 来测试自己实现的信号量。

# 信号量
Linux 的信号量秉持 POSIX 规范。
生产者-消费者问题基本结构
``` C 
Producer()
{
    // 生产一个产品 item;

    // 空闲缓存资源
    P(Empty);

    // 互斥信号量
    P(Mutex);

    // 将item放到空闲缓存中;
    V(Mutex);

    // 产品资源
    V(Full);
}

Consumer()
{
    P(Full);
    P(Mutex);

    //从缓存区取出一个赋值给item;
    V(Mutex);

    // 消费产品item;
    V(Empty);
}
```

# 多进程共享文件
Linux 通过 C 语言， 可以通过三种方法进行文件的读写：
* 使用标准 C 的 fopen(), fread(), fwrite(), fseek(), fclose();
* 使用系统调用 open(), read(), write(), lseek(), close();
* 通过内存镜像文件, 使用 mmap() 系统调用

fork() 函数调用成功后， 子进程会继承父进程拥有的大多数资源， 包括打开的文件。 子进程可以直接使用这些文件指针， 描述符， 句柄来访问文件。

使用标准 C 的文件操作函数要注意， 他们使用的是进程空间内的文件缓冲区， 父进程和子进程之间不共享这个缓冲区。 因此每次写操作之后必须 fflush() 将数据更新到磁盘， 其他进程才能读到数据。

# 终端也是临界资源
多个进程同时输出时， 终端也成为了一个临界资源， 所以要做好互斥保护。 printf() 之后， 信息保存在输出缓冲区中， 用 fflush(stdout) 可以确保数据送达终端。

# 原子操作、睡眠、唤醒
锁必然是一种原子操作，通过模仿 0.11 中的锁来实现信号量。
多进程对于磁盘的并发访问是一个需要锁的地方， Linux0.11 的基本处理方法是才内存中划出一块缓存，加速磁盘的访问。进程提出磁盘访问请求首先要到磁盘缓存中去找，如果有就直接返回；如果没有就申请一段磁盘缓存，向磁盘发送读写请求。请求发出后，首先要睡眠等待，因为磁盘读写很慢，需要让CPU执行其他进程。因此会有多个进程同时操作磁盘缓存，这里的磁盘缓存也需要考虑互斥问题，必然也用到了锁，睡眠和唤醒。
下面是从 kernel/blk_drv/ll_rw_blk.c 文件中取出的两个函数：
``` C
static inline void lock_buffer(struct buffer_head * bh)
{
    // 关中断
    cli();

    // 将当前进程睡眠在 bh->b_wait
    while (bh->b_lock)
        sleep_on(&bh->b_wait);
    bh->b_lock=1;
    // 开中断
    sti();
}

static inline void unlock_buffer(struct buffer_head * bh)
{
    if (!bh->b_lock)
        printk("ll_rw_block.c: buffer not locked\n\r");
    bh->b_lock = 0;

    // 唤醒睡眠在 bh->b_wait 上的进程
    wake_up(&bh->b_wait);
}
```
lock_buffer() 可以看出，访问锁通过开关中断来实现原子操作，阻止进程切换的发送。但这种方式不适用于多处理器环境中。不过对于 linux0.11 足够。sleep_on() 实现进程的睡眠, wake_up() 实现进程的唤醒， 他们的参数都是一个结构体指针—— struct task_struct *，即进程都睡眠或唤醒在该参数指向的一个进程 PCB 结构链表上。

sleep_on() 的功能是将当前进程睡眠在参数指定的链表上（注意，这个链表是一个隐式链表，详见《注释》一书）。wake_up() 的功能是唤醒链表上睡眠的所有进程。这些进程都会被调度运行，所以它们被唤醒后，还要重新判断一下是否可以继续运行。可参考 lock_buffer() 中的那个 while 循环。

# 实现信号量内核函数 
## sem.c
``` C
#include <sys/sem.h>
#include <errno.h>
#include <asm/segment.h>    /*  include get_fs_byte() */
#include <unistd.h>
#include <asm/system.h>     /*  include cli(), sti() */
#include <linux/kernel.h>

#define KSEM_SIZE   5   /* 表示系统信号量的大小 */

sem_t   ksem[KSEM_SIZE] = {
    {{'\0', }, 0, NULL, 0},
    {{'\0', }, 0, NULL, 0},
    {{'\0', }, 0, NULL, 0},
    {{'\0', }, 0, NULL, 0},
    {{'\0', }, 0, NULL, 0},
    /*name, value, wait, used*/
};

char    buf[NAME_SIZE];
/*
 * 字符串比较函数 
 */
int str_equ(const char *sa, const char *sb)
{
    int i;
    for (i = 0; sa[i] && sb[i]; i++)
        if (sa[i] != sb[i])
            return  (0);
    return  (1);
}
/*
 * 字符串拷贝函数 
 */
void str_cpy(char *dest,const char *src)
{
    int i;
    for (i=0; dest[i] = src[i]; i++);
}
/*
 * 打开信号量
 */
sem_t *sys_sem_open(const char *name, unsigned int value)
{
    int i;
    /*  get name */
    /*
     * 从用户态的name取到内核buf中
     */
    for (i = 0; buf[i] = get_fs_byte(name + i); i++); 

    /*  return sem had created*/
    for (i = 0; i < KSEM_SIZE; i++)
        if (ksem[i].used) {       
            if (str_equ(ksem[i].name, buf)) {   /*如果已经被使用，且查找到名称*/
                printk("old: buf: %s\n", buf);
                printk("old: %s, %d, %d, %d\n", ksem[i].name, ksem[i].value, ksem[i].wait, ksem[i].used);
                return  &ksem[i];               /*返回全局信号量的地址*/
            }
        }

    /*  cannot find a created one, create new sem*/
    for (i = 0; i < KSEM_SIZE; i++)
        if (!ksem[i].used) {                    /*还没有被使用*/
            str_cpy(ksem[i].name, buf);   
            ksem[i].value = value;
            ksem[i].wait  = NULL;
            ksem[i].used  = 1;                  /*初始化这个信号量参数*/
            printk("new: buf: %s\n", buf);
            printk("new: %s, %d, %d, %d\n", ksem[i].name, ksem[i].value, ksem[i].wait, ksem[i].used);
            return  &ksem[i];                   /*返回全局信号量的地址*/
        }

    /*  no sem free size*/
    return  (NULL);
}

int sys_sem_wait(sem_t *sem)
{
    cli();
    while (!sem->value)
        sleep_on(&sem->wait);
    sem->value--;
    sti();
    return  (0);
}

int sys_sem_post(sem_t *sem)
{
    cli();
    if (!sem->value)
        wake_up(&sem->wait);
    sem->value++;
    sti();
    return  (0);
}

int sys_sem_unlink(const char *name)
{
    int i;
    /*  get name */
    for (i = 0; buf[i] = get_fs_byte(name + i); i++);

    /* try to delete sem */
    for (i=0; i<KSEM_SIZE; i++)
        if (ksem[i].used)
            if (str_equ(ksem[i].name, buf)) {     /*找到要释放的名称*/
                ksem[i].used = 0;                 /*修改使用标记*/
                return  (0);
            }  

    return  (-ERROR);
}
```
sys_sem_wait(), sys_sem_post() 两个函数就是仿照磁盘读写缓冲的方式编写的临界态操作信号量值的函数。
## sem.h
``` C
/* add by jlb */
#ifndef _SEM_H
#define _SEM_H

/* include task_struct */
#include <linux/sched.h>

#define NAME_SIZE 10
/* add */
typedef struct { char name[NAME_SIZE]; unsigned int value; struct task_struct *wait; char used; } sem_t;
/* end */

#endif 
```

# 向系统加入内核调用
## 修改 unistd.h
``` C
/* add */
#include <sys/sem.h>
/* end */
...
/* add */
#define __NR_sem_open 72
#define __NR_sem_wait 73
#define __NR_sem_post 74
#define __NR_sem_unlink 75
/* end */
...
/* add */
sem_t *sem_open(const char *name, unsigned int value);
int sem_wait(sem_t *sem);
int sem_post(sem_t *sem);
int sem_unlink(const char *name);
/* end */
```

## 修改system_call.s
``` C
/* mod by jlb */
nr_system_calls = 76
```

## 修改sys.h
``` C
/* add */
extern int sys_sem_open();
extern int sys_sem_wait();
extern int sys_sem_post();
extern int sys_sem_unlink();
/* end */
...
fn_ptr sys_call_table[] = { sys_setup, sys_exit, sys_fork, sys_read,
sys_write, sys_open, sys_close, sys_waitpid, sys_creat, sys_link,
sys_unlink, sys_execve, sys_chdir, sys_time, sys_mknod, sys_chmod,
sys_chown, sys_break, sys_stat, sys_lseek, sys_getpid, sys_mount,
sys_umount, sys_setuid, sys_getuid, sys_stime, sys_ptrace, sys_alarm,
sys_fstat, sys_pause, sys_utime, sys_stty, sys_gtty, sys_access,
sys_nice, sys_ftime, sys_sync, sys_kill, sys_rename, sys_mkdir,
sys_rmdir, sys_dup, sys_pipe, sys_times, sys_prof, sys_brk, sys_setgid,
sys_getgid, sys_signal, sys_geteuid, sys_getegid, sys_acct, sys_phys,
sys_lock, sys_ioctl, sys_fcntl, sys_mpx, sys_setpgid, sys_ulimit,
sys_uname, sys_umask, sys_chroot, sys_ustat, sys_dup2, sys_getppid,
sys_getpgrp, sys_setsid, sys_sigaction, sys_sgetmask, sys_ssetmask,
sys_setreuid, sys_setregid, sys_sem_open, sys_sem_wait, sys_sem_post, sys_sem_unlink };
```

## 修改 makefile
``` Makefile
# mod by jlb
OBJS  = sched.o system_call.o traps.o asm.o fork.o \
    panic.o printk.o vsprintf.o sys.o exit.o \
    signal.o mktime.o sem.o
...
# mod by jlb
sem.s sem.o : sem.c ../include/sys/sem.h ../include/errno.h ../include/asm/segment.h \
  ../include/unistd.h ../include/linux/sched.h ../include/asm/system.h ../include/linux/kernel.h
```

## 编译内核，上传头文件
通过 make all 指令编译修改的内核， debug 编译的错误
将修改的 unistd.h, sys.h 和 新增的 sem.h 上传内核的库

# 编写应用程序
``` C
#define	__LIBRARY__
#include <unistd.h>		/*  include some constant*/
#include <fcntl.h>			/*  include printf() fflush()*/
#include <stdio.h>
#include <sys/sem.h>
/*
 * 添加系统调用
 */
_syscall0(int, fork)
_syscall0(int, getpid)
_syscall3(int, write, int, fildes, const char *, buf, off_t, count)
_syscall3(int, read,  int, fildes, char * , buf, off_t, count)
_syscall3(int, lseek, int, fildes, off_t, offset, int, origin)
_syscall0(int, sync)

_syscall2(sem_t *, sem_open, const char *, name, unsigned int, value)
_syscall1(int, sem_wait, sem_t *, sem)
_syscall1(int, sem_post, sem_t *, sem)
_syscall1(int, sem_unlink, const char *, name)

#define 	BUF_SIZE		10
#define 	FILE_BUF_SIZE	10
#define		PRODUCER_PATH	"./f"
#define		CONSUMER_NUM	10

int  main (void)
{
	int             i;
	int             pid;
	int             fd_p, fd_c;
	unsigned short  count			= 0;
	unsigned short	tmp				= 0;
	unsigned char   buf[BUF_SIZE] 	= {0}; /*读写缓冲*/
	char            empty_name[10]	= "empty";
	char            full_name[10]	= "full";
	char            mutex_name[10]	= "mutex";
	sem_t           *empty, *full, *mutex;

	fd_p = open(PRODUCER_PATH, O_CREAT | O_TRUNC | O_RDWR, 0666);
	fd_c = open(PRODUCER_PATH, O_CREAT | O_TRUNC | O_RDWR, 0666);

	sem_unlink(empty_name);
	sem_unlink(full_name);
	sem_unlink(mutex_name);

	empty   = sem_open(empty_name, FILE_BUF_SIZE);
	full    = sem_open(full_name, 0);
	mutex   = sem_open(mutex_name, 1);        /*创建信号量*/

	/*  Consumer 0~9*/
	for(i = 0; i < CONSUMER_NUM; i++) {       /*建立子进程*/
		if(!fork()) {
			pid = getpid();
			while(1) {
				sem_wait(full);                     /*等待缓冲区有数据*/
				sem_wait(mutex);                    /*互斥保护临界区*/

				read(fd_c, buf, sizeof(unsigned short));

				tmp = (unsigned short)buf[0] | ((unsigned short)buf[1] << 8);

				printf("Consumer %2d : %3d\n", pid, tmp); /*输出一个数*/
				fflush(stdout);                       /*输出需要冲刷缓冲区*/

				if(tmp % FILE_BUF_SIZE == 9)
					lseek(fd_c, 0, SEEK_SET);

				sem_post(mutex);
				sem_post(empty);                     /*表示增加缓冲区空位*/	
			}
			return  (0);
		}
	}
	
	/*  Producer 0*/
	pid = getpid();
	while (count <= 600)
	{
		sem_wait(empty);                      /*等待缓冲区空位*/
		sem_wait(mutex);                      /*互斥保护临界区*/

		buf[0] = (unsigned char)count;
		buf[1] = (unsigned char)(count >> 8);

		write(fd_p, buf, sizeof(unsigned short));
		sync();                               /*写文件需要落盘*/

		if(count % FILE_BUF_SIZE == 9)
			lseek(fd_p, 0, SEEK_SET);

		sem_post(mutex);
		sem_post(full);                     /*表示填写缓冲区*/

		count++;
	}
	while(1);
	
	return  (0);
}
```

## 实验问题
在 pc.c 中去掉所有与信号量有关的代码，再运行程序，执行效果有变化吗？为什么会这样？ 实验的设计者在第一次编写生产者——消费者程序的时候，是这么做的：
``` C
Producer()
{
    // 生产一个产品 item;

    // 空闲缓存资源
    P(Empty);

    // 互斥信号量
    P(Mutex);

    // 将item放到空闲缓存中;
    V(Mutex);

    // 产品资源
    V(Full);
}

Consumer()
{
    P(Full);
    P(Mutex);

    //从缓存区取出一个赋值给item;
    V(Mutex);

    // 消费产品item;
    V(Empty);
}
```
这样可行吗？如果可行，那么它和标准解法在执行效果上会有什么不同？如果不可行，那么它有什么问题使它不可行？
* 没有信号量，多线程的调度会导致取出的数据混乱和重复
* 这样可行， 但要注意在这个实验中 消费产品是用 printf 打印表示的，也需要临界区保护


# 实验结果
![](images/2021-07-21-08-43-58.png)

# 参考文献
> https://www.lanqiao.cn/courses/115 实验平台
> https://www.lanqiao.cn/courses/reports/1321123 实验报告
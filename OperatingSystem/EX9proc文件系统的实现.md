# proc 文件系统的实现

## 实验内容

在 linux 0.11 上实现 procfs(proc 文件系统) 内的 psinfo 节点。 读取此节点的内容时， 可以得到系统当前所有进程的状态信息。 例如， 用 **cat /proc/psinfo** 就会显示

```shell
$ cat /proc/psinfo
pid    state    father    counter    start_time
0    	1    	-1    		0    		0
1    	1    	0    		28    		1
4    	1    	1    		1    		73
3    	1    	1    		27    		63
6    	0    	4    		12    		817

$ cat /proc/hdinfo
total_blocks:    62000;
free_blocks:    39037;
used_blocks:    22963;
...
```

 ## proc 文件系统简介

正式的 Linux 内核实现了 procfs , 他是一个虚拟的文件系统， 通常被股再到 /proc 目录上， 通过虚拟文件和虚拟目录的方式提供访问系统参数的机会。

这些虚拟的文件和目录并没有真实的存在磁盘上， 而是内核中的各种数据的一种直观的标识。 虽然也是虚拟的， 但是他们都可以通过标准的系统调用 open 、 read 进行访问。

例如， **/proc/meminfo** 中包含内存使用的信息， 可以用 **cat** 指令显示其内容：

``` shell
$ cat /proc/meminfo
MemTotal:       384780 kB
MemFree:         13636 kB
Buffers:         13928 kB
Cached:         101680 kB
SwapCached:        132 kB
Active:         207764 kB
Inactive:        45720 kB
SwapTotal:      329324 kB
SwapFree:       329192 kB
Dirty:               0 kB
Writeback:           0 kB
……
```

其实， Linux 很多的系统命令也是通过读取 **/proc** 实现的。 例如 **uname -a**  的部分信息就是来自于 /proc/vsesion, 而 **uptime** 的部分信息来自于 **/proc/uptime** 和 **/proc/loadavg** 。

## 基本思路

Linux 是通过文件系统接口实现 `procfs`，并在启动时自动将其 mount 到 `/proc` 目录上。此目录下的所有内容都是随着系统的运行自动建立、删除和更新的，而且它们完全存在于内存中，不占用任何外存空间。Linux 0.11 还没有实现虚拟文件系统，也就是，还没有提供增加新文件系统支持的接口。所以本实验只能在现有文件系统的基础上，通过打补丁的方式模拟一个 `procfs`。Linux 0.11 使用的是 Minix 的文件系统，这是一个典型的基于 `inode` 的文件系统，《注释》一书对它有详细描述。它的每个文件都要对应至少一个 inode，而 inode 中记录着文件的各种属性，包括文件类型。文件类型有普通文件、目录、字符设备文件和块设备文件等。在内核中，每种类型的文件都有不同的处理函数与之对应。我们可以增加一种新的文件类型——proc 文件，并在相应的处理函数内实现 procfs 要实现的功能。

## 实验内容

### 新增文件类型

在 *include/sys/stat.h* 文件中定义了文件的类型和测试宏

``` C
#define S_IFMT  00170000

// 普通文件
#define S_IFREG  0100000

// 块设备
#define S_IFBLK  0060000

// 目录
#define S_IFDIR  0040000
/*Add*/
// PROC 文件类型
#define S_IFPROC 0030000
/*Add*/
// 字符设备
#define S_IFCHR  0020000
#define S_IFIFO  0010000
//……

// 测试 m 是否是普通文件
#define S_ISREG(m)      (((m) & S_IFMT) == S_IFREG)

// 测试 m 是否是目录
#define S_ISDIR(m)      (((m) & S_IFMT) == S_IFDIR)
/*Add*/
// 测试 PROC 文件类型
#define S_ISPROC(m)      (((m) & S_IFMT) == S_IFPROC)
/*Add*/
// 测试 m 是否是字符设备
#define S_ISCHR(m)      (((m) & S_IFMT) == S_IFCHR)

// 测试 m 是否是块设备
#define S_ISBLK(m)      (((m) & S_IFMT) == S_IFBLK)
#define S_ISFIFO(m)     (((m) & S_IFMT) == S_IFIFO)
```

* 定义一个类型宏 `S_IFPROC`，其值应在 `0010000` 到 `0100000` 之间，但后四位八进制数必须是 0（这是 `S_IFMT` 的限制，分析测试宏可知原因），而且不能和已有的任意一个 `S_IFXXX` 相同；
* 定义一个测试宏 `S_ISPROC(m)`，形式仿照其它的 `S_ISXXX(m)`；

### mknod() 支持新的文件类型

通过 `mknod() ` 建立 psinfo 节点 inode， 需要支持新添加的文件类型。修改 `fs/namei.c` 文件中的 `sys_mknod()` 函数中的一行代码：

``` C
if (S_ISBLK(mode) || S_ISCHR(mode) || S_ISPROC(mode))
     inode->i_zone[0] = dev;
// 文件系统初始化
```

### init 时建立根文件系统

内核的初始化全部在 `main()` 中完成， main() 在最后切换到用户态并且调用 `init()`，`init()` 第一件事情就是挂载根文件系统。`setup((void *) &drive_info);` , procfs 文件系统初始化应该在文件系统挂载之后

* 建立 `/proc` 目录；建立 `/proc` 目录下的各个结点。

* 建立目录和结点分别需要调用 `mkdir()` 和 `mknod()` 系统调用。因为初始化时已经在用户态，所以不能直接调用 `sys_mkdir()` 和 `sys_mknod()`。必须在初始化代码所在文件中实现这两个系统调用的用户态接口，即 API：

* ``` C
    _syscall2(int,mkdir,const char*,name,mode_t,mode)
    _syscall3(int,mknod,const char*,filename,mode_t,mode,dev_t,dev)
    ```

`mkdir()` 时 mode 参数的值可以是 “**0755**”（对应 `rwxr-xr-x`），表示只允许 root 用户改写此目录，其它人只能进入和读取此目录。

procfs 是一个只读文件系统，所以用 `mknod()` 建立 psinfo 结点时，必须通过 mode 参数将其设为只读。建议使用 `S_IFPROC|0444` 做为 mode 值，表示这是一个 proc 文件，权限为 **0444**（r--r--r--），对所有用户只读。

`mknod()` 的第三个参数 dev 用来说明结点所代表的设备编号。对于 procfs 来说，此编号可以完全自定义。proc 文件的处理函数将通过这个编号决定对应文件包含的信息是什么。例如，可以把 0 对应 psinfo，1 对应 meminfo，2 对应 cpuinfo。

``` C
//在 init/main.c
mkdir("/proc", 0755);
mknod("/proc/psinfo", S_IFPROC|0444, 0); 
mknod("/proc/hdinfo", S_IFPROC|0444, 1);
mknod("/proc/inodeinfo", S_IFPROC|0444, 2);
```

完成这些工作后编译， 启动， 执行指令 `ll /proc`

``` shell
# ll /proc
total 0
?r--r--r--   1 root     root              0 ??? ??  ???? psinfo
```

### 让系统可读 proc 文件

`open()` 没有变化，那么需要修改的就是 `sys_read()` 了。

``` C
int sys_read(unsigned int fd,char * buf,int count)
{
    struct file * file;
    struct m_inode * inode;
//    ……
    inode = file->f_inode;
    if (inode->i_pipe)
        return (file->f_mode&1)?read_pipe(inode,buf,count):-EIO;
    if (S_ISCHR(inode->i_mode))
        return rw_char(READ,inode->i_zone[0],buf,count,&file->f_pos);
    if (S_ISBLK(inode->i_mode))
        return block_read(inode->i_zone[0],&file->f_pos,buf,count);
    if (S_ISDIR(inode->i_mode) || S_ISREG(inode->i_mode)) {
        if (count+file->f_pos > inode->i_size)
            count = inode->i_size - file->f_pos;
        if (count<=0)
            return 0;
        return file_read(inode,file,buf,count);
    }
	/*Add*/
    if (S_ISPROC(inode->i_mode))
        return proc_read(inode->i_zone[0], &file->f_pos, buf, count);
    /*Add*/
    printk("(Read)inode->i_mode=%06o\n\r",inode->i_mode);    //这条信息很面善吧？
    return -EINVAL;
}
```

显然，要在这里一群 if 的排比中，加上 `S_IFPROC()` 的分支，进入对 proc 文件的处理函数。需要传给处理函数的参数包括：

- `inode->i_zone[0]`，这就是 `mknod()` 时指定的 `dev` ——设备编号
- `buf`，指向用户空间，就是 `read()` 的第二个参数，用来接收数据
- `count`，就是 `read()` 的第三个参数，说明 `buf` 指向的缓冲区大小
- `&file->f_pos`，`f_pos` 是上一次读文件结束时“文件位置指针”的指向。这里必须传指针，因为处理函数需要根据传给 `buf` 的数据量修改 `f_pos` 的值。

### proc.c

``` C
#include <linux/kernel.h>
#include <linux/sched.h>
#include <asm/segment.h>
#include <linux/fs.h>
#include <stdarg.h>
#include <unistd.h>

#define set_bit(bitnr, addr) ({\
        register int __res;\
        __asm__("bt %2,%3;setb %%al":"=a"(__res):"a"(0),"r"(bitnr),"m"(*(addr)));\
        __res;    \
    })
    
char proc_buf[4096]={'\0'};
extern int vsprintf(char *buf, const char* fmt, va_list args);
int sprintf(char *buf, const char *fmt, ...)
{
    va_list args;
    int i;
    /*
     * va_start() 和 va_end() 成对使用
     * fmd 是已知的最后一个参数
     */
    va_start(args, fmt);		
    i=vsprintf(buf, fmt, args);	//将参数args按照fmt，发送到buf上
    va_end(args);
    return i;
}
int get_psinfo()
{
    int read=0;
    read += sprintf(proc_buf+read, "%s", "pid\tstate\tfather\tcounter\tstart_time\n");
    struct task_struct **p;
    /*
     * 从第一个到最后一个拿到 pcb， 打印 pcb 中的信息
     */
    for(p = &FIRST_TASK; p <= &LAST_TASK; ++p)
    {
        if(*p != NULL)
        {
            read += sprintf(proc_buf, "%d\t", (*p)->pid);
            read += sprintf(proc_buf, "%d\t", (*p)->state);
            read += sprintf(proc_buf, "%d\t", (*p)->father);
            read += sprintf(proc_buf, "%d\t", (*p)->counter);
            read += sprintf(proc_buf, "%d\n", (*p)->start_time);
        }
    }
    return read;
}

int get_hdinfo()
{
    int read=0;
    int i, used;
    struct super_block * sb;
    sb=get_super(0x301);	//获得超级块
    read += sprintf(proc_buf+read, "Total blocks: %d\n", sb->s_nzones);
    used = 0;
    i = sb->s_nzones; 		//逻辑块数
    while(--i >= 0){		//检测每个块的位图
        if(set_bit(i&8191, sb->s_zmap[i>>13]->b_data)){
            used++;
        }
    }
    read += sprintf(proc_buf+read, "Used blocks:  %d\n", used);
    read += sprintf(proc_buf+read, "Free blocks:  %d\n", sb->s_nzones-used);
    read += sprintf(proc_buf+read, "Total inodes: %d\n", sb->s_ninodes);
    used = 0;
    i = sb->s_ninodes+1;
    while(--i){				//检测每个inode的位图
        if(set_bit(i&8191, sb->s_imap[i>>13]->b_data)){
            used ++;
        }
    }
    read += sprintf(proc_buf+read, "Used inodes: %d\n", used);
    read += sprintf(proc_buf+read, "Free inodes: %d\n", sb->s_ninodes-used);
    return read;
}
int get_inodeinfo()
{
    int read=0;
    int i;
    struct super_block * sb;
    struct m_inode * mi;
    sb = get_super(0x301);
    i=sb->s_ninodes+1;
    i=0;
    while(++i < sb->s_ninodes+1){
        if(set_bit(i&8191, sb->s_imap[i>>13]->b_data)){
            mi=iget(0x301, i);
            read += sprintf(proc_buf+read, "inr:%d;zone[0]:%d\n", mi->i_num, mi->i_zone[0]);
            input(mi);
        }
        if(read >= 4000){
            break;
        }
    }
    return read;
}
int proc_read(int dev, unsigned long * pos, char * buf, int count)
{
    int i;
    if(*pos % 1024 == 0)
    {
        if(dev==0){
            get_psinfo();
        }
        if(dev==1){
            get_hdinfo();
        }
        if(dev==2){
            get_inodeinfo();
        }
    }
    for(i=0; i<count; i++){
        if(proc_buf[i + *pos]  == '\0'){
            break;
        }
        put_fs_byte(proc_buf[i + *pos], buf + i + *pos);
    }
    *pos += i;
    return i;
}
```

### 实验结果

![image-20210822221444166](K:\git_mynote\mynote\OperatingSystem\images\image-20210822221444166.png)

### 问题

1. 如果要求你在 `psinfo` 之外再实现另一个结点，具体内容自选，那么你会实现一个给出什么信息的结点？为什么？**进程打开哪些文件的信息**
2. 一次 `read()` 未必能读出所有的数据，需要继续 `read()`，直到把数据读空为止。而数次 `read()` 之间，进程的状态可能会发生变化。你认为后几次 `read()` 传给用户的数据，应该是变化后的，还是变化前的？ + 如果是变化后的，那么用户得到的数据衔接部分是否会有混乱？如何防止混乱？ + 如果是变化前的，那么该在什么样的情况下更新 `psinfo` 的内容？**变化之前的, 在读取位置f_pos为0时才更新psinfo内容.也就是相当于仅仅打印的都是变化之前那一次的，只有 read 全部读完之后才回去重新跟新 psinfo的内容**



# 参考资料

> 实验环境 https://www.lanqiao.cn/courses/115/learning?id=575
>
> 关于 procfs 更多的信息 http://en.wikipedia.org/wiki/Procfs 
>
> 实验报告 https://www.lanqiao.cn/courses/reports/1301418
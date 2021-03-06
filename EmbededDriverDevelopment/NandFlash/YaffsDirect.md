[TOC]
# 什么是yaffs

Yaffs (Yet Another Flash File System) 是一种为了Nand存储设计的文件系统。尽管原本设计是为了Nand flash，但是也可以成功的使用在Nor flash，甚至Ram 文件系统。Yaffs可以作为系统的个性化模块这个模块就是Yaffs Direct Interfave（YDI），可以容易的与嵌入式系统和RTOS相结合。Yaffs2支持大量的Nand flash元件，包括 2k page 器件和 MLC（multi-level cells）器件。Yaffs通过RAM仿真层来做RAM文件系统，不过效率不如之间的RAM文件系统，不过提供了一种同时使用 flash 和 RAM 文件系统的方法。

# 特性
* 被大量使用在不同的操作系统，编译器，处理器上。
* 通过 C 语言编写，并且端中性 。
* 提供坏块处理和ECC算法。
* yaffs是一个log结构的文件系统，不用担心掉电造成巨大影响。
* 高度优化和可预测垃圾收集测量，提高性能。
* yaffs相比其他log结构的文件系统占用更少的内存
* 提供了大面积的 POSIX-style 文件系统
* yaffs可配置不同的flash结构，ECC选项······
* Yaffs Direct Interface 可以简单地加入一个系统中
* 可以和其他的内存技术一起使用

# 系统要求
* yaffs端中性，可以同时工作在大端和小端的处理器上
* yaffs支持多种32位和64位CPU，包括：MIPS，68000，ARM，ColdFire，PowerPC，x86，甚至DSP架构。也可以用在16位CPU，不过没有测试过，可能需要一些调整。
* yaffs作为log结构，需要RAM来构建数据结构。根据经验，每一个页要占用2byte内存。
  
| 存储空间 | 每页大小 | 页数 | yaffs占用内存大小       |
| -------- | -------- | ---- | ----------------------- |
| 1M       | 512byte  | 2048 | 2byte*2048 = **4kbyte** |
| 1M       | 2048byte | 512  | 2byte*512 = **1kbyte**  |

# 怎样移植Yaffs
Yaffs Direct Interface(YDI)提供了整合的方法。需要为yaffs提供一些函数来使用硬件和RTOS。分为三个部分
* POSIX 应用接口：POSIX 应用接口来方位文件系统(open, close, read, write, etc)。
* RTOS 整合接口：提供这些方法来让 Yaffs 访问 RTOS 的资源(initialise, lock, unlock, get time, memory allocation, set error, etc)。
* Flash 器件接口：这个结构是安装 flash 设备和适当设备的地方。

# 源文件

## 核心代码
| **源文件**                              | **说明**                                                  |
| --------------------------------------- | --------------------------------------------------------- |
| yaffs_allocator.c                       | Allocates Yaffs object and tnode structures.              |
| yaffs_bitmap.c                          | Block and chunk bitmap handling code.                     |
| yaffs_checkpointrw.c                    | Streamer for writing checkpoint data                      |
| yaffs_ecc.c                             | 1-bit Hamming ECC code                                    |
| yaffs_guts.c                            | The major Yaffs algorithms.                               |
| yaffs_nameval.c                         | Name/value code for handling extended attributes (xattr). |
| yaffs_nand.c                            | Flash interfacing abstraction.                            |
| yaffs_packedtags1.c yaffs_packedtags2.c | Tags packing code                                         |
| yaffs_summary.c                         | Code for handling block summaries.                        |
| yaffs_tagscompat.c                      | Tags compatibility code to support Yaffs1 mode.           |
| yaffs_tagsmarshall.c                    | Tags marshalling code.                                    |
| yaffs_verify.c                          | Yaffs1-mode specific code.                                |
| yaffs_yaffs2.c                          | Yaffs2-mode specific code.                                |

## Yaffs direct interface
| 源代码          | 说明                                               |
| --------------- | -------------------------------------------------- |
| yaffs_attribs.c | Attribute handling.                                |
| yaffs_error.c   | Error reporting code.                              |
| yaffsfs.c       | Yaffs Direct Interface wrapper code.               |
| yaffs_hweight.c | Counts the number of hot bits in a byte, word etc. |
| yaffs_qsort.c   | Qsort used during Yaffs2 scanning                  |

## flash器件、仿真和配置，在 direct/test-framework 目录下
| 源代码               | 说明                                                                                                                                                               |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| nanddrv.c            | A NAND driver layer that performs the commands to access NAND flash in a rudimentary manner. This code should work on many styles of CPU with little modification. |
| yaffs_nanddrv.c      | A wrapper around the NAND driver which plugs it into Yaffs.                                                                                                        |
| nandsim.c            | A NAND simulator layer that works with a nandstore_xxx backing store. This simulates a NAND chip.                                                                  |
| nandstore_file.c     | A storage backend for a NAND simulator that stores the data to file.                                                                                               |
| nandsim_file.c       | A wrapper around nandsim.c and nandstore_file.c which makes a NAND simulator that saves its data in a file.                                                        |
| yaffs_nandsim_file.c | A wrapper which creates a nand simulator instance, hooks up the yaffs NAND driver and then adds it to yaffs for use.                                               |
| yaffs_nor_drv.c      | A driver for CFI-style NOR flash.                                                                                                                                  |
| yaffs_m18_drv.c      | A driver for Intel M18-style NOR flash.                                                                                                                            |
| yaffs_osglue.c       | An OS glue layer example used in the test harness.                                                                                                                 |

## 例子和测试构建文件

| 源代码                          | 说明                                                                                                             |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| dtest.c                         | A test harness. Also has sample code that can be used for better  understanding of how some function calls work. |
| yaffscfg2k.c                    | A test configuration.                                                                                            |
| yaffs_fileem.c yaffs_fileem2k.c | Nand flash simulation using a file as backing store.                                                             |

## 其他的测试可以查看 direct/tests 和 direct/python

# POSIX 应用接口
yaffs的应用接口定义在 **yaffsfs.h** 中。
* 直接使用 yaffs提供的方法：`yaffs_open()`, `yaffs_close()`
* 通过宏定义：`#define open(path, oflag, mode) yaffs_open(path, oflag, mode)`
* 通过一个函数包装：`int open(const char *path,...){return yaffs_open(...);}`
* 通过操作系统的方法整合

# RTOS 整合接口
修改配置文件，例子在 yaffscfg.c 和 yaffscfg2k.c。
接口函数有：
* `void yaffsfs_SetError(int err)`: Called by Yaffs to set the system error.
* `void yaffsfs_Lock(void)`: Called by Yaffs to lock Yaffs from multi-threaded access.
* `void yaffsfs_Unlock(void)`: Called by Yaffs to unlock Yaffs.
* `u32 yaffsfs_CurrentTime(void)`: Get current time from RTOS.
* `void *yaffsfs_malloc(size_t size)`: Called to allocate memory.
* `void yaffsfs_free(void *ptr)`: Called to free memory.
* `void yaffsfs_OSInitialisation(void)`: Called to initialise RTOS context.
* `void yaffs_bug_fn(const char *file_name, int line_no)`: Function to report a
bug.
* `int yaffsfs_CheckMemRegion(const void *addr, size_t size, int write_request)` : Function to check if accesses to a memory region is valid. This function must return zero if the test passes and negative if it fails.

如果yaffs使用在多线程的环境中，通过`yaffs_LocalInitialisation()`来初始化RTOS信号量，`yaffs_Lock()`和`yaffs_Unlock()`会上锁和释放信号量。

# Nand 模块
* 每个**flash**由**block**组成。每一个block由整数个的chunk组成。
* 一个 **chunk** 是flash申请的单位。对于yaffs1一个chunk等于512-byte 数据和 16-byte 空余。对于yaffs2一个 chunk 支持2kbyte 数据和 64-byte 空余。
* 所有的访问都是以 page 为单位。
* nand的bit只能由1到0。

chunk的ID可以由一下计算得到：
**chunkId = block_id * chunks_per_block + chunk_offset_in_block**

## 格式化
yaffs通过擦除块来格式化。yaffs_format()函数。

## yaffs1 NAND 模型

## yaffs2 NAND 模型
* 没有典型的 spare area、ECC 布局。
* 没有标准的坏块标记实现。

| 函数接口            | 说明                              |
| ------------------- | --------------------------------- |
| drv_write_chunk_fn  | Write a data chunk                |
| drv_read_chunk_fn   | Read a data chunk.                |
| drv_erase_fn        | Erase a block                     |
| drv_mark_bad_fn     | Mark a block bad                  |
| drv_check_bad_fn    | Check bad block status of a block |
| drv_initialise_fn   | Initialisation.                   |
| drv_deinitialise_fn | De-initialise                     |

# 配置和flash驱动安装
1. 填写 paramers (dev->param)
2. 填写 driver (dev->drv)
3. yaffs_add_device() function添加设备

# POSIX文件系统接口
## 基本的概念
| 单词                          | 概念                                                                                                                                                                |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| File                          | 文件是文件系统中的一个对象。yaffs支持文件类型：**常规文件**存储了一系列的字节；**目录**连接到其他的文件；其他文件的**符号链接**；**特殊文件**拥有设备id和其他信息。 |
| Directory                     | 一个包含连接并且允许文件通过名字呗找到的结构。                                                                                                                      |
| Link                          | 一个可以找到对应文件的名字。分为硬链接和软链接两种。                                                                                                                |
| Handle                        | 提供访问和操作一个文件的上下文。                                                                                                                                    |
| File mode                     | 设置文件的种类和许可。                                                                                                                                              |
| Path                          | 表示文件和目录的字符串。                                                                                                                                            |
| File metadata                 | 除了文件内容的文件信息，包括名称，模式，大小和其他细节。                                                                                                            |
| Mount，partition，file system | 把flash空间挂载到一个普通的目录结构下面。                                                                                                                           |
| inode number                  | 用来描述挂载的对象。                                                                                                                                                |

## 错误码
错误码是从函数返回的一个整型，返回值小于0表示发生错误。

## 硬链接
文件是文件系统中存储对象，链接表示它怎样挂在一个目录结构中的。
第一个链接的创建：yaffs_open(),yaffs_mknod(), yaffs_symlink() and yaffs_mkdir()
链接的销毁：yaffs_unlink(),yaffs_rmdir()
链接重命名：yaffs_rename() 
多个链接创建：yaffs_link()

## 软链接
链接的销毁：yaffs_unlink()
软链接的特点：
1. 不存在文件的别名
2. 不在一个挂载点下面的文件的别名
3. 对目录创建一个软链接
4. 软链接不能保证一个文件存在。只有硬链接可以。

## 基于Handle的文件处理
一个文件的handle被定义为大于0的整型。
一个handle是由yaffs_open()接口返回的。如果返回的是一个负值说明yaffs返回一个错误码。
yaffs_open()可以带三个参数：
1. 要打开文件的路径名称
2. 访问的标志
3. 创建模式

访问标志包括：
| 标志     | 备注                                            |
| -------- | ----------------------------------------------- |
| O_CREAT  | 如果没有就创建这个文件                          |
| O_EXCL   | 和O_CREAT一起使用。如果已经存在文件则返回错误。 |
| O_TRUNC  | 删减文件长度位0字节                             |
| O_APPEND | 往文件的后面附加内容                            |
| O_RDWR   | 读写权限打开文件                                |
| O_WRONLY | 只写方式打开文件                                |
| 0 (zero) | 只读方式打开                                    |

创建的模式包括：
| 模式     | 备注   |
| -------- | ------ |
| S_IREAD  | 只读   |
| S_IWRITE | 只写   |
| S_IEXEC  | 可执行 |

通过Handle可以进行的操作：
| 调用                                         | 描述                               |
| -------------------------------------------- | ---------------------------------- |
| yaffs_close(handle)                          | 关闭handle，不能再使用             |
| yaffs_fsync(handle)                          | 数据落盘                           |
| yaffs_datasync(handle)                       | 数据落盘，不包括 metadata元数据    |
| yaffs_flush(handle)                          | 数据落盘同yaffs_fsync()            |
| yaffs_dup(handle)                            | 复制handle                         |
| yaffs_lseek(handle, offset,whence)           | 设置文件的读写位置                 |
| yaffs_ftruncate(handle, size)                | 改变文件的大小                     |
| yaffs_fstat(handle, buf)                     | 得到文件状态                       |
| yaffs_fchmod(handle, mode)                   | 改变文件模式                       |
| yaffs_read(handle, buffer,nbytes)            | 从文件指针读取数据，指针会自动增加 |
| yaffs_pread(handle, buffer,nbytes, position) | 从指定为位置读取数据               |
| yaffs_write(handle, buffer,nbytes)           | 从文件指针写入数据，指针会自动增加 |

## 改变文件大小
``` C
    yaffs_truncate(path, newSize)
    yaffs_ftruncate(handle, newSize)
```

## 读取、设置文件的信息
``` C
    yaffs_chmod(path, mode)
    yaffs_fchmod(handle, mode)
    yaffs_access(path,amode)
    yaffs_stat(path, struct yaffs_stat *buf)
    yaffs_fstat(handle, struct yaffs_stat *buf)
    yaffs_lstat(path, struct yaffs_stat *buf)
    yaffs_readlink(path, buf, bufsize)
```

## 改变目录结构和名称
``` C
    yaffs_open(path, flags, mode)
    yaffs_mkdir(path, mode)
    yaffs_symlink(targetPath, linkPath)
    yaffs_mknod(pathname, mode, dev)
    yaffs_link(existingPath, newath)
    yaffs_unlink(path)
    yaffs_rmdir(path)
    yaffs_rename(oldPath, newPath)
```

## 搜索目录
``` C
    yaffs_DIR *yaffs_opendir(const YCHAR *dirname) ;
    struct yaffs_dirent *yaffs_readdir(yaffs_DIR *dirp) ;直接目录
    void yaffs_rewinddir(yaffs_DIR *dirp) ;复位目录
    int yaffs_closedir(yaffs_DIR *dirp) ;
```

## 搜索目录（文件接口）
``` C
    int yaffs_open(const YCHAR *dirname, O_RDONLY, 0) ;
    struct yaffs_dirent *yaffs_readdir_fd(int fd) ;
    void yaffs_rewinddir_fd(int fd) ;
    int yaffs_close(int fd) ;
```

## 挂载控制
``` C
    yaffs_mount(path) ;
    yaffs_mount2(path, readOnly);
    yaffs_unmount(path); //如果有文件打开会失败
    yaffs_unmount2(path, int force);  //如果有文件打开也会强制取消挂载
    yaffs_remount(path, force, readOnly);
    yaffs_sync(path) ;
    yaffs_format(const YCHAR *path, //格式化
                    int unmount_flag,
                    int force_unmount_flag,
                    int remount_flag)
```

## 扩展属性（xattrt）
``` C
    /*
     * 设置、获得、删除、列出 xattr
     * flag:
     * XATTR_CREATE:创建，如果已经存在，返回报错
     * XATTR_REPLACE：替换已有的
     * 0：仅仅创建，如果存在就替换，如果空间不足报错
     */
    int yaffs_setxattr(const char *path, const char *name, const void *data, int size, int flags);
    int yaffs_lsetxattr(const char *path, const char *name, const void *data, int size, int flags);
    int yaffs_fsetxattr(int fd, const char *name, const void *data, int size, int flags);
    
    int yaffs_getxattr(const char *path, const char *name, void *data, int size);
    int yaffs_lgetxattr(const char *path, const char *name, void *data, int size);
    int yaffs_fgetxattr(int fd, const char *name, void *data, int size);

    int yaffs_removexattr(const char *path, const char *name);
    int yaffs_lremovexattr(const char *path, const char *name);
    int yaffs_fremovexattr(int fd, const char *name);

    int yaffs_listxattr(const char *path, char *list, int size);
    int yaffs_llistxattr(const char *path, char *list, int size);
    int yaffs_flistxattr(int fd, char *list, int size);
```

# 参考文献
>YaffsDirect.pdf官方说明文档
>官网:https://yaffs.net/
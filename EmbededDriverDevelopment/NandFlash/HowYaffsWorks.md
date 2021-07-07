[TOC]

---

# 什么是yaffs
yaffs是用于征地闪存的特性设计的用于NAND闪存的文件系统。Yaffs1是yaffs的初始版本。支持512字节的NAND设备。Yaffs2是Yaffs1的后续产品，在原来基础上扩展支持更大的设备。

---

# 对象 (Objects)
对象object是存储在文件系统的一切，包括了：
* 通常的数据文件
* 目录
* 硬链接
* 软链接
* 特殊对象 (管道，设备等等)

所有的对象有一个唯一的整数对象标识(obj_id)
在标准的POSIX中包括了 inodes 和 directory entries(dentries)。inode一般是常规文件，目录或者特殊文件。dentry提供了一种定位inodes的机制。在POSIX下，每个inode可能有0个，1个，或者多个dentries。一对一最好理解。允许多个dentries访问同一个inode，这是通过硬链接实现的。inode也会拥有0个dentries，当inode打开时取消链接，这是inode依然存在但是没有dentries。

---

# Yaffs1 存储文件
Yaffs1 拥有一个修改过的日志结构，Yaffs2 拥有真正的日志结构。真正的日志结构文件系统只能按照顺序写入。Yaffs中使用的删除标记，打破了这一规则。
文件系统不是在特定的位置写入数据，而是按照顺序以日志的形式写入，每个日志都占一个chunk大小，有两种类型的chunk：
* Data chunk: 保存通常的文件内容
* Object Header: 对象的描述符，包含详细信息，例如父目录的标识符，对象名称等等。
每一个chunk都有一个关联的tags，包含以下重要字段：
* Object Id(obj_id)：表示这个chunk属于哪一个对象
* Chunk Id(chunk_id)：表示该块在文件中的位置。如果是Object Header的 chunk_id == 0，文件的第一个数据chunk_id == 1，往后以此类推。
* Deletion Marker(is_deleted)：yaffs1独占。表示这一个chunk不再使用。
* Byte Count(n_bytes)：一个数据chunk中字节的数量
* Serial Number(serial_number)：yaffs1独占。用来区别chunk，当拥有相同的obj_id和chunk_id。
## 举个例子
1. 有一个每个block只有4个chunk的nand类型，创建一个文件
   Block|Chunk|obj_id|chunk_id|Deletion|Comment
   ----|----|----|----|----|----
   1|0|500|0|Live|Object Header
2. 往这个文件中写入一些数据
   Block|Chunk|obj_id|chunk_id|Deletion|Comment
   ----|----|----|----|----|----
   1|0|500|0|Live|Object Header
   1|1|500|1|Live|数据的第一个chunk
   1|2|500|2|Live|数据的第二个chunk
   1|3|500|3|Live|数据的第三个chunk
3. 接下来关闭这个文件，这时会为这个object创建一个新的header。这就意味着要删除之前的对象标题。
   Block|Chunk|obj_id|chunk_id|Deletion|Comment
   ----|----|----|----|----|----
   1|0|500|0|**Deleted**|旧的 Object Header(长度为0)
   1|1|500|1|Live|数据的第一个chunk
   1|2|500|2|Live|数据的第二个chunk
   1|3|500|3|Live|数据的第三个chunk
   2|0|500|0|Live|新的 Object Header(长度为n)
4. 接着打开这个文件进行读写，覆盖文件的第一个chunk的内容
   Block|Chunk|obj_id|chunk_id|Deletion|Comment
   ----|----|----|----|----|----
   1|0|500|0|Deleted|旧的 Object Header
   1|1|500|1|**Deleted**|旧的数据的第一个chunk(长度为0)
   1|2|500|2|Live|数据的第二个chunk
   1|3|500|3|Live|数据的第三个chunk
   2|0|500|0|**Deleted**|旧的 Object Header(长度为n)
   2|1|500|1|Live|新的第一个数据 chunk
   2|2|500|0|Live|有一个新的 Object Header(长度为n)
5. 这时通过 O_TRUNC 打开文件并关闭文件使文件大小为0，那么就会写入一个长度为0的新Object Header，并且删除之前的数据块。
    Block|Chunk|obj_id|chunk_id|Deletion|Comment
   ----|----|----|----|----|----
   1|0|500|0|Deleted|旧的 Object Header
   1|1|500|1|Deleted|旧的数据的第一个chunk(长度为0)
   1|2|500|2|**Deleted**|数据的第二个chunk
   1|3|500|3|**Deleted**|数据的第三个chunk
   2|0|500|0|Deleted|旧的 Object Header(长度为n)
   2|1|500|1|**Deleted**|新的第一个数据 chunk
   2|2|500|0|**Deleted**|有一个新的 Object Header(长度为n)
   2|3|500|0|Live|新的 Object Header(长度为0)
6. 当block1中的所有chunk都标记为删除的时候，我们可以擦除块1并且重新利用这一块空间。
   Block|Chunk|obj_id|chunk_id|Deletion|Comment
   ----|----|----|----|----|----
   1|0|500| | | 已删除
   1|1|500| | | 已删除
   1|2|500| | | 已删除
   1|3|500| | | 已删除
   2|0|500|0|Deleted|旧的 Object Header(长度为n)
   2|1|500|1|Deleted|旧的第一个数据 chunk
   2|2|500|0|Deleted|旧的 Object Header(长度为n)
   2|3|500|0|Live|新的 Object Header(长度为0)
7. 重命名该文件，会产生一个新的对象头
   Block|Chunk|obj_id|chunk_id|Deletion|Comment
   ----|----|----|----|----|----
   1|0|500| | | 已删除
   1|1|500| | | 已删除
   1|2|500| | | 已删除
   1|3|500| | | 已删除
   2|0|500|0|Deleted|旧的 Object Header(长度为n)
   2|1|500|1|Deleted|旧的第一个数据 chunk
   2|2|500|0|Deleted|旧的 Object Header(长度为n)
   2|3|500|0|**Deleted**|旧的 Object Header(长度为0)
   3|0|500|0|Live|包含新名称的Object Header
   此时的block也是可以擦除的状态了
   这些tags起到的作用：
   * obj_id:当前的chunk属于哪一个obj
   * chunk_id:这个chunk在文件中的哪一个位置
   * Deletion：哪一个chunk才是当前正在使用的

在闪存中没有文件分配表或者类似的结构，这样减少写入和擦除的次数，增加了稳定性。

---

# 垃圾收集
从block中去复制那些有用的chunk，这样原来的chunk就可以擦除，使得整个block可以被擦除重用，这一过程就是垃圾收集。

工作的流程如下：
1. 找到一个值得收集的block
2. 遍历这个block中的chunks。将正在使用的chunk创建一个新的副本，删除原来的。修改RAM的数据结构反映更改。

确定一个区块是否值得收集的启发式如下：
1. 如果有很多可以擦除的块，yaffs只会尝试对使用很少的block进行收集，这种称为被动垃圾收集。一次只收集几个chunk，将block的收集工作放到多个收集周期来完成，提高系统的响应能力
2. 如果被擦除的块很少，yaffs努力恢复更多的空间，称为激进垃圾收集。空间紧迫，在一个来收集周期中收集整个块。

---

# Yaffs1序列号(serial numbers)
Yaffs1 中的每一个chunk都会有一个2bit的序列号，当一个相同tags的chunk被拷贝或者替换时，这个序列号就会增加。为了提高系统的稳定性，在删除一个旧的chunk前，换先去创建新的chunk。如果系统出现故障，会出现两个相同的tags的chunk，就需要通过检查序列号来确定哪一个chunk是当前的。

---

# Yaffs2 NAND模型
Yaffs2是在原来功能上的扩展来实现新的目标包括：
1. 零复写。Yaffs1上需要复写备用区域中的一个bit位来标志删除标记。更现在的NAND对复写的容忍度更低，Yaffs2实现了零复写。
2. block内的顺序写入。现代的NAND更倾向顺序的写入。由于Yaffs2不需要使用删除标记，采用更加严格的顺序写入。
3. 新的闪存技术(MLC)只有在顺序写入才能工作。
   
Yaffs2不使用删除标记，通过以下机制确定那一块chunk是当前的：
* 序列号(seq_number)：随着block的分配，文件系统的seq_number随之增加，并给每一个chunk标记。组织成为时间顺序的log结构。
* 收缩头标记(shrink_header)：收缩头标记用于收缩文件数据大小是而写入的文件头。

注意：在Yaffs2仍然有chunk的删除。chunk会在RAM的数据结构中被标记用于垃圾回收等等，没有把标记写入flash

## 序列号(sequence number)
序列号可以让Yaffs确定事件的序列和回复文件的状态。通过向后扫描的时间顺序（扫描从最高的序列号到最低的序列号）。因此:
* 由于是向后扫描，首先遇到的与 obj_id:chunk_id 匹配的chunk是当前使用的，后续扫描到的都是过去的需要被删除。
* Object Header中的文件长度用于修建文件的大小。超出文件长度的块会被视为要删除。当前的和弃置的对象头都需要对大小进行精确地重建。

## 收缩头(shrink header)
收缩标头标记的目的是显示该对象标头指示文件的大小已缩小，并防止垃圾收集删除这些对象标头。

考虑下列操作:
``` C
#define MB (1024 * 1024)

h = open(“foo”,O_CREAT| O_RDWR, S_IREAD|S_IWRITE); /* create file */
write(h,data,5*MB); /* write 5 MB of data */
truncate(h,1*MB); /* truncate to 1MB */
lseek(h,2*MB, SEEK_SET); /* set the file access position to 2MB */
write(h,data,1*MB); /* write 1MB of data */
close(h);
```
该文件现在将是 3MB 的长度，但在 1MB 和 2MB 之间会有一个“漏洞”。根据 POSIX（以及帮助安全等），“漏洞”应该始终读回为零。
在Yaffs2中会通过以下的序列chunk来表示：
1. 创建时的Object Header(file length == 0)
2. 5MB 的数据块（0 到 5MB）
3. 收缩的 Object Header(file length == 1)
4. 1MB 的数据块（2MB 到 3MB）
5. 关闭时的 Object Header(file length == 3)
   
当前的数据chunk有：
* 步骤2中创建的 1MB 数据
* 步骤4中创建的 1MB 数据
* 步骤5中的Object Header

Yaffs2需要记住其中发生的截断，否则就会忘记文件中间有一个洞。Yaffs2会创建一个shrink header标记洞的开始，用一个常规Object Header表示洞的结束。

---

# 坏块处理
任何没有有效坏块处理策略的闪存文件系统都不适合 NAND 闪存。Yaffs1 使用 Smart-Media 风格的坏块标记。这将检查备用区域的第六个字节（字节 5）。在一个好的块中，这应该是 0xFF。工厂标记的坏块应该是 0x00。如果 Yaffs1 确定一个块坏了，它会用 0x59 ('Y') 标记为坏。使用独特的标记可以将 Yaffs 标记的坏块与工厂标记的坏块区分开来。Yaffs2 模式旨在支持更广泛的设备和备用区域布局。因此，Yaffs2 不会决定标记哪些字节，而是调用驱动程序函数来确定块是坏的，还是将其标记为坏的。如果读取或写入操作失败或检测到三个 ECC 错误，Yaffs 将标记一个块为坏块。标记为坏的块将不再使用，从而提高文件系统的健壮性。NAND 闪存单元有时会受到 NAND 活动和电荷损失的干扰（损坏）。这些错误由纠错码 (ECC) 纠正，纠错码可以在硬件、软件驱动程序或 Yaffs 本身中实现。同样，任何缺乏有效 ECC 处理的闪存文件系统都不适合 NAND 闪存。Yaffs1 模式既可以使用内置的 ECC，也可以使用驱动程序或硬件提供的 ECC。由于 Yaffs2 模式是为更广泛的设备设计的，它内部不提供 ECC，而是要求驱动程序提供 ECC。Yaffs 提供的 ECC 代码 (yaffs_ecc.c) 是我们所知的智能媒体兼容 ECC 算法的最快 C 代码实现。此代码将纠正 256 字节数据块内的任何单个位错误，并检测每个 256 字节数据块中的 2 个错误。这足以在大多数 SLC 型 NAND 闪存上提供非常高的可靠性的纠错。

---

# RAM结构
RAM的结构定义在 yaffs_guts.h 中，它们的主要用途是：
* 设备/分区：这被命名为yaffs_dev 。这是保存与 Yaffs“分区”或“挂载点”相关的信息所必需的。使用这些而不是全局变量，允许 Yaffs 同时支持多个分区和不同类型的分区。事实上，这里提到的几乎其他数据结构都是该结构的一部分或通过该结构访问。
* NAND Block信息：它被命名为yaffs_block_info并保存 NAND 块的当前状态。每个 yaffs_dev 都有一个这些数组。
* NAND Chunk信息：附加到 yaffs_dev 的位域，用于保存系统中每个块的当前使用状态。分区中的每个块有一位。
* Object：它被命名为yaffs_obj并保存对象的状态。文件系统中的每个对象都有一个，其中一个对象是常规文件、目录、硬链接、符号链接或特殊链接之一。yaffs_Objects 有不同的变体来反映这些不同对象类型所需的不同数据。每个对象都由 obj_id 唯一标识。
* 文件结构：对于每个文件对象，Yaffs 持有一个树，它提供了在文件中查找数据块的机制。树由称为yaffs_tnode（树节点）的节点组成。
* 目录结构：目录结构允许按名称查找对象。目录结构是从以目录为根的双向链表构建的，并将目录中的同级对象绑定在一起。
* 对象编号哈希表：对象编号哈希表提供了一种从对象的 obj_id 中查找对象的机制。obj_id 被散列以选择一个散列桶。每个哈希桶都有一个属于这个哈希桶的对象的双向链接列表，以及桶中对象的计数。
* 缓存：Yaffs 提供读/写缓存，可显着提高短操作的性能。缓存的大小可以在运行时设置。

## Yaffs 对象(struck yaffs_obj)
每一个文件系统中的对象都有一个 yaffs_obj 结构表示。yaffs_obj主要功能存储大部分对象元数据和特定类型的信息。元数据包括：
* obj_id：表示对象的编号
* parent：指向父目录的指针，并非所有对象都有父对象。未链接和已删除文件的根目录和特殊目录没有父目录，因此为 NULL。
* short_name：如果名称足够短以适合固定大小的小数组（默认为 16 个字符），则将其存储在此处，否则每次请求时都必须从闪存中获取。
* type：对象的类型。
* 权限(Permission)、时间(Time)、所有权(ownership)和其他属性

根据不同的对象类型，yaffs_obj还会存储：
* 数据文件：tnode树，文件范围
* 目录：目录链
* 硬链接：等效对象
* 软链接：字符串

## 通过 obj_id 查找
这个机制的目的是通过 obj_id 快速访问对应的设备。每一个 yaffs_dev 都使用一个哈希表，哈希表有256个桶，每一个桶包含一个双向链表存放 yaffs_objs。yaffs的哈希函数只是屏蔽了obj_ids最低有效位。yaffs分配使得每个桶中的对象的数量维持在相当低的水平。

## 目录结构
目录结构的目的是通过名称快速访问对象。这是执行文件操作（例如打开、重命名等）所必需的。Yaffs的目录结构是由YAFFS_OBJECT_TYPE_DIRECTORY 类型的对象树组成。这些对象有一个双向连接的子节点。yaffs_obj也有一个双向链表节点，称为兄弟节点，将目录中的兄弟节点链接在一起。每个yaffs分区还会有一些假目录，这些目录不存在NAND中，但会在挂载的时候创建：
* lost+found：此目录用作存储任何无法放置在正常目录树中的丢失文件部分的地方。
* Unlinked and deleted：将对象放置在这些目录中会使它们处于未链接或已删除的状态。

名称解析通过两种机制加速：
* short_name 直接存储在 yaffs_obj 中，因此不必从内存中加载
* 每一个对象都有一个**名称总和**，通过一个简单的名称散列加快了匹配过程

![](images/2021-07-07-17-03-17.png)
上图包含了以下的目录树
/|根目录
----|----
/a|目录包含 c,d,e
/b|空目录
/lost+found|空目录
/a/c|目录
/a/d|数据文件
/a/e|数据文件
如果yaffs分区损害，在扫描器件创建对象时，找不到对象头(包含对象名称)。则该对象没有名称，yaffs会为这个对象创建一个假名，以便放在目录结构中。

## 硬链接
在 POSIX 文件系统中，Object（Linux 术语中的 inode，BSD 中的 vnode）和Object Names（Linux 术语中的 dentry）是独立的，因为一个对象可以有零个、一个或多个名称。
具有零链接的文件是由以下结构引起的：
``` C
h = open(“foo”,O_CREAT | O_RDWR, S_IREAD | S_IWRITE); /* create file “foo” */
unlink(“foo”); /* unlink it to removedirectory entry */
/* we still have a handle to the file so can still access it via the handle */
read(h,buffer,100);
write(h,buffer,100);
close(h); /* as soon as the handle is closed the file is deleted */
```
Yaffs不单独存储 object 和 name，这样会导致额外的读写。Yaffs为大多数典型情况，一个name对应一个Object做了优化。通过“作弊”来实现其他的功能：
* 

---

# 参考文献
> https://yaffs.net/documents/how-yaffs-works
> 官方文档	HowYaffsWorks.pdf
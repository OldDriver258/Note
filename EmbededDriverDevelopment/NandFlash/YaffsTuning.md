# Yaffs 的直接配置
## 初始化参数 Parameters
每一个 yaffs_dev 结构都会包含一个 yaffs设备的参数结构体，包含以下词条：
名称|设备可选名称
----|----
totalBytesPerChunk|flash一个页中的字节数量。如果未设置inbandTags，整个区域用来存储数据，备用区域存储标签；如果设置了整个区域用来存放标签，剩余的区域用来存放数据。
spareBytesPerChunk|备用区域的可用字节数，这一区域用于存储ECC，坏块标记。
startBlock|分区第一个使用的块号，0是第一个块。
endBlock|分区使用的最后一个块
|
nReservedBlocks|保留的好块的数量，可以用于垃圾收集等等。至少需要2个，5个是最典型的数字。
inbandTags|Yaffs2独占。定义是否标签被保存在数据区域的标志Flag。如果使用inbandTags，需要同时开启nShortOpCaches。
useNANDECC|Yaffs1独占。是否使用nand自带的ECC。设置为0表示启用yaffs的ECC。
noTagsECC|Yaffs2独占。标签是否包含ECC数据。
isYaffs2|是否使用Yaffs2机制
|
nShortOpCaches|短操作缓冲的页的数量。设置为0表示不使用Cache。典型值为10到20。
emptyLostAndFound|是否在挂载的时候删除所有lost+found中的文件。
skipCheckpointRead|Yaffs2独占。是否跳过读取挂载时的检查点。跳过意味着强制重新扫描。
skipCheckpointWrite|Yaffs2独占。在同步和取消挂载的时候是否写入检查点。
refreshPeriod|Yaffs2独占。多久进行一次块的刷新。值小于10表示关闭块刷新。典型值为1000
|
initialiseNAND|初始化驱动的函数的指针
deinitialiseNAND|取消初始化驱动的函数的指针
eraseBlockInNAND|擦除flash的块的函数的指针
|
writeChunkToNAND|Yaffs1独占。写一个chunk的函数的指针
readChunkFromNAND|Yaffs1独占。读一个chunk的函数的指针
|
writeChunkWithTagsToNAND|Yaffs2独占。写一个chunk和tags的函数的指针
readChunkWithTagsFromNAND|Yaffs2独占。读一个chunk和tags的函数的指针
markNANDBlockBad|Yaffs2独占。标记坏块的函数
queryNANDBlock|Yaffs2独占。询问块的状态函数
|
gcControl|返回垃圾收集控制标志位的回调函数，可选
removeObjectCallback|一个object移除是会调用的回调函数
markSuperBlockDirty|第一次修改一个干净的文件系统调用的函数
|
disableSoftDelete|Yaffs1独占，调试用。禁用软删除。
|
useHeaderFileSize|Debug only.
disableLazyLoading|Debug only.
wideTnodeDisabled|Debug only.
|
deferDirectoryUpdate|延迟目录更新

## 编译设置

* CONFIG_YAFFS_ALWAYS_CHECK_CHUNK_ERASED
  通常yaffs只在写每个block的第一个chunk的时候才会检查擦除。这个表标志可以强制检查所有的chunk
* CONFIG_YAFFS_CASE_INSENSITIVE
  有些操作系统大小写不敏感，需要设置这个标志位来让yaffs也大小不敏感
* CONFIG_YAFFS_DIRECT
  使用Yaffs_Direct
* CONFIG_YAFFS_DISABLE_LAZY_LOAD
  延迟加载用于在访问object的时候，延迟加载object的细节。从而加速挂载的时间。这个标志位可以关闭延迟加载。
* CONFIG_YAFFS_ECC_WRONG_ORDER
  这个标志位用于在旧的Linux MTD中有ECC字节顺序的bug的时候。
* CONFIG_YAFFS_NO_YAFFS1
  只编译yaffs2的相关操作。目前不支持
* CONFIG_YAFFS_SHORT_NAMES_IN_RAM
  yaffs将object的短名字存在RAM里。这样更快，但是会占用更多的内存。
* CONFIG_YAFFS_UNICODE
  配置yaffs使用 Unicode文件名
* CONFIG_YAFFS_USE_OWN_SORT
  默认个情况下，在挂载扫描时，yaffs使用qsort来块排序，这种排序很快。通过这个标志位强制让yaffs使用内部的排序。Debug only.
* CONFIG_YAFFS_UTIL
  编译yaffs为实用程序
* CONFIG_YAFFS_WINCE
  编译为 WinCE
* CONFIG_YAFFS_YAFFS2
  当使用Yaffs2模式时配置。目前应当始终配置。
* CONFIG_YAFFSFS_PROVIDE_VALUES
  编译 Yaffs Direct 是配置。

# 参考文献
>https://yaffs.net/documents/yaffs-tuning
>官方文档	YaffsTuning.pdf
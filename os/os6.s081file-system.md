# os|6.s081|file-system

使用一定的数据结构，来索引磁盘上特定位置所保存的数据

![](./imgs/fs-organization.png)

## 持久化

- 文件系统为用户的文件提供层级的路径名，便于组织和管理
- 借助文件名，用户和进程方便使用和在程序中操作文件
- 断电保存数据，重启后仍能恢复使用

## 实现

- 使用字符串表示文件具体的路径名，打开文件后，fs会为write函数保存当前写fd的offset
- 使用`link`可以给同一个文件指定不同的名字，fs需要保存这种映射关系
- fd是直接与文件（内部的对象）相对应，而传入字符串只是为了找到该文件，与该字符串（文件名）没有关系

### inode

一个文件在fs中的存在形式，类似于一个`class/struct`，互相之间通过fs内部的一个编号区分；同时，不同的路径名表示同一个inode，通过link count计数

- 文件——包含多个data block，所以inode也算一种metadata

- 64bytes

  ~~~c
  struct inode {
    uint dev;           // Device number
    uint inum;          // Inode number
    int ref;            // Reference count
    struct sleeplock lock; // protects everything below here
    int valid;          // inode has been read from disk?
  
    short type;         // file or directory?
    short major;
    short minor;
    short nlink;//name counter
    uint size;//sizeof data in the file
    uint addrs[NDIRECT+1];//0-11 point to the first 12 blocks of data;block[12]->a block(256 numbers which represent the indexes of blocks storing the data of file in 
  };
  ~~~

- 最多共使用12+256个blocks存data（如果想拓展，则可以使用更加复杂的索引结构）
- 对于一个目录的保存，使用多个directory entries，而其中每一个entry表示其中的某一个文件，其格式为`inode(文件或当前目录)+name of file`

### fd

...

## 存储设备

IO速度量级：SSD:0.1-1ms;HDD:10ms

- sector（磁盘本身，硬件角度）：disk driver 读写最小单元-512bytes
- block（fs角度，软件）：多个sectors

磁盘驱动可以看作是一种有统一标准的软件，用于抽象底层的物理存储设备，从而提供给上层统一的api，忽略不同disk的物理特性

文件系统：在磁盘的特定位置保留meta data，重启后可以重新构建fs（软件断电后状态消失）

- block0-》boot system
- block1：meta data
- blockxxx：logging
- blockxxx：all inodes
- bitmap block：后续的某个data block是否空闲
- data block

## xv6

~~~shell
nmeta 46 (boot, super, log blocks 30 inode blocks 13, bitmap blocks 1) blocks 954 total 1000
~~~

- 创建文件：写一个新的inode，并更改对应路径下所有上一级目录的inode和data block
- 写入文件：扫描bitmap找可以用于写入数据的data block并改变标记；写入对应data block，并更新元数据

### 具体实现-创建inode

- 首先解析路径名的最后一个文件，如果该文件不存在，则调用`ialloc`
- 在alloc函数中，遍历所有inode，若为空闲，则使用该inode
- 检查inode状态时，需要从block中读取相关的信息；这时需要进入buffer cache中找block的缓存，且每一个进程都有可能使用该缓存，所以要在找到后refcnt++，所以使用锁保护bcache；之后对于具体的block，也需要使用sleep lock保护
- 当refcnt为0时，该block才会被请出buffer cache

#### sleep lock（期待disk）

- block cache作为访问或修改block的中间介质，可能需要磁盘操作 which need很长时间，且spinlock（期待memory）关闭了中断，即使disk准备好后也无法获知该信息，所以即使获取了锁也可能会导致在一个cpu上空转
- sleep lock作为一个中间的替代品，获取spinlock之后，表明当前sleep lock有可能被获取，若无法获取，则通过sleep出让cpu，也不需要等待disk；否则通过`lk->locked=1`表明获取了该锁，之后可以释放spinlock，直接操作disk
- Todo:操作disk和trap的关系？
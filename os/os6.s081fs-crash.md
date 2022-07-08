# os|6.s081|fs-crash

## 问题定义

- 两个文件有同一个data block
- 两个文件占据同一个inode

断电导致重启后block不一致的问题

类似于运行中的中断

## example



## logging

- 文件系统的数据结构处于一种无法被fs处理的状态
- 数据的不一致

> 使用虚拟机跑logecfatcache，跑多了之后，重启奔溃
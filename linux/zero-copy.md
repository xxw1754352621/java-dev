# Linux零拷贝分为：直接io、mmap()、sendfile()、splice()

传统IO：

- 问题：传统IO（比如java中的FileInputStream等类的操作），在读，写文件的时候，数据经过【用户空间】进程空间 和【内核空间】 页缓存的多次拷贝，给cpu和内存带来巨大的开销

- 优点：内核空间和用户空间隔离，安全运行；降低IO次数，减低磁盘的压力

  

# 零拷贝：mmap()方法

- 代替read方法
  - 磁盘->文件缓冲区->用户空间
- mmap
  - 磁盘->用户空间

- 数据通过DMA拷贝到系统内核的缓存区，这个缓存区就是进程和操作系统内核共享的缓存区。所以，内核和进程不需要进行任何的数据拷贝操作

# 零拷贝：sendfile()方法

- 代替传统的read+write+buffer的socket通信方式
- sendfile
  磁盘->文件缓冲区->socket缓冲区->协议引擎

# [kafka层面的使用](https://manbuyun.github.io/2017/01/13/%E4%B8%BA%E4%BB%80%E4%B9%88Kafka%E9%82%A3%E4%B9%88%E5%BF%AB/)
# 生产者：mmap()

  - [MappedByteBuffer](https://www.cnblogs.com/huxi2b/p/6637425.html)类操作
      - mmap的内存文件映射在full gc时才会进行释放
  - 实现内存文件映射，操作共享内存，后期定时刷新到磁盘
  - MappedByteBuffer对文件进行读写操作。其中，利用了NIO中的FileChannel模型将磁盘上的物理文件直接映射到**用户态的内存**地址中（这种Mmap的方式减少了传统IO将磁盘文件数据在操作系统内核地址空间的缓冲区和用户应用程序地址空间的缓冲区之间来回进行拷贝的性能开销），将对文件的操作转化为直接对内存地址进行操作，从而极大地提高了文件的读写效率，后期定时刷新到磁盘

#   消费者：sendfile()

- Kafka把所有的消息都存放在一个个文件中，当消费者需要数据的时候Kafka直接把“文件”发送给消费者

- Kafka是用mmap作为文件读写方式的，它就是一个文件句柄，所以直接把它传给sendfile；偏移也好解决，用户会自己保持这个offset，每次请求都会发送这个offset

# 总结

Kafka速度的秘诀在于，它把所有的消息都变成一个的文件。通过mmap提高I/O速度，写入数据的时候它是末尾添加所以速度最优；读取数据的时候配合sendfile直接暴力输出
视频: https://www.bilibili.com/video/BV1fE411V7Ho?from=search&seid=16368021598862205361
博客: https://blog.csdn.net/prestigeding/article/details/78888290
书籍: rocketmq技术内幕



1消息发送与启动
定时获取以及主动拉取nameserver topic的queue信息信息 【根据配置不存在则创建】


2commitlog
shareLog 一个broker一个topic只有一个逻辑化的commitlog[物理上被切割成很多小的]
commitlog对应MapedQueue.java 每一个小的文件块对应MappedFile

消息可以控制可读可写权限
场景，比如顺序消息的扩容和缩容，要控制可写

commitlog是顺序写
consume queue是消费队列[存储元信息]
maxoffset在消息顺序写入commitlog后增加

2 kafka在单broker的时候tps降低

2 commitlog是顺序写随机读随机读是否影响性能







TiDB存储引擎












2 映射文件的获取创建以及消息的写入
commitlog
映射文件
fileFromOffset
起始偏移量 其实就是commitlog文件名的尾缀、

AllocateMappedFileService








3 同步刷盘与异步刷盘
        
       直接到页缓存或者堆外内存
       byteBuffer.put(this.msgStoreItemMemory.array(), 0, msgLen);
       堆外到os页缓存
       protected void commit0(final int commitLeastPages) {
            int writePos = this.wrotePosition.get();
            int lastCommittedPosition = this.committedPosition.get();
            if (writePos - this.committedPosition.get() > 0) {
                try {
                    ByteBuffer byteBuffer = writeBuffer.slice();
                    byteBuffer.position(lastCommittedPosition);
                    byteBuffer.limit(writePos);
                    this.fileChannel.position(lastCommittedPosition);
                    // 写到OSPAGECache
                    this.fileChannel.write(byteBuffer);// 堆外缓存
                    this.committedPosition.set(writePos);
                } catch (Throwable e) {
                    log.error("Error occurred when commit data to FileChannel.", e);
                }
            }
        }
        刷盘方式
          this.fileChannel.force(false);
          this.mappedByteBuffer.force();




一段时间或者磁盘空间不足 删除commitlog





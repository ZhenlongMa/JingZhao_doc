# 总体设计

京兆网卡的整体架构如下图所示：

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

其中QueueSubsystem负责管理所有类型的队列资源，包括SQ、RQ、CQ、EQ；ResMgtSubsystem负责管理所有类型的通信状态资源，包括队列上下文、MPT、MTT；RDMACore是RDMA逻辑的核心部分，解析WQE、进行虚实地址转换，并将数据负载发送给传输子系统TransportSubsystem。

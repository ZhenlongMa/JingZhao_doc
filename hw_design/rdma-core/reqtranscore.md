# ReqTransCore

ReqTransCore负责处理本地通信操作的发起部分。该模块由四个流水级组成。第一个流水级负责接收subWQE，并发起上下文读请求；第二个流水级接收到上下文读响应后发起Memory Region元数据的读请求；第三个流水级收到Memory Region元数据响应后发起数据gather的读请求；第四个流水级收集所有待发送数据，发送至transport引擎。

Thread 1较为简单，此处省略介绍。

Thread 2的状态机跳转如图所示：

<figure><img src="../../.gitbook/assets/thread2_sm.png" alt=""><figcaption><p>Thread 2状态机</p></figcaption></figure>

当Thread 2接收到上下文响应的时候，先判断当前待执行的sub-WQE是否为RDMA Read操作或者inline传输模式，如果是这两种情况则在本次处理过程中不需要读取数据MR，状态机跳转至BYPASS状态，否则跳转至FETCH\_DATA\_MR状态。在FETCH_DATA_MR状态中，视是否需要当前发起完成信号和完成事件跳往相应的状态。

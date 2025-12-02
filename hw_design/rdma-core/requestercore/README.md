# RequesterCore

## 模块功能

RequesterCore是RDMA协议引擎的核心组件，负责处理请求发送和响应接收的完整流程。该模块实现了两个关键数据路径：TX\_REQ（请求发送）路径处理WQE并生成网络数据包；RX\_RESP（响应接收）路径处理网络响应包并完成数据传输。模块采用模块化设计，集成了三个关键子模块协同工作，确保高效可靠的RDMA操作。

* ReqTransCore子模块：
  * 解析来自WQEParser的WQE元数据
  * 通过OoOStation获取QP上下文和内存区域信息
  * 管理DMA读操作(gather)获取WQE数据
  * 生成网络数据包并插入PacketBuffer
  * 触发CQ/EQ更新机制
  * 将WQE信息写入WQEBuffer供响应处理
  * 负责TX\_REQ路径处理
* RespRecvCore子模块：
  * 解析来自PacketDeparser的网络响应包
  * 通过OoOStation获取QP上下文和内存区域信息
  * 管理DMA写操作(scatter)将数据写回主机内存
  * 从WQEBuffer读取对应WQE完成处理
  * 释放接收缓冲区资源
  * 触发CQ/EQ更新机制
  * 负责RX\_RESP路径处理
* WQEBuffer (DynamicMultiQueue)子模块：
  * 为256个QP提供独立WQE存储队列
  * 每个队列支持64个WQE槽位
  * 槽位宽度为256位，适配RC类型WQE
  * 支持多队列并发入队(ReqTransCore)和出队(RespRecvCore)
  * 通过队列索引和槽位号精确管理WQE生命周期
  * WQE动态存储管理

## 模块架构

<figure><img src="../../../.gitbook/assets/RequesterCore.png" alt=""><figcaption></figcaption></figure>

## 模块接口

<table><thead><tr><th width="247">信号名</th><th width="95">输入/输出</th><th width="87">位宽</th><th width="151">对接模块</th><th width="247">中文说明</th></tr></thead><tbody><tr><td>clk</td><td>input</td><td>1</td><td>全局时钟</td><td>时钟信号</td></tr><tr><td>rst</td><td>input</td><td>1</td><td>全局复位</td><td>复位信号</td></tr><tr><td>TX_REQ_sub_wqe_valid</td><td>input</td><td>1</td><td>WQEParser</td><td>TX_REQ WQE有效信号</td></tr><tr><td>TX_REQ_sub_wqe_meta</td><td>input</td><td>576</td><td>WQEParser</td><td>TX_REQ WQE元数据</td></tr><tr><td>TX_REQ_sub_wqe_ready</td><td>output</td><td>1</td><td>WQEParser</td><td>TX_REQ WQE就绪反压</td></tr><tr><td>TX_REQ_fetch_cxt_ingress_valid</td><td>output</td><td>1</td><td>OoOStation(CxtMgt)</td><td>TX_REQ上下文请求有效</td></tr><tr><td>TX_REQ_fetch_cxt_ingress_head</td><td>output</td><td>160</td><td>OoOStation(CxtMgt)</td><td>TX_REQ上下文请求头</td></tr><tr><td>TX_REQ_fetch_cxt_ingress_data</td><td>output</td><td>576</td><td>OoOStation(CxtMgt)</td><td>TX_REQ上下文请求数据</td></tr><tr><td>TX_REQ_fetch_cxt_ingress_start</td><td>output</td><td>1</td><td>OoOStation(CxtMgt)</td><td>TX_REQ上下文请求开始标志</td></tr><tr><td>TX_REQ_fetch_cxt_ingress_last</td><td>output</td><td>1</td><td>OoOStation(CxtMgt)</td><td>TX_REQ上下文请求结束标志</td></tr><tr><td>TX_REQ_fetch_cxt_ingress_ready</td><td>input</td><td>1</td><td>OoOStation(CxtMgt)</td><td>TX_REQ上下文请求就绪反压</td></tr><tr><td>TX_REQ_fetch_cxt_egress_valid</td><td>input</td><td>1</td><td>OoOStation(CxtMgt)</td><td>TX_REQ上下文响应有效</td></tr><tr><td>TX_REQ_fetch_cxt_egress_head</td><td>input</td><td>160</td><td>OoOStation(CxtMgt)</td><td>TX_REQ上下文响应头</td></tr><tr><td>TX_REQ_fetch_cxt_egress_data</td><td>input</td><td>640</td><td>OoOStation(CxtMgt)</td><td>TX_REQ上下文响应数据</td></tr><tr><td>TX_REQ_fetch_cxt_egress_start</td><td>input</td><td>1</td><td>OoOStation(CxtMgt)</td><td>TX_REQ上下文响应开始标志</td></tr><tr><td>TX_REQ_fetch_cxt_egress_last</td><td>input</td><td>1</td><td>OoOStation(CxtMgt)</td><td>TX_REQ上下文响应结束标志</td></tr><tr><td>TX_REQ_fetch_cxt_egress_ready</td><td>output</td><td>1</td><td>OoOStation(CxtMgt)</td><td>TX_REQ上下文响应就绪反压</td></tr><tr><td>TX_REQ_fetch_mr_ingress_valid</td><td>output</td><td>1</td><td>OoOStation(MRMgt)</td><td>TX_REQ内存区域请求有效</td></tr><tr><td>TX_REQ_fetch_mr_ingress_head</td><td>output</td><td>160</td><td>OoOStation(MRMgt)</td><td>TX_REQ内存区域请求头</td></tr><tr><td>TX_REQ_fetch_mr_ingress_data</td><td>output</td><td>576</td><td>OoOStation(MRMgt)</td><td>TX_REQ内存区域请求数据</td></tr><tr><td>TX_REQ_fetch_mr_ingress_start</td><td>output</td><td>1</td><td>OoOStation(MRMgt)</td><td>TX_REQ内存区域请求开始标志</td></tr><tr><td>TX_REQ_fetch_mr_ingress_last</td><td>output</td><td>1</td><td>OoOStation(MRMgt)</td><td>TX_REQ内存区域请求结束标志</td></tr><tr><td>TX_REQ_fetch_mr_ingress_ready</td><td>input</td><td>1</td><td>OoOStation(MRMgt)</td><td>TX_REQ内存区域请求就绪反压</td></tr><tr><td>TX_REQ_fetch_mr_egress_valid</td><td>input</td><td>1</td><td>OoOStation(MRMgt)</td><td>TX_REQ内存区域响应有效</td></tr><tr><td>TX_REQ_fetch_mr_egress_head</td><td>input</td><td>160</td><td>OoOStation(MRMgt)</td><td>TX_REQ内存区域响应头</td></tr><tr><td>TX_REQ_fetch_mr_egress_data</td><td>input</td><td>576</td><td>OoOStation(MRMgt)</td><td>TX_REQ内存区域响应数据</td></tr><tr><td>TX_REQ_fetch_mr_egress_start</td><td>input</td><td>1</td><td>OoOStation(MRMgt)</td><td>TX_REQ内存区域响应开始标志</td></tr><tr><td>TX_REQ_fetch_mr_egress_last</td><td>input</td><td>1</td><td>OoOStation(MRMgt)</td><td>TX_REQ内存区域响应结束标志</td></tr><tr><td>TX_REQ_fetch_mr_egress_ready</td><td>output</td><td>1</td><td>OoOStation(MRMgt)</td><td>TX_REQ内存区域响应就绪反压</td></tr><tr><td>TX_REQ_cq_req_valid</td><td>output</td><td>1</td><td>CompletionQueueMgt</td><td>TX_REQ CQ请求有效</td></tr><tr><td>TX_REQ_cq_req_head</td><td>output</td><td>64</td><td>CompletionQueueMgt</td><td>TX_REQ CQ请求头</td></tr><tr><td>TX_REQ_cq_req_ready</td><td>input</td><td>1</td><td>CompletionQueueMgt</td><td>TX_REQ CQ请求就绪反压</td></tr><tr><td>TX_REQ_cq_resp_valid</td><td>input</td><td>1</td><td>CompletionQueueMgt</td><td>TX_REQ CQ响应有效</td></tr><tr><td>TX_REQ_cq_resp_head</td><td>input</td><td>96</td><td>CompletionQueueMgt</td><td>TX_REQ CQ响应头</td></tr><tr><td>TX_REQ_cq_resp_ready</td><td>output</td><td>1</td><td>CompletionQueueMgt</td><td>TX_REQ CQ响应就绪反压</td></tr><tr><td>TX_REQ_eq_req_valid</td><td>output</td><td>1</td><td>EventQueueMgt</td><td>TX_REQ EQ请求有效</td></tr><tr><td>TX_REQ_eq_req_head</td><td>output</td><td>64</td><td>EventQueueMgt</td><td>TX_REQ EQ请求头</td></tr><tr><td>TX_REQ_eq_req_ready</td><td>input</td><td>1</td><td>EventQueueMgt</td><td>TX_REQ EQ请求就绪反压</td></tr><tr><td>TX_REQ_eq_resp_valid</td><td>input</td><td>1</td><td>EventQueueMgt</td><td>TX_REQ EQ响应有效</td></tr><tr><td>TX_REQ_eq_resp_head</td><td>input</td><td>96</td><td>EventQueueMgt</td><td>TX_REQ EQ响应头</td></tr><tr><td>TX_REQ_eq_resp_ready</td><td>output</td><td>1</td><td>EventQueueMgt</td><td>TX_REQ EQ响应就绪反压</td></tr><tr><td>TX_REQ_gather_req_wr_en</td><td>output</td><td>1</td><td>DMA引擎</td><td>TX_REQ DMA读请求使能</td></tr><tr><td>TX_REQ_gather_req_din</td><td>output</td><td>160</td><td>DMA引擎</td><td>TX_REQ DMA读请求数据(地址+长度)</td></tr><tr><td>TX_REQ_gather_req_prog_full</td><td>input</td><td>1</td><td>DMA引擎</td><td>TX_REQ DMA读请求FIFO满</td></tr><tr><td>TX_REQ_net_data_rd_en</td><td>output</td><td>1</td><td>GatherData</td><td>TX_REQ网络数据读使能</td></tr><tr><td>TX_REQ_net_data_dout</td><td>input</td><td>512</td><td>GatherData</td><td>TX_REQ网络数据输出</td></tr><tr><td>TX_REQ_net_data_empty</td><td>input</td><td>1</td><td>GatherData</td><td>TX_REQ网络数据空标志</td></tr><tr><td>TX_REQ_scatter_req_wen</td><td>output</td><td>1</td><td>DMA引擎</td><td>TX_REQ DMA写请求使能</td></tr><tr><td>TX_REQ_scatter_req_din</td><td>output</td><td>160</td><td>DMA引擎</td><td>TX_REQ DMA写请求数据(地址+长度)</td></tr><tr><td>TX_REQ_scatter_req_prog_full</td><td>input</td><td>1</td><td>DMA引擎</td><td>TX_REQ DMA写请求FIFO满</td></tr><tr><td>TX_REQ_scatter_data_wen</td><td>output</td><td>1</td><td>DMA引擎</td><td>TX_REQ DMA写数据使能</td></tr><tr><td>TX_REQ_scatter_data_din</td><td>output</td><td>512</td><td>DMA引擎</td><td>TX_REQ DMA写数据</td></tr><tr><td>TX_REQ_scatter_data_prog_full</td><td>input</td><td>1</td><td>DMA引擎</td><td>TX_REQ DMA写数据FIFO满</td></tr><tr><td>TX_REQ_insert_req_valid</td><td>output</td><td>1</td><td>PacketBuffer</td><td>TX_REQ插入包有效</td></tr><tr><td>TX_REQ_insert_req_start</td><td>output</td><td>1</td><td>PacketBuffer</td><td>TX_REQ插入包开始标志</td></tr><tr><td>TX_REQ_insert_req_last</td><td>output</td><td>1</td><td>PacketBuffer</td><td>TX_REQ插入包结束标志</td></tr><tr><td>TX_REQ_insert_req_head</td><td>output</td><td>14</td><td>PacketBuffer</td><td>TX_REQ插入包头(槽位索引)</td></tr><tr><td>TX_REQ_insert_req_data</td><td>output</td><td>512</td><td>PacketBuffer</td><td>TX_REQ插入包数据</td></tr><tr><td>TX_REQ_insert_req_ready</td><td>input</td><td>1</td><td>PacketBuffer</td><td>TX_REQ插入包就绪反压</td></tr><tr><td>TX_REQ_insert_resp_valid</td><td>input</td><td>1</td><td>PacketBuffer</td><td>TX_REQ插入包响应有效</td></tr><tr><td>TX_REQ_insert_resp_data</td><td>input</td><td>14</td><td>PacketBuffer</td><td>TX_REQ插入包响应数据(槽位索引)</td></tr><tr><td>TX_REQ_egress_pkt_valid</td><td>output</td><td>1</td><td>TransportSubsystem</td><td>TX_REQ出口包有效</td></tr><tr><td>TX_REQ_egress_pkt_head</td><td>output</td><td>488</td><td>TransportSubsystem</td><td>TX_REQ出口包头(元数据)</td></tr><tr><td>TX_REQ_egress_pkt_ready</td><td>input</td><td>1</td><td>TransportSubsystem</td><td>TX_REQ出口包就绪反压</td></tr><tr><td>RX_RESP_ingress_pkt_valid</td><td>input</td><td>1</td><td>PacketDeparser</td><td>RX_RESP入口包有效</td></tr><tr><td>RX_RESP_ingress_pkt_head</td><td>input</td><td>488</td><td>PacketDeparser</td><td>RX_RESP入口包头(元数据)</td></tr><tr><td>RX_RESP_ingress_pkt_ready</td><td>output</td><td>1</td><td>PacketDeparser</td><td>RX_RESP入口包就绪反压</td></tr><tr><td>RX_RESP_fetch_cxt_ingress_valid</td><td>output</td><td>1</td><td>OoOStation(CxtMgt)</td><td>RX_RESP上下文请求有效</td></tr><tr><td>RX_RESP_fetch_cxt_ingress_head</td><td>output</td><td>160</td><td>OoOStation(CxtMgt)</td><td>RX_RESP上下文请求头</td></tr><tr><td>RX_RESP_fetch_cxt_ingress_data</td><td>output</td><td>488</td><td>OoOStation(CxtMgt)</td><td>RX_RESP上下文请求数据</td></tr><tr><td>RX_RESP_fetch_cxt_ingress_start</td><td>output</td><td>1</td><td>OoOStation(CxtMgt)</td><td>RX_RESP上下文请求开始标志</td></tr><tr><td>RX_RESP_fetch_cxt_ingress_last</td><td>output</td><td>1</td><td>OoOStation(CxtMgt)</td><td>RX_RESP上下文请求结束标志</td></tr><tr><td>RX_RESP_fetch_cxt_ingress_ready</td><td>input</td><td>1</td><td>OoOStation(CxtMgt)</td><td>RX_RESP上下文请求就绪反压</td></tr><tr><td>RX_RESP_fetch_cxt_egress_valid</td><td>input</td><td>1</td><td>OoOStation(CxtMgt)</td><td>RX_RESP上下文响应有效</td></tr><tr><td>RX_RESP_fetch_cxt_egress_head</td><td>input</td><td>160</td><td>OoOStation(CxtMgt)</td><td>RX_RESP上下文响应头</td></tr><tr><td>RX_RESP_fetch_cxt_egress_data</td><td>input</td><td>640</td><td>OoOStation(CxtMgt)</td><td>RX_RESP上下文响应数据</td></tr><tr><td>RX_RESP_fetch_cxt_egress_start</td><td>input</td><td>1</td><td>OoOStation(CxtMgt)</td><td>RX_RESP上下文响应开始标志</td></tr><tr><td>RX_RESP_fetch_cxt_egress_last</td><td>input</td><td>1</td><td>OoOStation(CxtMgt)</td><td>RX_RESP上下文响应结束标志</td></tr><tr><td>RX_RESP_fetch_cxt_egress_ready</td><td>output</td><td>1</td><td>OoOStation(CxtMgt)</td><td>RX_RESP上下文响应就绪反压</td></tr><tr><td>RX_RESP_fetch_mr_ingress_valid</td><td>output</td><td>1</td><td>OoOStation(MRMgt)</td><td>RX_RESP内存区域请求有效</td></tr><tr><td>RX_RESP_fetch_mr_ingress_head</td><td>output</td><td>160</td><td>OoOStation(MRMgt)</td><td>RX_RESP内存区域请求头</td></tr><tr><td>RX_RESP_fetch_mr_ingress_data</td><td>output</td><td>352</td><td>OoOStation(MRMgt)</td><td>RX_RESP内存区域请求数据</td></tr><tr><td>RX_RESP_fetch_mr_ingress_start</td><td>output</td><td>1</td><td>OoOStation(MRMgt)</td><td>RX_RESP内存区域请求开始标志</td></tr><tr><td>RX_RESP_fetch_mr_ingress_last</td><td>output</td><td>1</td><td>OoOStation(MRMgt)</td><td>RX_RESP内存区域请求结束标志</td></tr><tr><td>RX_RESP_fetch_mr_ingress_ready</td><td>input</td><td>1</td><td>OoOStation(MRMgt)</td><td>RX_RESP内存区域请求就绪反压</td></tr><tr><td>RX_RESP_fetch_mr_egress_valid</td><td>input</td><td>1</td><td>OoOStation(MRMgt)</td><td>RX_RESP内存区域响应有效</td></tr><tr><td>RX_RESP_fetch_mr_egress_head</td><td>input</td><td>160</td><td>OoOStation(MRMgt)</td><td>RX_RESP内存区域响应头</td></tr><tr><td>RX_RESP_fetch_mr_egress_data</td><td>input</td><td>288</td><td>OoOStation(MRMgt)</td><td>RX_RESP内存区域响应数据</td></tr><tr><td>RX_RESP_fetch_mr_egress_start</td><td>input</td><td>1</td><td>OoOStation(MRMgt)</td><td>RX_RESP内存区域响应开始标志</td></tr><tr><td>RX_RESP_fetch_mr_egress_last</td><td>input</td><td>1</td><td>OoOStation(MRMgt)</td><td>RX_RESP内存区域响应结束标志</td></tr><tr><td>RX_RESP_fetch_mr_egress_ready</td><td>output</td><td>1</td><td>OoOStation(MRMgt)</td><td>RX_RESP内存区域响应就绪反压</td></tr><tr><td>RX_RESP_cq_req_valid</td><td>output</td><td>1</td><td>CompletionQueueMgt</td><td>RX_RESP CQ请求有效</td></tr><tr><td>RX_RESP_cq_req_head</td><td>output</td><td>64</td><td>CompletionQueueMgt</td><td>RX_RESP CQ请求头</td></tr><tr><td>RX_RESP_cq_req_ready</td><td>input</td><td>1</td><td>CompletionQueueMgt</td><td>RX_RESP CQ请求就绪反压</td></tr><tr><td>RX_RESP_cq_resp_valid</td><td>input</td><td>1</td><td>CompletionQueueMgt</td><td>RX_RESP CQ响应有效</td></tr><tr><td>RX_RESP_cq_resp_head</td><td>input</td><td>96</td><td>CompletionQueueMgt</td><td>RX_RESP CQ响应头</td></tr><tr><td>RX_RESP_cq_resp_ready</td><td>output</td><td>1</td><td>CompletionQueueMgt</td><td>RX_RESP CQ响应就绪反压</td></tr><tr><td>RX_RESP_eq_req_valid</td><td>output</td><td>1</td><td>EventQueueMgt</td><td>RX_RESP EQ请求有效</td></tr><tr><td>RX_RESP_eq_req_head</td><td>output</td><td>64</td><td>EventQueueMgt</td><td>RX_RESP EQ请求头</td></tr><tr><td>RX_RESP_eq_req_ready</td><td>input</td><td>1</td><td>EventQueueMgt</td><td>RX_RESP EQ请求就绪反压</td></tr><tr><td>RX_RESP_eq_resp_valid</td><td>input</td><td>1</td><td>EventQueueMgt</td><td>RX_RESP EQ响应有效</td></tr><tr><td>RX_RESP_eq_resp_head</td><td>input</td><td>96</td><td>EventQueueMgt</td><td>RX_RESP EQ响应头</td></tr><tr><td>RX_RESP_eq_resp_ready</td><td>output</td><td>1</td><td>EventQueueMgt</td><td>RX_RESP EQ响应就绪反压</td></tr><tr><td>RX_RESP_scatter_req_wen</td><td>output</td><td>1</td><td>DMA引擎</td><td>RX_RESP DMA写请求使能</td></tr><tr><td>RX_RESP_scatter_req_din</td><td>output</td><td>160</td><td>DMA引擎</td><td>RX_RESP DMA写请求数据(地址+长度)</td></tr><tr><td>RX_RESP_scatter_req_prog_full</td><td>input</td><td>1</td><td>DMA引擎</td><td>RX_RESP DMA写请求FIFO满</td></tr><tr><td>RX_RESP_scatter_data_wen</td><td>output</td><td>1</td><td>DMA引擎</td><td>RX_RESP DMA写数据使能</td></tr><tr><td>RX_RESP_scatter_data_din</td><td>output</td><td>512</td><td>DMA引擎</td><td>RX_RESP DMA写数据</td></tr><tr><td>RX_RESP_scatter_data_prog_full</td><td>input</td><td>1</td><td>DMA引擎</td><td>RX_RESP DMA写数据FIFO满</td></tr><tr><td>RX_RESP_delete_req_valid</td><td>output</td><td>1</td><td>PacketBuffer</td><td>RX_RESP删除包请求有效</td></tr><tr><td>RX_RESP_delete_req_head</td><td>output</td><td>24</td><td>PacketBuffer</td><td>RX_RESP删除包请求头(队列索引+槽位)</td></tr><tr><td>RX_RESP_delete_req_ready</td><td>input</td><td>1</td><td>PacketBuffer</td><td>RX_RESP删除包请求就绪反压</td></tr><tr><td>RX_RESP_delete_resp_valid</td><td>input</td><td>1</td><td>PacketBuffer</td><td>RX_RESP删除包响应有效</td></tr><tr><td>RX_RESP_delete_resp_start</td><td>input</td><td>1</td><td>PacketBuffer</td><td>RX_RESP删除包响应开始标志</td></tr><tr><td>RX_RESP_delete_resp_last</td><td>input</td><td>1</td><td>PacketBuffer</td><td>RX_RESP删除包响应结束标志</td></tr><tr><td>RX_RESP_delete_resp_data</td><td>input</td><td>512</td><td>PacketBuffer</td><td>RX_RESP删除包响应数据</td></tr><tr><td>RX_RESP_delete_resp_ready</td><td>output</td><td>1</td><td>PacketBuffer</td><td>RX_RESP删除包响应就绪反压</td></tr></tbody></table>


| 信号名 | 输入/输出 | 位宽 | 对接模块 | 中文说明 |
|-------|-----------|-------|-----------|-----------|
| clk | input | 1 | 全局时钟 | 时钟信号 |
| rst | input | 1 | 全局复位 | 复位信号 |
| TX_REQ_sub_wqe_valid | input | 1 | WQEParser | TX_REQ WQE有效信号 |
| TX_REQ_sub_wqe_meta | input | 576 | WQEParser | TX_REQ WQE元数据 |
| TX_REQ_sub_wqe_ready | output | 1 | WQEParser | TX_REQ WQE就绪反压 |
| TX_REQ_fetch_cxt_ingress_valid | output | 1 | OoOStation(CxtMgt) | TX_REQ上下文请求有效 |
| TX_REQ_fetch_cxt_ingress_head | output | 160 | OoOStation(CxtMgt) | TX_REQ上下文请求头 |
| TX_REQ_fetch_cxt_ingress_data | output | 576 | OoOStation(CxtMgt) | TX_REQ上下文请求数据 |
| TX_REQ_fetch_cxt_ingress_start | output | 1 | OoOStation(CxtMgt) | TX_REQ上下文请求开始标志 |
| TX_REQ_fetch_cxt_ingress_last | output | 1 | OoOStation(CxtMgt) | TX_REQ上下文请求结束标志 |
| TX_REQ_fetch_cxt_ingress_ready | input | 1 | OoOStation(CxtMgt) | TX_REQ上下文请求就绪反压 |
| TX_REQ_fetch_cxt_egress_valid | input | 1 | OoOStation(CxtMgt) | TX_REQ上下文响应有效 |
| TX_REQ_fetch_cxt_egress_head | input | 160 | OoOStation(CxtMgt) | TX_REQ上下文响应头 |
| TX_REQ_fetch_cxt_egress_data | input | 640 | OoOStation(CxtMgt) | TX_REQ上下文响应数据 |
| TX_REQ_fetch_cxt_egress_start | input | 1 | OoOStation(CxtMgt) | TX_REQ上下文响应开始标志 |
| TX_REQ_fetch_cxt_egress_last | input | 1 | OoOStation(CxtMgt) | TX_REQ上下文响应结束标志 |
| TX_REQ_fetch_cxt_egress_ready | output | 1 | OoOStation(CxtMgt) | TX_REQ上下文响应就绪反压 |
| TX_REQ_fetch_mr_ingress_valid | output | 1 | OoOStation(MRMgt) | TX_REQ内存区域请求有效 |
| TX_REQ_fetch_mr_ingress_head | output | 160 | OoOStation(MRMgt) | TX_REQ内存区域请求头 |
| TX_REQ_fetch_mr_ingress_data | output | 576 | OoOStation(MRMgt) | TX_REQ内存区域请求数据 |
| TX_REQ_fetch_mr_ingress_start | output | 1 | OoOStation(MRMgt) | TX_REQ内存区域请求开始标志 |
| TX_REQ_fetch_mr_ingress_last | output | 1 | OoOStation(MRMgt) | TX_REQ内存区域请求结束标志 |
| TX_REQ_fetch_mr_ingress_ready | input | 1 | OoOStation(MRMgt) | TX_REQ内存区域请求就绪反压 |
| TX_REQ_fetch_mr_egress_valid | input | 1 | OoOStation(MRMgt) | TX_REQ内存区域响应有效 |
| TX_REQ_fetch_mr_egress_head | input | 160 | OoOStation(MRMgt) | TX_REQ内存区域响应头 |
| TX_REQ_fetch_mr_egress_data | input | 576 | OoOStation(MRMgt) | TX_REQ内存区域响应数据 |
| TX_REQ_fetch_mr_egress_start | input | 1 | OoOStation(MRMgt) | TX_REQ内存区域响应开始标志 |
| TX_REQ_fetch_mr_egress_last | input | 1 | OoOStation(MRMgt) | TX_REQ内存区域响应结束标志 |
| TX_REQ_fetch_mr_egress_ready | output | 1 | OoOStation(MRMgt) | TX_REQ内存区域响应就绪反压 |
| TX_REQ_cq_req_valid | output | 1 | CompletionQueueMgt | TX_REQ CQ请求有效 |
| TX_REQ_cq_req_head | output | 64 | CompletionQueueMgt | TX_REQ CQ请求头 |
| TX_REQ_cq_req_ready | input | 1 | CompletionQueueMgt | TX_REQ CQ请求就绪反压 |
| TX_REQ_cq_resp_valid | input | 1 | CompletionQueueMgt | TX_REQ CQ响应有效 |
| TX_REQ_cq_resp_head | input | 96 | CompletionQueueMgt | TX_REQ CQ响应头 |
| TX_REQ_cq_resp_ready | output | 1 | CompletionQueueMgt | TX_REQ CQ响应就绪反压 |
| TX_REQ_eq_req_valid | output | 1 | EventQueueMgt | TX_REQ EQ请求有效 |
| TX_REQ_eq_req_head | output | 64 | EventQueueMgt | TX_REQ EQ请求头 |
| TX_REQ_eq_req_ready | input | 1 | EventQueueMgt | TX_REQ EQ请求就绪反压 |
| TX_REQ_eq_resp_valid | input | 1 | EventQueueMgt | TX_REQ EQ响应有效 |
| TX_REQ_eq_resp_head | input | 96 | EventQueueMgt | TX_REQ EQ响应头 |
| TX_REQ_eq_resp_ready | output | 1 | EventQueueMgt | TX_REQ EQ响应就绪反压 |
| TX_REQ_gather_req_wr_en | output | 1 | DMA引擎 | TX_REQ DMA读请求使能 |
| TX_REQ_gather_req_din | output | 160 | DMA引擎 | TX_REQ DMA读请求数据(地址+长度) |
| TX_REQ_gather_req_prog_full | input | 1 | DMA引擎 | TX_REQ DMA读请求FIFO满 |
| TX_REQ_net_data_rd_en | output | 1 | GatherData | TX_REQ网络数据读使能 |
| TX_REQ_net_data_dout | input | 512 | GatherData | TX_REQ网络数据输出 |
| TX_REQ_net_data_empty | input | 1 | GatherData | TX_REQ网络数据空标志 |
| TX_REQ_scatter_req_wen | output | 1 | DMA引擎 | TX_REQ DMA写请求使能 |
| TX_REQ_scatter_req_din | output | 160 | DMA引擎 | TX_REQ DMA写请求数据(地址+长度) |
| TX_REQ_scatter_req_prog_full | input | 1 | DMA引擎 | TX_REQ DMA写请求FIFO满 |
| TX_REQ_scatter_data_wen | output | 1 | DMA引擎 | TX_REQ DMA写数据使能 |
| TX_REQ_scatter_data_din | output | 512 | DMA引擎 | TX_REQ DMA写数据 |
| TX_REQ_scatter_data_prog_full | input | 1 | DMA引擎 | TX_REQ DMA写数据FIFO满 |
| TX_REQ_insert_req_valid | output | 1 | PacketBuffer | TX_REQ插入包有效 |
| TX_REQ_insert_req_start | output | 1 | PacketBuffer | TX_REQ插入包开始标志 |
| TX_REQ_insert_req_last | output | 1 | PacketBuffer | TX_REQ插入包结束标志 |
| TX_REQ_insert_req_head | output | 14 | PacketBuffer | TX_REQ插入包头(槽位索引) |
| TX_REQ_insert_req_data | output | 512 | PacketBuffer | TX_REQ插入包数据 |
| TX_REQ_insert_req_ready | input | 1 | PacketBuffer | TX_REQ插入包就绪反压 |
| TX_REQ_insert_resp_valid | input | 1 | PacketBuffer | TX_REQ插入包响应有效 |
| TX_REQ_insert_resp_data | input | 14 | PacketBuffer | TX_REQ插入包响应数据(槽位索引) |
| TX_REQ_egress_pkt_valid | output | 1 | TransportSubsystem | TX_REQ出口包有效 |
| TX_REQ_egress_pkt_head | output | 488 | TransportSubsystem | TX_REQ出口包头(元数据) |
| TX_REQ_egress_pkt_ready | input | 1 | TransportSubsystem | TX_REQ出口包就绪反压 |
| RX_RESP_ingress_pkt_valid | input | 1 | PacketDeparser | RX_RESP入口包有效 |
| RX_RESP_ingress_pkt_head | input | 488 | PacketDeparser | RX_RESP入口包头(元数据) |
| RX_RESP_ingress_pkt_ready | output | 1 | PacketDeparser | RX_RESP入口包就绪反压 |
| RX_RESP_fetch_cxt_ingress_valid | output | 1 | OoOStation(CxtMgt) | RX_RESP上下文请求有效 |
| RX_RESP_fetch_cxt_ingress_head | output | 160 | OoOStation(CxtMgt) | RX_RESP上下文请求头 |
| RX_RESP_fetch_cxt_ingress_data | output | 488 | OoOStation(CxtMgt) | RX_RESP上下文请求数据 |
| RX_RESP_fetch_cxt_ingress_start | output | 1 | OoOStation(CxtMgt) | RX_RESP上下文请求开始标志 |
| RX_RESP_fetch_cxt_ingress_last | output | 1 | OoOStation(CxtMgt) | RX_RESP上下文请求结束标志 |
| RX_RESP_fetch_cxt_ingress_ready | input | 1 | OoOStation(CxtMgt) | RX_RESP上下文请求就绪反压 |
| RX_RESP_fetch_cxt_egress_valid | input | 1 | OoOStation(CxtMgt) | RX_RESP上下文响应有效 |
| RX_RESP_fetch_cxt_egress_head | input | 160 | OoOStation(CxtMgt) | RX_RESP上下文响应头 |
| RX_RESP_fetch_cxt_egress_data | input | 640 | OoOStation(CxtMgt) | RX_RESP上下文响应数据 |
| RX_RESP_fetch_cxt_egress_start | input | 1 | OoOStation(CxtMgt) | RX_RESP上下文响应开始标志 |
| RX_RESP_fetch_cxt_egress_last | input | 1 | OoOStation(CxtMgt) | RX_RESP上下文响应结束标志 |
| RX_RESP_fetch_cxt_egress_ready | output | 1 | OoOStation(CxtMgt) | RX_RESP上下文响应就绪反压 |
| RX_RESP_fetch_mr_ingress_valid | output | 1 | OoOStation(MRMgt) | RX_RESP内存区域请求有效 |
| RX_RESP_fetch_mr_ingress_head | output | 160 | OoOStation(MRMgt) | RX_RESP内存区域请求头 |
| RX_RESP_fetch_mr_ingress_data | output | 352 | OoOStation(MRMgt) | RX_RESP内存区域请求数据 |
| RX_RESP_fetch_mr_ingress_start | output | 1 | OoOStation(MRMgt) | RX_RESP内存区域请求开始标志 |

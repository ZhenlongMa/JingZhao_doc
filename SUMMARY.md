# Table of contents

* [京兆2号高性能网卡说明](README.md)
* [概述](manual/manual.md)

## 硬件设计 <a href="#hw_design" id="hw_design"></a>

* [总体设计](hw_design/hw_design.md)
* [RDMA Core](hw_design/rdma-core/README.md)
  * [RequesterCore](hw_design/rdma-core/requestercore/README.md)
    * [RespRecvCore](hw_design/rdma-core/requestercore/resprecvcore/README.md)
      * [RespRecvCore\_Thread\_1](hw_design/rdma-core/requestercore/resprecvcore/resprecvcore_thread_1.md)
      * [RespRecvCore\_Thread\_2](hw_design/rdma-core/requestercore/resprecvcore/resprecvcore_thread_2.md)
      * [RespRecvCore\_thread\_3](hw_design/rdma-core/requestercore/resprecvcore/resprecvcore_thread_3.md)
    * [ReqTransCore](hw_design/rdma-core/requestercore/reqtranscore/README.md)
      * [ReqTransCore\_Thread\_1](hw_design/rdma-core/requestercore/reqtranscore/reqtranscore_thread_1.md)
      * [ReqTransCore\_Thread\_2](hw_design/rdma-core/requestercore/reqtranscore/reqtranscore_thread_2.md)
      * [ReqTransCore\_Thread\_3](hw_design/rdma-core/requestercore/reqtranscore/reqtranscore_thread_3.md)
      * [ReqTransCore\_Thread\_4](hw_design/rdma-core/requestercore/reqtranscore/reqtranscore_thread_4.md)
  * [ResponderCore](hw_design/rdma-core/respondercore/README.md)
    * [ReqRecvCore](hw_design/rdma-core/respondercore/reqrecvcore/README.md)
      * [ReqRecvCore\_Thread\_1](hw_design/rdma-core/respondercore/reqrecvcore/reqrecvcore_thread_1.md)
      * [ReqRecvCore\_Thread\_2](hw_design/rdma-core/respondercore/reqrecvcore/reqrecvcore_thread_2.md)
      * [ReqRecvCore\_Thread\_3](hw_design/rdma-core/respondercore/reqrecvcore/reqrecvcore_thread_3.md)
    * [RespTransCore](hw_design/rdma-core/respondercore/resptranscore/README.md)
      * [RespTransCore\_Thread\_1](hw_design/rdma-core/respondercore/resptranscore/resptranscore_thread_1.md)
* [Queue Subsystem](hw_design/Queue_Subsystem/que_subsys.md)
  * [RQ Management](hw_design/Queue_Subsystem/RQ_Management.md)
  * [SQ Management](hw_design/Queue_Subsystem/SQ_Management.md)
* [Resource Manage](hw_design/Resource_Management/resc_manage.md)
* [Transport Subsystem](hw_design/Transport_Subsystem/trans_subsys.md)

## 软件设计

* [内核态驱动](software/kernel_driver.md)
* [用户态驱动](software/user_driver.md)

## 硬件仿真

* [硬件仿真](hw_simulation/hw_simulation.md)

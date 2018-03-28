# YARN介绍

​	Apache Hadoop YARN（Yet Another Resource Negotiator，另一种资源协调者）是一种全新的Hadoop资源管理器，是一个通用的资源管理系统，可为上层应用提供统一的资源管理和调度，它的引入为集群在利用率、资源统一管理和数据共享等方面带来巨大的好处。因此再Hadoop2.0中YARN是最核心的，下图是Hadoop1.0和Hadoop2.0的区别。

![images](https://github.com/WatermelonAI/Spark-/blob/master/images/hadoop1_hadoop2.png)

![images](https://github.com/WatermelonAI/Spark-/blob/master/images/hadoop2_yarn.png)

​	从上图我们可以将YARN看作为一个操作系统，以往的操作系统是单机安装的，而YARN这个操作系统是可以看作安装在集群上的操作系统。而MapReduce可以看作安装在操作系统YARN上一个软件，该系统的好处是当我们跑MapReduce的时候我们可以将一些资源通过调度来给其他的一些软件使用比如Spark，Kafka等等，因此可以提高利用效率。

​	YARN主要包含三大模块：ResourceManager（RM）、NodeManager（NM）、ApplicationMaster（AM）



## ResourceManger介绍

1、RM负责整个集群的资源管理和分配，是一个全局的资源管理系统

2、NM一心跳的方式向RM汇报资源使用情况（主要是CPU和内存的使用情况）。RM只接受NM的资源回报信息，对于具体的资源处理则交给NM自己处理。

3、YARN Scheduler根据application的请求为其分配资源，不负责application job的监控、追踪、运行状态反馈、启动等工作。

## NodeManger介绍

1、NM是每个节点上的资源和任务管理器，它管理这台机器的代理，负责该节点程序的运行，以及该节点的资源弄得管理和监控。YARN集群每个节点都运行一个NM

2、NM定时向RM汇报本节点资源（CPU、内存）的使用情况和container的运行状态。当RM宕机的时候NM自动连接RM备用节点。

3、NM接受并处理AM的container启动、停止等各种请求。

## ApplicationMaster介绍

1、用户提交的每个应用程序均包含一个AM，它可以运行在RM以外的机器上。

2、负责与RM调度器协商以获取资源（用container来表示）

3、将得到的任务进一步分配给内部的任务（资源的二次分配）

4、与NM通信以启动/停止任务

5、监控所有的任务的运行状态，并在任务运行失败时重新为任务申请资源以重启任务。

6、当前YARN自带了两个AM实现，一个用于演示AM编写方法的实例程序，DistributedShell，它可以申请一定数目的Container以并行运行一个shell命令或则和shell脚本；另一个时运行MapReduce应用程序的AM-MRAppMaster

注：RM只负责监控AM，并在AM运行失败的时候启动它，RM不负责AM内部任务的容错，任务的容错由AM自己完成。

## YARN运行流程

![images](https://github.com/WatermelonAI/Spark-/blob/master/images/yarn_operation_flow.png)

1、client向RM提交应用程序，其中包括启动该应用的ApplicationMaster的必须信息，例如ApplicationMater程序，启动ApplicationMaster的命令、用户程序等。

2、RM启动一个container用于运行AM

3、启动中的AM下个RM注册自己，启动成功后与RM保持心跳

4、AM向RM发送请求，申请相应数目的container。

5、RM返回AM的申请的containers信息。申请成功的container，由AM进行初始化。container启动信息初始化后，AM与对应的NM通信，要求NM启动container。AM与NM保持心跳，从而对NM上运行的任务进行监控。

6、container运行期间，ApplicationMaster对container进行监控。container通过RPC协议向对应的AM汇报自己的进度和状态等信息。

7、应用运行期间，clien直接与AM通信获取应用的状态，进度更新等信息。8、应用运行结束后，AM向RM注销自己，并允许属于它的container被收回。

$\sum_{i=0}^N\int_{a}^{b}g(t,i)\text{d}t$


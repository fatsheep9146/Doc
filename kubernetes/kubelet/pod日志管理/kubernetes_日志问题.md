kubernetes 日志系统

kubernetes 日志方案一直处于比较初级的阶段，并且有很多场景目前都没有很好的解决方案。

这篇文档，主要记录社区对日志方案的一些讨论。

首先日志主要分为两种不同的来源

- 容器标准输出，标准错误输出
- 容器内文件日志

这对这两种日志来源都有一些统一的需求

https://github.com/kubernetes/kubernetes/issues/17183#issuecomment-210090673

- 日志的资源隔离，每个容器日志占用的磁盘，网络都能够进行隔离，限制
- 日志生命周期单独管理，能够维持历史日志
- 支持对日志进行打标记，并且能够和一些 k8s 的原信息进行关联

但是目前的现况是

针对标准输出/标准错误输出 [2]

- 直接占用 docker 的业务盘空间，磁盘占用不方便统计
- 和容器生命周期绑定，不方便管理
- k8s 目前的 log 的 api 需要访问 node 上面的 log，所以那些不会落盘的日志方案不好作为替代方案

针对文件日志

- 根本没有一个很好的方案

社区的方向

针对任意来源的日志，我们都希望日志系统能够

- 将日志直接用本地进行存储
- 为日志添加元信息
- 对日志支持简单的回滚，但是这里有个小问题，kubelet 如果不感知日志的格式，可能会做出错误的回滚。
- 不对日志内容进行解析
- 能够限制日志对应磁盘的大小和iops
- 将日志和容器的生命周期解绑

有的团队对于日志的需求进行进一步细化，并且设计的实现的优先级[[5]](https://docs.google.com/document/d/1K2hh7nQ9glYzGE-5J7oKBB7oK3S_MKqwCISXZK-sB2Q/edit#)

至于目前具体的实现方式，vishh 在 issue 中给出了初步的[设想](https://github.com/kubernetes/kubernetes/issues/24677#issuecomment-231479502)。

对于某一个 pod 的所有日志都会通过某种方式重定向到某一个路径下面，并且会区分标准输出，标准错误，已经文件日志，还有存放这些日志对应的原信息文件。并且这个路径是由 kubelet 决定，并且告知 runtime。由于 kubelet 知道这些路径信息，就可以对这些日志进行生命周期的管理了。

```
/var/log/pod/containerName_instanceID_stdout.log
/var/log/pod/loggingVolume/user_dir/user_generated_file.log
/var/log/pod/.metadata
``` 

那么针对实现社区中仍旧在讨论着一些问题

1. 对于标准输出日志到底通过是以怎样的形式落盘，社区中仍旧存在两种声音

- 一种认为就应该直接落盘 [https://github.com/kubernetes/kubernetes/issues/24677#issuecomment-213598517](https://github.com/kubernetes/kubernetes/issues/24677#issuecomment-213598517)
- 第二种认为可以考虑使用 journald，并且他们提出 journald 已经被 coreos 进行改造。[https://github.com/kubernetes/kubernetes/issues/24677#issuecomment-242957812](https://github.com/kubernetes/kubernetes/issues/24677#issuecomment-242957812)


2. 为 log 实现一套单独的 api

这个需求主要源自对 container runtime 的 interface 重构，log interface 只是其中的一个子部分，这是 [根 issue](https://github.com/kubernetes/kubernetes/issues/22964) [根 issue 的需求草稿文档](https://docs.google.com/document/d/1ietD5eavK0aTuMQTw6-21r67UU73_vqYSUIPFdA0J5Q/edit#heading=h.ky2glvbgtu52)，后续的[追踪 issue](https://github.com/kubernetes/kubernetes/issues/28789) 。

根 issue 和 追踪 issue 中提到了很多需要改造的 interface 其中就包括了 log interface。因为 log interface 其实也是维护整个生态的一个非常重要的 api 类型，所以也需要尽量设计的完善。

目前对于 log api 的设计的相关 issue 如下：

- https://github.com/kubernetes/kubernetes/issues/30709
- https://github.com/kubernetes/kubernetes/pull/33111
- https://github.com/kubernetes/kubernetes/pull/34376

其中在 33111 中，yujuhong 整理了对于这个 log api PR 根本的[需求](https://github.com/kubernetes/kubernetes/pull/33111#issuecomment-248708235) 

## issue 历史追踪

https://github.com/kubernetes/kubernetes/pull/13010

在这个 issue 中，题主提出了当前对 k8s 容器内文件日志的收集没有一套非常完备的方案。所以希望引入一个 LogDir 的 Volume 类型。

https://github.com/kubernetes/kubernetes/pull/33111

这个是针对重新定义 container interface 的大需求中，针对 Log 相关的 api 的重新设计的初步 proposal。讨论主题比较宽泛。

并且这个 PR 最终由于其他人针对 stdout/stderr 提出了比较好的设计方案，并且也没有针对 CRI 特性进行设计，所以被关闭。但是 log volume 方面还是没有很好地继续推进。

https://github.com/kubernetes/kubernetes/pull/34376

这个 pr 中提出了有关 stdout/stderr 的第一版实现设计，结合了有关 CRI 的交互。

主要提出 3 点日志需求：

- 继续满足 kubectl logs 提供的所有功能
- kubelet 管理日志的生命周期，资源使用，日志与容器生命周期解绑。
- 允许开源日志收集组件能够快速便捷的和 k8s 日志整合

但是这个文档只满足了两个需求

- 让日志落在机器的文件系统上面，并且 kubelet ，日志收集组件都能找到它们。
- 让 runtime 把用户日志修改为 kubernetes 能理解的方式。

整体来讲， kubelet 会没每个容器创建一个特定的日志路径。并且要求容器运行时来把日志进行修改，满足需要的格式。

https://github.com/kubernetes/kubernetes/pull/34858

https://github.com/kubernetes/kubernetes/pull/35348

这两 pr 将开始实现上一个 pr 的设计文档中提出的思路

相关测试 PR

https://github.com/kubernetes/kubernetes/issues/34661

https://github.com/kubernetes/kubernetes/pull/34877

https://github.com/kubernetes/kubernetes/pull/36637

至此 client 端(kubelet)的重构已经完成，还需要 server 端(docker)来完成日志格式改造方面的工作。







## 引用

1. Add LogDir: https://github.com/kubernetes/kubernetes/pull/13010
2. Log management roadmap: https://github.com/kubernetes/kubernetes/issues/17183
3. RFC: Handling logs in the future: https://github.com/kubernetes/kubernetes/issues/13846
4. https://github.com/kubernetes/kubernetes/issues/24677
5. https://docs.google.com/document/d/1K2hh7nQ9glYzGE-5J7oKBB7oK3S_MKqwCISXZK-sB2Q/edit#

## recently

https://github.com/kubernetes/kubernetes/issues/24677#issuecomment-242957812

话题开始转向如何结合 journald 来进行使用
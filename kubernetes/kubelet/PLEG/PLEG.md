PLEG 工作流程

# 简述

kubelet 作为它所在机器上所有 k8s 集群内的 pod 管理者。它不仅仅需要时刻 watch pod spec 的变化，还要及时的获取当前机器上面所有 container 的状态变化，从而对 Pod 原信息进行及时的更新。

所以 PLEG 模块是 kubernetes 中非常重要的一个模块，也是 kubelet 主工作循环中非常重要的一个事件生产者。kubelet 可以通过它获取 container 的 status 的变化，从而触发相应的事件。

所以 PLGE 的可扩展性非常关键，因为大量并发的对 container runtime 的请求很有可能使 CPU 利用率显著提升，引起机器的不稳定。


# 基本工作流程

整个 PLEG 的工作流程是一个被无限执行的循环，在每一次循环中都需要完成几个动作

1. 首先获取当前机器所有容器的状态
2. 根据刚刚获取的容器状态，结合在内存中缓存的容器状态，得出要被触发的行为。
3. 所有有事件与之关联的容器，需要重新具体查看它的信息，
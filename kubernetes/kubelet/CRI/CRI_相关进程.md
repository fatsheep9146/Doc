# CRI

### 第一次将重新设计容器运行时接口作为问题抛到社区

https://github.com/kubernetes/kubernetes/issues/22964

### 随后第一次提出接口设计方案

https://github.com/kubernetes/kubernetes/pull/25899

其中包含了设计文档已经第一版接口 interface 文件。

### 之后又建立了 PR 用于记录和规划整个特性的实现

https://github.com/kubernetes/kubernetes/issues/28789

其中包括了很多子问题，分支问题的讨论与实现

### 问题1 提供 kubelet 和 runtime 之间的一个适配层

https://github.com/kubernetes/kubernetes/pull/28396

https://github.com/kubernetes/kubernetes/pull/29824

在 pkg/kubelet/container/runtime.go 文件中定义了期望底层 runtime 需要实现的 interface 列表

```
// Runtime interface defines the interfaces that should be implemented
// by a container runtime.
// Thread safety is required from implementations of this interface.
type Runtime interface {
	Type() string
	Version() (Version, error)
	APIVersion() (Version, error)
	Status() (*RuntimeStatus, error)
	GetPods(all bool) ([]*Pod, error)
	GarbageCollect(gcPolicy ContainerGCPolicy, allSourcesReady bool, evictNonDeletedPods bool) error
	SyncPod(pod *v1.Pod, apiPodStatus v1.PodStatus, podStatus *PodStatus, pullSecrets []v1.Secret, backOff *flowcontrol.Backoff) PodSyncResult
	KillPod(pod *v1.Pod, runningPod Pod, gracePeriodOverride *int64) error
	GetPodStatus(uid types.UID, name, namespace string) (*PodStatus, error)
	GetNetNS(containerID ContainerID) (string, error)
	GetPodContainerID(*Pod) (ContainerID, error)
	GetContainerLogs(pod *v1.Pod, containerID ContainerID, logOptions *v1.PodLogOptions, stdout, stderr io.Writer) (err error)
	DeleteContainer(containerID ContainerID) error	ImageService
	UpdatePodCIDR(podCIDR string) error
}
```

至于这个接口背后怎么实现，kubelet 中将会将会创建一个 kubeGenericRuntimeManager 类型的对象，该对象将把假设依赖于实现了 CRI 接口的 runtime 代码，从而进一步解释 Runtime 接口中的功能。

这也就是这个 PR 着重创建的部分，就是这个 kubeGenericRuntimeManager 对象，以及他是如何调用底层 CRI 接口，来实现 runtime 接口的。

这个 PR 只是一个框架的构建，很多 runtime 接口的具体实现还没有真实落实。后续的几个 PR 就是用于落实每一个接口的实现。

- 实现 Labels 相关接口：[#30049](https://github.com/kubernetes/kubernetes/pull/30049
)
- 实现生成 Sandbox/Pod 配置的相关接口[#30083](https://github.com/kubernetes/kubernetes/pull/30083)
- 实现生成 GetPods 相关信息的接口 [#30121](https://github.com/kubernetes/kubernetes/pull/30121)
- 添加可以对 runtime 访问方式的配置标记 [#30212](https://github.com/kubernetes/kubernetes/pull/30212)
- 添加对 GetPodStatus 的支持 [#30267](https://github.com/kubernetes/kubernetes/pull/30267)



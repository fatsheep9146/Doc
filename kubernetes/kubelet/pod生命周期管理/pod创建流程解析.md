# pod 创建流程解析

这篇文档将主要关注有关 pod 的生命周期管理。将主要解析 kubelet 代码中有关 pod 的创建，删除，更新等操作背后的实现。

# 1 简介

Pod 的生命周期主要通过它所在机器的 kubelet 进行管理。在介绍 kubelet 整体框架的时候也介绍过，kubelet 实现了一系列 Handler，用于应对处理这个 Pod 上面发生的不同的事件，比如创建，更新，删除。

并且 kubelet 也把这一系列 Handler 都抽象到一个接口中

```
pkg/kubelet/kubelet.go

// SyncHandler is an interface implemented by Kubelet, for testability
type SyncHandler interface {
	HandlePodAdditions(pods []*v1.Pod)
	HandlePodUpdates(pods []*v1.Pod)
	HandlePodRemoves(pods []*v1.Pod)
	HandlePodReconcile(pods []*v1.Pod)
	HandlePodSyncs(pods []*v1.Pod)
	HandlePodCleanups() error
}

```

并且 kubelet 对象自身实现了这些方法。

下面我们将具体看一下，每一种 Handler 背后都做了什么事情。

# 2 pod 创建

## 2.1 HandlePodAdditions

我们首先看一下用于处理 Pod 创建事件的 HandlePodAdditions 函数。

具体函数看这一小节的末尾

该函数中比较重要的几个操作

- 将 pod 的元信息写入 kubelet.podManager 缓存中；podManager 就会开始管理该 Pod 对象以及它的 secret 和 configmap
- 判断这个 pod 是不是一个已经被删除的 pod，如果不是，则这个 pod 的创建动作需要通过 canAdmitPod 函数来进行确认
- kubelet 会把这个 Pod 的创建工作通过 dispatchWork 函数下发下去
- kubelet.probeManager 会开始对这个 pod 配置的 liveness，readiness 配置进行轮询。

```
// HandlePodAdditions is the callback in SyncHandler for pods being added from
// a config source.
func (kl *Kubelet) HandlePodAdditions(pods []*v1.Pod) {
	...
	for _, pod := range pods {
		...
		kl.podManager.AddPod(pod)

		...

		if !kl.podIsTerminated(pod) {
			...
			activePods := kl.filterOutTerminatedPods(existingPods)

			// Check if we can admit the pod; if not, reject it.
			if ok, reason, message := kl.canAdmitPod(activePods, pod); !ok {
				kl.rejectPod(pod, reason, message)
				continue
			}
		}
		
		...
		kl.dispatchWork(pod, kubetypes.SyncPodCreate, mirrorPod, start)
		kl.probeManager.AddPod(pod)
	}
}
```

## 2.2 dispatchWork

该函数是 HandlePodAdditions 中调用的用于执行 Pod 创建的函数。而且这个函数是一个比较通用的函数，主要就是将调用者发送过来的对 pod 的不同事件都通过 kubelet.podWorker 对象的 UpdatePod 方法进行处理

## 2.3 UpdatePod 

该函数是 kubelet.podWorker 对象用于完成对 pod 的生命周期管理的关键函数。

在 podWorker 中针对每一个 Pod 都会有一个 goroutine，这个 goroutine 将以一个 chan 作为事件源，不停的处理对这个 Pod 的新的 Sync 请求，请求可能包含多种类型，比如创建，更新，删除。

在 Pod 第一次被创建的时候，这个 Pod 将会被分配一个 chan，作为后续的事件源，并且在 podWorker 中也会维护一个 podid 与 chan 之间的对应关系。然后每一次有新的和这个 Pod 相关的事件发生时，podWorker 就会根据这个对应关系找到 pod 的 chan，将事件传递进去。

另外，有可能在 Pod goroutine 处理某一个事件的中间，又发生了一个新的事件。针对这个问题，podWorker 维护了一个 IsWorking 的 map，以及一个 lastUndeliveredWorkUpdate 的 map。

IsWorking map 用来表明这个 Pod Work goroutine 是否正在处理某一个事件中。
而 lastUndeliveredWorkUpdate map 中则保存，如果 pod work goroutine 正在处理事件的话，这里会暂存一个以及提交过来的，但是因为 pod work 正忙无法及时处理的事件。

可见 podWorker 对于同一个 Pod 的多个未处理事件的处理逻辑就是覆盖，只做最后一个。
当然有一个例外，如果事件是杀死 Pod，那么这个事件要么被处理，要么会一直在 lastUndeliveredWorkUpdate 中。

```
func (p *podWorkers) UpdatePod(options *UpdatePodOptions) {
	pod := options.Pod
	uid := pod.UID
	var podUpdates chan UpdatePodOptions
	var exists bool

	p.podLock.Lock()
	defer p.podLock.Unlock()
	if podUpdates, exists = p.podUpdates[uid]; !exists {
		...
		podUpdates = make(chan UpdatePodOptions, 1)
		p.podUpdates[uid] = podUpdates

		...
		go func() {
			defer runtime.HandleCrash()
			p.managePodLoop(podUpdates)
		}()
	}
	if !p.isWorking[pod.UID] {
		p.isWorking[pod.UID] = true
		podUpdates <- *options
	} else {
		...
		update, found := p.lastUndeliveredWorkUpdate[pod.UID]
		if !found || update.UpdateType != kubetypes.SyncPodKill {
			p.lastUndeliveredWorkUpdate[pod.UID] = *options
		}
	}
}
```

## 2.4 managePodLoop

在 Update 中真实启动一个 Pod Work goroutine 是通过 managePodLoop 函数完成。

这个函数的逻辑就比较简单了，就是两件事情

- 调用 kubelet 的 syncPodFn 函数完成对这个 Pod 的事件的同步操作
- 同步完成后处理后续的一些事宜，比如将待完成的 lastUndeliveredWorkUpdate 中的事件传入 chan，并且将 IsWorking 字段设置为 false

## 2.5 syncPod

下面才真正进入创建 Pod 的实操阶段。

其中创建的流程主要涉及到以下操作

- 根据内部获取的 container 状态，更新 pod 里面的 podStatus 参数，生成这个 Pod 的最新 PodStatus：``apiPodStatus := kl.generateAPIPodStatus(pod, ``podStatus)
- 将这个最新的 PodStatus 通过 StatusManager 同步到 Apiserver：``kl.statusManager.SetPodStatus(pod, apiPodStatus)``
- 为这个 pod 创建相应的 pod cgroup 以及更新 QoS level cgroup：``kl.containerManager.UpdateQOSCgroups();  pcm.EnsureExists(pod)``
- 创建存放 pod 信息的数据目录 ``kl.makePodDataDirs(pod)``
- 等待 kubelet 内的 volumeManager 子模块为这个 pod mount 好所有它需要的 volume 到本地
- 调用 kubelet 内的 containerRuntime 子模块来对 Pod 进行和运行时交互下的创建，更新，删除等操作


## 2.6 SyncPod

在上一个 syncPod 函数中，最关键的步骤就是调用 kubelet.containerRuntime 子模块的 SyncPod 函数，真正完成 Pod 内容器实例的创建。

这个函数的实现位于 pkg/kubelet/kuberuntime/kuberuntime_manager.go 中

通过注释就可以看到，这个函数主要完成的步骤，

```
// SyncPod syncs the running pod into the desired pod by executing following steps:
//
//  1. Compute sandbox and container changes.
//  2. Kill pod sandbox if necessary.
//  3. Kill any containers that should not be running.
//  4. Create sandbox if necessary.
//  5. Create init containers.
//  6. Create normal containers.
func (m *kubeGenericRuntimeManager) SyncPod(pod *v1.Pod, _ v1.PodStatus, podStatus *kubecontainer.PodStatus, pullSecrets []v1.Secret, backOff *flowcontrol.Backoff) (result kubecontainer.PodSyncResult) {
	// Step 1: Compute sandbox and container changes.
	podContainerChanges := m.computePodContainerChanges(pod, podStatus)	...
	
	// Step 2: Kill the pod if the sandbox has changed.
	if podContainerChanges.CreateSandbox || (len(podContainerChanges.ContainersToKeep) == 0 && len(podContainerChanges.ContainersToStart) == 0) {
	...
	killResult := m.killPodWithSyncResult(pod, kubecontainer.ConvertPodStatusToRunningPod(m.runtimeName, podStatus), nil)
	...
	} else {
	// Step 3: kill any running containers in this pod which are not to keep.
	for containerID, containerInfo := range podContainerChanges.ContainersToKill {
	...
	if err := m.killContainer(pod, containerID, containerInfo.name, containerInfo.message, nil); err != nil {
	...
	
	// Step 4: Create a sandbox for the pod if necessary.
	...
	podSandboxID, msg, err = m.createPodSandbox(pod, podContainerChanges.Attempt)
	...
	
	// Step 5: start init containers.
	status, next, done := findNextInitContainerToRun(pod, podStatus)
	if msg, err := m.startContainer(podSandboxID, podSandboxConfig, container, pod, podStatus, pullSecrets, podIP); err != nil {
	...
	
	// Step 6: start containers in podContainerChanges.ContainersToStart.
	for idx := range podContainerChanges.ContainersToStart {
	...
	if msg, err := m.startContainer(podSandboxID, podSandboxConfig, container, pod, podStatus, pullSecrets, podIP); err != nil {
	...

}
```


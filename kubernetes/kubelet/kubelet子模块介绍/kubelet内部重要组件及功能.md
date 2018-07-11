kubelet 内部重要组件及功能

# ContainerManager

### 主要功能

- 负责管理该机器上所有的 Cgroup 的配置
- 负责对 pod 的 Cgroup 配置的创建，更新
- 负责保证对 QoS Level Cgroup 的及时更新


# containerRuntime

### 主要功能

- 负责对接 kubelet 和 container runtime 之间的对接，并且使 container runtime developer 能够专注于实现 CRI 接口的研发，无需关注 k8s 逻辑。

# podManager

### 主要功能

- 作为这台机器上由 kubelet 管理的所有 Pod 的信息存储源。
- 管理所有 pod 的 secret 和 configmap

# statusManager

### 主要功能

- 主要负责跟 apiserver 同步 kubelet 感知到的 pod status 信息的更新
- 另外也会周期性的同步所有 pod 的 status 信息

# podWorker

### 主要功能

- 真实执行对容器的具体操作的对象 



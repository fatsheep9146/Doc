kubelet源码解析

## kubelet重要默认配置

cadvisor port: 4194
healthz port: 10248
默认配置值所在文件路径
kubernetes/kubernetes/pkg/api/componentconfig


## kubelet源码：重要类型

type KubeletServer struct {
    KubeletFlags
    componentconfig.KubeletConfiguration
}

KubeletFlags中存放的是有关kubelet server不在运行时动态改动的flag，或者是不能同时被多个节点共享的性质。
KubeletConfiguration存放的是会动态改动的配置。

## kubelet源码：重要方法解析

### 启动 kubelet函数

// kubelet/app/server.go

app.run(s *options.KubeletServer, kubeDeps *kubelet.Dependencies) 

功能：启动kubelet函数

描述：

这是整个kubelet启动时所运行的函数，其中包含很多初始化工作。

其中比较重要的初始化函数就是对这个kubelet的kubeDeps组件的操作。

kubelet在用户没有显式指定的情况下会初始化一系列kubedeps组件以保证kubelet server的正确运行。

其中有几个比较重要的kubeDeps:

kubeClient:
externalKubeClient:
eventClient:
3个和kube-apiserver通讯的clientset

mounter: 
用于进行linux主机mount操作。

dockerClient:
用于和docker daemon通讯的client

OSInterface:
用于完成操作系统调用的接口

Writer:
用于完成写文件操作的接口

CAdvisorInterface:
用于启动一个cadvisor

ContainerManager:
用于管理底层的容器管理者








### 初始化 kubelet 注入依赖函数

// cmd/kubelet/app/server.go

UnsecuredDependencies
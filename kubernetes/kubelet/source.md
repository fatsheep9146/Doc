kubelet源码解析

## kubelet源码：重要类型

type KubeletServer struct {
    KubeletFlags
    componentconfig.KubeletConfiguration
}

## kubelet源码：重要方法解析

### 启动 kubelet函数

// kubelet/app/server.go

app.run(s *options.KubeletServer, kubeDeps *kubelet.Dependencies) 

功能：启动kubelet函数

描述：

这是整个kubelet启动时所运行的函数，其中包含很多初始化工作。

其中比较重要的初始化函数包括

* UnsecuredDependencies：初始化 kubelet 注入依赖函数


### 初始化 kubelet 注入依赖函数

// cmd/kubelet/app/server.go

UnsecuredDependencies
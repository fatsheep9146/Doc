kubernetes重要library

* k8s.io/client-go
    
  用于和kubernetes集群通讯的client包。

* k8s.io/apimachinery 

  用于处理schema, typing, encoding, decoding等等操作的依赖库

## client-go

在client-go中的vendor中没有k8s.io/apimachinery和glog两个依赖的代码，因为可能会引发编译问题。

在使用client-go与kubernetes集群通讯并且创建某些k8s对象时，应该依赖于client-go库中的包，而不是采用k8s.io/api库。因为采用其他库中的代码很有可能造成无法和client-go源码共同编译成功的问题。这个也是client-go官方提出的问题。


比如如下依赖方式：
```
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/pkg/apis/extensions/v1beta1"
    "k8s.io/client-go/pkg/api"
    "k8s.io/client-go/pkg/api/v1"
```

其中client-go/kubernetes 可以用于获取和kubernetes通讯的client对象，kubernetes.ClientSet。

apis/extensions/v1beta1可以用于获取Deployment等类型的对象。

k8s.io/client-go/pkg/api/v1可以用于获取Service等类型的对象。






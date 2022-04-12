# sample controller source
## package
### main
#### 初始化klog日志的命令行参数
包括日志路径，日志级别等设置

#### 创建接收SIGTERM和SIGINT信号的channel
当收到系统的终止信号时，会向改channel中发送struct{}

#### 创建配置
获取配置的顺序
- 从参数中获取 kubeconfig 路径 和 master api 的address
- 如果都不存在，先从系统的环境变量 KUBERNETES_SERVICE_HOST 和 KUBERNETES_SERVICE_PORT 中获取链接地址，从 /var/run/secrets/kubernetes.io/serviceaccount/token 获取token，从 /var/run/secrets/kubernetes.io/serviceaccount/ca.crt 中获取证书，这些都是在安装 k8s 时设定的信息。旨在未有任何外部参数的时候，通过默认安装 k8s 时产生的信息来创建配置
- 如果存在其一，创建配置

#### 创建 k8s client
通过配置，创建 k8s client。是这一套标准的 k8s api 客户端。PS，如果不指定RateLimiter，QPS 和 Burst 会被启用

#### 创建一个基于本controller 客户端
生成一个client的客户端，这个客户端只针对于本controller

#### 创建 k8s 和 controller 的 sharedInformer 工厂函数
在后续再注册相应的需要 list&watch 的类型

#### 创建 controller
在本样例中，这个controller需要 k8s clinet，本controller的clinet，以及 k8s deployment资源的 informer cleint，还有 本cotroller资源的 informer client。在informer client中注册事件处理的回调函数，当对应的GVK发生变更，被watch，会检查对应的事件类型触发相应的方法。

##### 这个样例的controller对象大概情况：
- 将这个样例的scheme注册到系统的scheme中
- 创建一个广播员，在回调函数中会被用到
- 注册informer的事件回调函数

#### 开始两个 informer client 的 list&watch
开始两个lit&watch对象，当有停止信号的时候退出

### pkg/apis/samplecontroller
#### register.go
定义组名

### pkg/apis/samplecontroller/v1alpha1
#### types.go
- 定义了这个 controller 的对象
- 定义了这个 controller list 的对象
- 这里面会有一些注释用于自动生成深拷贝代码
- 还有些注释用于生成client的，包括clientSet类型和informer类型

#### register.go
- 定义GV，这是这册scheme所用到的对象
- 定义了一些获取 GK,GR 的方法
- 定义了将当前的scheme注册到系统scheme的方法

#### zz_generated.deepcopy.go
自动生成的深拷贝方法文件

### pkg/generated/clientset/versioned/scheme
#### register.go
自动生成的客户端
注册scheme的方法，调用了 pkg/apis/samplecontroller/v1alpha1 中定义的注册方法

### pkg/generated/clientset/versioned/typed/samplecontroller/v1alpha1
自动生成的客户度方法
这里面定义了本资源的标准增删该查方法，标准k8s api需要的方法

### pkg/generated/clientset/versioned
定义了创建clientset的方法

### pkg/generated/informers/externalversions
创建informer类型客户端的方法

### pkg/signals
信号处理方法
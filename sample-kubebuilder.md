# kubebuilder 创建的controller
使用controller-runtime package 来处理controller

## 执行初始化命令之后的状态
执行命令创建了一些内容
```
kubebuilder init --domain madongming.com --repo github.com/madongming
```
### 生成的文件及目录
#### Dockerfile
经这个controller打包成镜像的Dockerfile

#### PROJECT
记录执行kubebuilder的meta信息，在之后执行增加api，webhook等命令的时候，提供相关的meta信息

#### go.mod / go.sum
它创建golang的工作目录，相当于执行了
```
go mod init github.com/madongming
```
还安装了一些 controller 需要的一些依赖包

#### Makefile
一些构建命令，包括代码生成的命令

#### config
一些yaml文件，如crd文件，以来的其他gvr等。在部署的时候会被create到k8s集群中

#### hack
存储生成代码工具的脚本

#### main.go
代码的入口文件

### 当前源码分析
当前只有一个main.go文件
#### 全局变量
- scheme: 一个对象，主要创建一些映射，用来存放类型对GVK的相互映射
- setupLog：设置log，这里是setup相关的日志

#### init()函数
通过client-go将scheme注册到k8s api中

### 解析参数
- 设置监控地址端口
- 健康检查地址端口
- 是否启用leader election

### 应用参数
- 设置为开发模式
- 应用命令行传入的参数

### 加载配置
config 模块
- 模块初始化会设置一个命令行参数 kubeconfig ，从参数中获取
- 如果没有出入这个参数，会从环境变量 KUBECONFIG 中获取
- 如果还是没有，从环境变量 KUBERNETES_SERVICE_HOST 和 KUBERNETES_SERVICE_PORT 中获取 server 地址，使用 /var/run/secrets/kubernetes.io/serviceaccount/ca.crt中的证书文件，通过/var/run/secrets/kubernetes.io/serviceaccount/token token去创建配置
- 如果没有设置 QPS，会启用ratelimit并设置为
```
cfg.QPS = 20.0
cfg.Burst = 30.0
```

### 创建manager
这个Manager为controller-runtime模块中的接口，它初始化 Caches 和 Clients 等共享依赖，并将它们提供给 Runnables。 创建控制器需要Manager

创建过程：
- 应用参数
- 创建一个cluster对象，这个对象提供了与集群交互的方法
- 创建一个记录器，用来记录日志
- 处理leader election
- 创建监控服务的接口
- 创建探活的服务接口
- 创建一个检查是否存活的对象，里面实际上是很多其他服务的channel，用于接收其他服务是否还在运行的状态
- 构建Manager对象并返回（实际上是controllerManager 对象，它实现了Manager接口）

### 添加额外服务
- 添加健康检查的方法函数
- 添加是否ready的方法函数（同上一个，应该是为了兼容不同的应用端调用不同的检查接口）

### 启动服务
- 增加日志类型的说明
- 启动服务。这里主要是把之前的设置应用，以及启动一些子进程，等带接收相关的信号

## 添加api
```
kubebuilder create api --group webapp --version v1 --kind Sample
```

### 生成的文件及目录
执行过程中提示是否添加Resource和controller

#### api
##### package api/v1
###### groupversion_info.go
- 定义了这个资源的 GV
- 创建了注册这个资源scheme的对象，这个对象可以注册这个资源的scheme
- 创建了这个资源的scheme

###### sample_types.go
- 定义这个资源的crd描述struct
- 定义这个资源的list
- init()函数执行，调用上面的注册方法，将scheme注册到k8s api中

###### zz_generated.deepcopy.go
生成的对象深拷贝方法

#### controllers
##### package
- 定义reconcile对象
- 添加注释，让kubebuilder生成权限控制的 rbac 资源
- 定义对象的 Reconcile 方法，来轮训控制资源的动作
- 使用manger里的NewControllerManagedBy方法，创建controller，并将reconcile对象添加到这个controller中。它维护这个例子中定义的资源状态。使用的也是watch方法

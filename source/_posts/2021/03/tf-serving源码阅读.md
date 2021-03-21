---
title: tf serving源码阅读
date: 2021-03-21 13:58:06
tags: 
- tensorflow 
- tf serving
categories:
- 源码阅读
---

# TF-Serving简介
TensorFlow Serving是一个用于在生产环境中部署机器学习模型的应用系统，原生集成了TensorFlow模型，也可以扩展以应用其他类型的模型和数据。

# TF-Serving架构
![tf serving 架构](https://www.tensorflow.org/tfx/serving/images/serving_architecture.svg?hl=zh-cn)

- Servables
- Loaders
- Sources
- Managers
- Core
## Servable
Servable是TF-Serving核心的对模型的抽象，Servable的大小和粒度都很灵活，任何能提供算法或数据查询的实体都可以抽象为Servable，服务可以是任何类型和接口。Servable不负责管理自己的生命周期，而是交由Manager管理。

典型的Servables包括：
- TesnorFlow SavedModelBundle
- Embeddings查找表或词查找表

### Servable相关的数据结构
#### tensorflow.serving.ServableId
```c++
struct ServableId{
    string name;
    int64 version;
    string DebugString(){}
};
```

#### tensorflow.serving.ServableData
```c++
template<typename T>
class ServableData{
public:
    ServableData(const ServableId&, T data);
    T& DataorDie();
    T ConsumeDataorDie();
private:
    ServableData()=delete;
    const ServableId id_;
    const Status status_;
    T data_;
};
```

#### tensorflow.serving.ServableHandle
```c++
class UntypedServableHandle{
public:
    virtual const ServableId& id()const = 0;
    virtual AnyPtr servable()=0;
};

template <typename T>
class ServableHandle{
public:
    const ServableId& id() const {return untyped_handle_->id();}
    T& operator*() const {return *get();}
    T* operator->() const {return get();}
    T* get() const {return servalbe_;}
private:
    friend class Manager;
    std::unique_ptr<UntypedServableHandle> Untyped_handle_;
    T* servable_ = nullptr;
};

class SharedPtrHandle: public UntypedServableHandle{
public:
    ~SharedPtrHandle() override = default;
    explicit SharedPtrHandle(const ServableId& id, std::shared_ptr<Loader> loader)
        : id_(id), loader_(std::move(loader)) {}
    AnyPtr servable() override { return loader_->servable(); }
    const ServableId& id() const override { return id_; }

private:
    const ServableId id_;
    std::shared_ptr<Loader> loader_;
};
```
#### tensorflow.serving.ServableState
```c++
struct ServableState{
    ServableId id;
    enum class ManagerState : int {
        kStart, kLoading, kAvailable, kUnloading, kEnd,
    };
    static string ManagerStateString(ManagerState state){...}
    MangerState manager_state;
    Status health;
    string DebugString() const {...}
};
```


## Loader
Loader对Servable的生命周期进行控制，包括load/unload接口、资源预估接口等，加载后的Servable也存在Loader里面。Loader也用于扩展算法和数据后端（Tensorflow是其中一种）。当我们要添加一个新的backends时（如Pytorch等），需要为其实现一个新的Loader，以用于加载、卸载模型。

```c++
//表示一个从storage加载的一个SavedModel
class SavedModelBundleInterface{
public:
    virtual ~SavedModelBundleInterface();
    virtual Session* GetSession()=0;
    virtual const protobuf::Map<string, SignatureDef>& GetSignatures()=0;
};

struct SavedModelBundle: public SavedModelBundleInterface{
    ~SavedModelBundle();
    SavedModelBundle();
    Session* GetSession(){return session.get();}
    protobuf::Map<string, SignatureDef>& GetSignatures(){return meta_graph_def.signature_der();}

    std::unique_ptr<Session> session;
    MetaGraphDef meta_graph_def;
    std::unique_ptr<GraphDebugInfo> debug_info;
};
```

### LoaderHarness
LoaderHarness是对Loader的封装，LoaderHarness负提供Loader的状态跟踪，ServingMap和ManagedMap里面保存的都是LoaderHarness对象，只有通过LoaderHarness才能访问Loader的接口。

## Source
Source是对Servable的来源的抽象，Source监控外部资源，发现新的模型版本，并通知Target。Source为其提供的Servable的每个可用版本都提供一个Loader实例。

Source可以是：
- 文件系统，本地或者HDFS
- RPC

```c++
template<typename T>
class Source{
public:
    virtual ~Source() = default;
    using AspiredVersionsCallback = std::function<void(
      const StringPiece servable_name, std::vector<ServableData<T>> versions)>;
    // 提供要使用的AspiredVersionCallback
    virtual void SetAspiredVersionsCallback(AspiredVersionsCallback callback)=0;
};

class StaticStoragePathSource : public Source<StoragePath>{
public:
    static Status Create(const StaticStoragePathSourceConfig& config,   std::unique_ptr<StaticStoragePathSource>* result){
        auto raw_result = new StaticStoragePathSource;
        raw_result->config_ = config;
        result->reset(raw_result);
        return Status::Ok();
    }
    ~StaticStoragePathSource() override = default;
    void SetAspiredVersionsCallback(AspiredVersionsCallback callback){
        const ServableId id = {config_.servable_name(), config_.version_num()};
        LOG(INFO) << "Aspiring servable" << id;
        callback(configt_.servable_name(), {CreateServableData(id, confg_.version_path())});
    }
private:
    StaticStoragePathSource() = default;
    StaticStoragePathSourceConfig config_;
    TF_DISALLOW_COPY_AND_ASSIGN(StaticStoragePathSource);
};


class FileSystemStoragePathSource : public Source<StoragePath>{
public:
    static Status Create(const FileSystemStoragePathSourceConfig& config, std::unique_ptr<FileSystemStoragePathSource>* result);
    
    ~FileSystemStoragePathSource() override;
    Status UpdateConfig(const FileSystemStoragePathSourceConfig& config);

    void SetAspiredVersionsCallback(AspiredVersionCallback callback) override;

    FileSystemStoragePathSource config() const{
        mutex_lock l(mu_);
        return config_;
    }
private:
    friend class internal::FileSystemStoragePathSourceTestAccess;
    FileSystemStoragePathSource() = default;

    Status PollFileSystemAndInvokeCallback();

    Status UnaspireServables(const std::set<string>& servable_name) TF_EXCLUSIVE_LOCKS_REQUIRED(mu_);

    template<typename... Args>
    void CallAspiredVersionsCallback(Args&&... args){
        ...;
    }
    void SetAspiredVersionsCallbackNotifier(std::function<void()> fn) {
    mutex_lock l(mu_);
    aspired_versions_callback_notifier_ = fn;
    }

    mutable mutex mu_;
    FileSystemStoragePathSourceConfig config TF_GUARDED_BY(mu_);
    AspiredVersionsCallback aspired_versions_callback_ TF_GUARDED_BY(mu_);
    std::function<void()> aspired_versions_callback_notifier_ TF_GUARDED_BY(mu_);
    using ThreadType = absl::variant<absl::monostate, PeriodicFunction, std::unique_ptr<Thread>>;
    std::unique_ptr<ThreadType> fs_polling_thread_ TF_GUARDED_BY(mu_);
    TF_DISALLOW_COPY_AND_ASSIGN(FileSystemStoragePathSource);
}
```

### Adapter
Adapter是为了Source转成Loader而引入的抽象，这样server core的实现和具体的平台解耦，server core只需要调用LoaderHarness中的方法管理Servable（访问、加载、卸载等）。

### SourceRouter
Adapter是平台相关的，每个平台一个Adapter，这里的平台指的是TF、Pytorch等。而Source是与Servable相关的，这样在Adapter和Source之间存在一对多的关系，Router负责维护这些对应关系。（这里似乎有些问题，需要仔细看下）

## ServerCore
服务系统的创建和维护，建立HTTP REST Server、GRPC Server和模型管理部分(AspiredVersionManger)之间的关系。

## AspiredVersionManager
模型管理的上层控制部分，负责执行Source发出的模型管理指令，一部分通过回调的方式由Source调用，一部分由独立线程执行。

## BasicManager
负责Servable的管理，包括加载、卸载、状态查询、资源跟踪，对外提供如下接口：
1. ManageServable
2. LoadServable
3. UnloadServable
4. StopManagerServable

提供接口查询servableHandle(GetUntypeServableHandle)，也就是加载好的模型，供http rest或grpc server调用进行推理。

所有受管理的servable都放在ManagedMap里，已经正常加载的servable同时也放在ServingMap进行管理，提供查询接口。



## Target
Target是和Source对应的抽象概念，AspiredVersionManager、Router都是Target。

# 模型加载
`tensorflow_serving/model_servers/BUILD`中配置，可知，`tensorflow_model_server`的入口位于`tensorflow_serving/model_servers/main.cc`

## 大致流程
```c++
int main(int argc, char** argv){
    Options option;
    vector<Flag> flag_list;
    // cmd_line可以指定单个模型，多个模型需要配置config_file
    usage = Flags::Usage(argv[0], flag_list);
    // Usage 使用命令行cmdline返回用法消息，并返回标志flag_list中的用法文本字符串。
    Flags::Parse(&argc, argv, flag_list);
    // Parser 解析cmdline输入参数
    port::InitMain(argv[0], &argc, &argv);
    // InitMain 实现为空，我瞎了？？？？
    Server server;
    status = server.BuildAndStart(options);
    server.WaitForTermination();
    return 0;
}

BuildAndStart(options){
    SOME_CHECK_AND_PROCESS(options);
    ServerCore::Create(move(options), &server_core_);
    ::grpc::ServerBuilder builder;
    builder.AddListeningPort();
    builder.RegeisterService(xxx);
    //注册服务，model_service/prediction_service/profiler_service
    grpc_server = builder.BuildAndStart();
    // 启动grpc服务 
    return ;
}
Status ServerCore::Create(Options options, ServerCore* servercore){
    options.servable_state_monitor_creator;
    ServerRequestLogger::Create(nullptr, options.server_request_logger);
    aspired_version_policy = move(options.aspired_version_policy);
    server_core.reset(new ServerCore(move(options)));
    (*server_core)->Initialize(std::move(aspired_version_policy));
    return (*server_core)->ReloadConfig(model_server_config);
    // ReloadConfig 加载模型
}

Initialize(){
    
}

Status ServerCore::ReloadConfig(ModelServerConfig& new_config){
    mutex_lock l(config_mu);
}

```

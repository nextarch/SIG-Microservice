# Router（路由）

[English](./router.md) | 简体中文

---

## 服务路由

通常一个服务包含多个实例。在简单场景下，每个实例是对等的，通过负载均衡组件访问任意一个即可。但是，在分SET容灾、灰度发布、多特性环境场景下，需要把请求路由到相关特性环境的实例。
服务路由主要用于实现这类场景，根据业务需求过滤出符合需求的实例，再通过负载均衡在实例子集选出一个实例。
要完成服务路由，需要有以下三个元素

- 实例标签
- 流量标签
- 匹配规则

### 实例标签

每个实例具有逻辑属性和物理属性：

- 逻辑属性：版本、协议、业务Set、特性环境等。北极星允许用户为每个实例设置自定义标签
- 物理属性：地理位置。北极星从公司CMDB获取每个实例的地域-城市-园区信息

我们将逻辑属性以及物理属性，统一作为实例的标签属性。
实例标签属性的设置，主要有两种途径，一种是直接通过 SDK 注册实例时进行打标，另一种是直接利用 K8S 中可以对 Workload 进行设置 labels 的能力，将其同步作为实例的标签

#### SDK打标

```go
DiscoveryClient.RegisterInstance({
    Host: "",
    Port: 80,
    Namespace: "",
    Service: "",
    Labels: {
        "{LABEL_KEY}": "{LABEL_VALUE}"
    }
})
```

#### K8S打标

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      # POD 的 labels 会同步作为实例的 labels 属性
      labels:
        {LABEL_KEY}: {LABEL_VALUE}
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

### 流量标签

流量标签我们可以分为业务标签以非业务标签

- 非业务标签：请求来源地址、请求路径、协议、协议版本
- 业务标签：用户身份信息、环境信息、组织

常见的流量标签来源有这么几个地方

- http协议
  - header
  - path
  - cookie
- grpc协议
  - metadata
- 线程上下文

### 匹配规则

#### 元数据匹配

**计算方式**
通过标签（key-value）进行等值查询实例，被查询的实例的标签信息必须包含本次查询的所有标签键值对才能作为结果。
**兜底措施**
进行元数据路由时，允许配置相关的兜底措施

- 通配所有可用ip实例，等于关闭meta路由
- 匹配不带 metaData key路由
- 匹配自定义meta，需要自己填写兜底规则需要的标签信息

#### 规则匹配

**匹配符**

- 等于（EQUALS）
- 不等于（NOT_EQUALS）
- 小于（LESS_THAN）
- 大于（GREATER_THAN）
- 包含（IN）
- 不包含（NOT_IN）
- 正则（REGEX）

**计算方式**

1. 服务A调用服务B时，可以存在多条路由规则
2. 规则之间的匹配顺序，根据规则列表从上到下以此匹配

    ![企业微信截图_363601a5-8b06-4d97-96e3-34607f39679e.png](https://cdn.nlark.com/yuque/0/2022/png/333972/1659686228764-e544bc21-ca7e-4d85-9f33-8bbff333a412.png#clientId=ucb1be53b-a870-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=173&id=u0971ac41&margin=%5Bobject%20Object%5D&name=%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_363601a5-8b06-4d97-96e3-34607f39679e.png&originHeight=173&originWidth=2023&originalType=binary&ratio=1&rotation=0&showTitle=false&size=21458&status=done&style=none&taskId=ua7745d47-7263-4931-97b5-c6599f75951&title=&width=2023)

3. 根据每个请求的流量标签信息，根据规则列表顺序依次作用，选出符合本次调用请求的被调服务实例

    ![](https://cdn.nlark.com/yuque/0/2022/jpeg/333972/1659689578004-5662a2bf-4e79-4949-b3a1-d1a37cc3c079.jpeg)

**兜底措施**
由于规则的匹配是从上到下以此进行的，因此可以在服务A调用服务B的路由规则列表中，将最后一条规则配置为兜底规则。

#### 就近匹配

根据主调服务所在的地域，根据就近匹配约定优先的最小地域进行匹配，如果匹配失败，则使用更大一级的地域信息再次进行匹配，直到匹配至所设置的最大地域匹配级别。
标签前缀：`internal-location-region`、`internal-location-zone`、`internal-location-campus`
![Screen Shot 2022-08-05 at 11.01.16.png](https://cdn.nlark.com/yuque/0/2022/png/333972/1659668491825-1cc22a01-e0f4-480e-af59-817fea6bdc83.png#clientId=uadcbaf8a-2e89-4&crop=0&crop=0&crop=1&crop=1&from=drop&id=pVyzz&margin=%5Bobject%20Object%5D&name=Screen%20Shot%202022-08-05%20at%2011.01.16.png&originHeight=484&originWidth=1653&originalType=binary&ratio=1&rotation=0&showTitle=false&size=112305&status=done&style=none&taskId=u33112f05-51b3-4900-9a55-91d71757a3c&title=)

#### 实例分组

通过规则匹配之后筛选出了本次符合要求的实例，但是我们可以进一步对选中的实例在进行一个分组，并且可以通过分组之间的权重信息进行流量控制。
![Screen Shot 2022-08-05 at 15.30.54.png](https://cdn.nlark.com/yuque/0/2022/png/333972/1659684659780-0747acbe-340a-487c-ada5-d83ed42ee2fa.png#clientId=ucb1be53b-a870-4&crop=0&crop=0&crop=1&crop=1&from=drop&id=u2adc0f24&margin=%5Bobject%20Object%5D&name=Screen%20Shot%202022-08-05%20at%2015.30.54.png&originHeight=249&originWidth=883&originalType=binary&ratio=1&rotation=0&showTitle=false&size=17076&status=done&style=none&taskId=u9a4c2431-0431-4eaa-854e-dfdb1205859&title=)

### 流量染色

#### 网关染色

> 动态染色

- 所有环境都公用同一套网关集群
- 网关去实现流量染色的能力
- 在网关层会解析流量的相关信息，并根据配置的流量染色规则，为当前流量打上符合规则的相关流量标签，规则主要有以下两类
  - 条件染色：根据规则计算，得出当前流量应该打上的染色标签信息
  - 随机染色
    - 直接对流量随机染色
    - 先对流量进行规则匹配，对满足匹配规则流量再继续随机染色

![image.png](https://cdn.nlark.com/yuque/0/2022/png/333972/1660880986728-abecb816-fffb-4649-bd4a-44b45b41f3f6.png#clientId=ub7c4a5af-3374-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1158&id=u6901e424&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1158&originWidth=1166&originalType=binary&ratio=1&rotation=0&showTitle=false&size=410318&status=done&style=none&taskId=u82cf9404-5f4c-4427-bcae-c4301682f85&title=&width=1166)

#### 主调方染色

- 在部署应用实例时，直接设定了在当前环境下这个实例所有发送的请求携带的相关流量标签
  - 需要主调方在所有发送请求的地方，注入相关特定的流量标签

![image.png](https://cdn.nlark.com/yuque/0/2022/png/333972/1660879677396-6b1c1ebe-204f-4196-9277-4107752ae9f8.png#clientId=ub7c4a5af-3374-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1054&id=u95d8b337&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1054&originWidth=1088&originalType=binary&ratio=1&rotation=0&showTitle=false&size=387057&status=done&style=none&taskId=u256d2b3d-d339-4809-b79f-91b5c38e9ce&title=&width=1088)

### 标签透传 

#### 跨进程间标签传递

// TODO 附件能力

#### 进程内标签传递

我们约定所有的流量标签均已 <Key, Value> 的形式进行传递

> Java 语言生态

- 使用 ThreadLocal
  - 定义专门用于传递流量标签的 ThreadLocal
  - 定义好一系列可以支持标签自动传递的线程池
  - 用户需要在程序内进行改造，将线程的使用转为使用改线程池
- 使用 java agent 对线程进行增强
  - 定义专门用于传递流量标签的 ThreadLocal
  - 在 agent 内定义相关异步 Interface 或者 Class 的拦截
  - 针对 Runnable、Callable 进行增强，在 run、call 方法的前后，处理 ThreadLocal

> Golang 生态

- 通过标准 context.WithValue 的能力

> Python 生态

- 借助 threading.local() 能力
- 借助 contextvars.ContextVar 能力

#### 借助 opentelemetry（较为推荐，无需再做重复的事情）

- 利用 tracing 的能力，将标签信息放在名称为 io.microservice.traffic-labels 的 Baggage 中

# 场景化支持

## 多环境路由

### 场景

![image.png](https://cdn.nlark.com/yuque/0/2022/png/333972/1659683482533-e002d0a9-ab01-4fd8-936b-850bef575feb.png#clientId=uadcbaf8a-2e89-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=578&id=u57875479&margin=%5Bobject%20Object%5D&name=image.png&originHeight=578&originWidth=1280&originalType=binary&ratio=1&rotation=0&showTitle=false&size=71133&status=done&style=none&taskId=u5e0670a5-1ef6-48f4-9545-4a20ad3d5d7&title=&width=1280)
在实际的开发过程中，一个微服务架构系统下的不同微服务可能是由多个团队进行开发与维护的，每个团队只需关注所属的一个或多个微服务，而各个团队维护的微服务之间可能存在相互调用关系。如果一个团队在开发其所属的微服务，调试的时候需要验证完整的微服务调用链路。此时需要依赖其他团队的微服务，如何部署开发联调环境就会遇到以下问题：

1. 如果所有团队都使用同一套开发联调环境，那么一个团队的测试微服务实例无法正常运行时，会影响其他依赖该微服务的应用也无法正常运行。
2. 如果每个团队有单独的一套开发联调环境，那么每个团队不仅需要维护自己环境的微服务应用，还需要维护其他团队环境的自身所属微服务应用，效率大大降低。同时，每个团队都需要部署完整的一套微服务架构应用，成本也随着团队数的增加而大大上升。

此时可以使用测试环境路由的架构来帮助部署一套运维简单且成本较低开发联调环境。测试环境路由是一种基于服务路由的环境治理策略，核心是维护一个稳定的基线环境作为基础环境，测试环境仅需要部署需要变更的微服务。多测试环境有两个基础概念，如下所示：

1. 基线环境（Baseline Environment）: 完整稳定的基础环境，是作为同类型下其他环境流量通路的一个兜底可用环境，用户应该尽量保证基线环境的完整性、稳定性。
2. 测试环境（Feature Environment）: 一种临时环境，仅可能为开发/测试环境类型，测试环境不需要部署全链路完整的服务，而是仅部署本次有变更的服务，其他服务通过服务路由的方式复用基线环境服务资源。

部署完成多测试环境后，开发者可以通过一定的路由规则方式，将测试请求打到不同的测试环境，如果测试环境没有相应的微服务处理链路上的请求，那么会降级到基线环境处理。因此，开发者需要将开发新测试的微服务部署到对应的测试环境，而不需要更新或不属于开发者管理的微服务则复用基线环境的服务，完成对应测试环境的测试。

### 流量染色

流量染色即为每个请求打上目标测试环境标签，路由转发时根据请求标签匹配请求。而流量染色可以分为以下几种方式：

#### 流量标签收集

1. 从 http 的 header、url、cookie 中进行获取
2. 从 grpc 的 metadata 中进行获取
3. 从当前线程上下文中进行获取

#### 染色方式

- 静态染色
  - 静态染色所有经过当前实例的请求都带上当前实例的标签信息。例如经过 env=f1 的实例的请求都携带 env=f1 的标签信息。
- 动态染色
  - 静态染色是把服务实例的某些标签作为请求标签，服务实例标签相对静态，应用启动后初始化一次之后就不再变更。但是在实际的应用场景下，不同的请求往往需要设置不同的标签信息。此时则需要通过动态染色的能力。因此需要按照一定的规则或逻辑，为每个请求动态生成标签信息。

#### 标签透传

- **X-Metadata-Transitive-{标签 KEY} = {标签 VALUE}**
  - 默认会在整个服务链路中进行传递。
- **X-Metadata-Disposable-{标签 KEY} = {标签 VALUE}**
  - ** **默认只会从A服务传递到B服务，仅进过一跳的流量链路。

### 路由

#### 网关-> 应用

网关将流量进行染色之后需要将流量转发至第一个被调服务时，这里需要通过元数据路由的方式筛选出全部带有 env=f1 的服务实例

#### 应用->应用

服务之间的调用需要通过规格匹配的方式，计算出当前 env=f1 的主调实例，可以调用那个 env 下的被调服务实例

## 金丝雀（灰度）路由

### 场景

金丝雀发布就是用生产环境一小部分流量验证应用的一种方法。从这个名字的由来也可以看到，金丝雀发布并不是完美的，如果代码出现问题，那么背用作测试的小部分流量会出错，就跟矿坑中昏厥的金丝雀一样。这种做法在非常敏感的业务中几乎无法接受，但是当系统复杂的到一定程度，错误无法完全避免的时候，为了避免出现更大的问题，牺牲一小部分流量，就可以将大部分错误的影响控制在一定范围内。
![](https://cdn.nlark.com/yuque/0/2022/jpeg/333972/1662117899219-ad584195-b326-4430-978b-19512f52c1ff.jpeg)

### 流量染色

#### 流量标签收集

1. 从 http 的 header、url、cookie 中进行获取
2. 从 grpc 的 metadata 中进行获取

#### 标签透传

**X-Metadata-Transitive-  **开头的流量标签默认会在整个服务链路中进行传递。
**X-Metadata-Disposable-   **开头的流量标签默认只会从A服务传递到B服务，仅进过一跳的流量链路。

### 路由

对于金丝雀发布场景中涉及到的服务路由，我们可以这样子进行设置一条路由规则
![Screen Shot 2022-08-05 at 15.00.45.png](https://cdn.nlark.com/yuque/0/2022/png/333972/1659682849453-2e9fc8f8-2ced-4eb3-a1c5-d5992d020a7b.png#clientId=uadcbaf8a-2e89-4&crop=0&crop=0&crop=1&crop=1&from=drop&id=u94ca32fd&margin=%5Bobject%20Object%5D&name=Screen%20Shot%202022-08-05%20at%2015.00.45.png&originHeight=715&originWidth=872&originalType=binary&ratio=1&rotation=0&showTitle=false&size=57319&status=done&style=none&taskId=uff4781f4-9010-4e23-8855-90aa722adcd&title=)
即我们将本次进行金丝雀发布的服务实例，在一条路由规则中进行实例分组，这样流量就会同时发往这两个分组下的实例；同时通过调整实例分组各自的权重比，我们将流量的百分之 80 发往了 version = v1.0.0 的实例分组中，将流量的百分之 20 发往了 version = v2.0.0 的实例分组中。
随着金丝雀发布动作的进行，可以逐步调整 version = v1.0.0 以及 version = v2.0.0 分组

## 就近路由

### 场景

生产环境服务为了高可用、容灾等能力往往需要多机房、多城市、多地域部署。
![](https://cdn.nlark.com/yuque/0/2022/png/333972/1659606966733-a40062d0-0ef0-4a35-b560-4ccd55882d27.png#clientId=u44dc7a0c-f04d-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=udce8a573&margin=%5Bobject%20Object%5D&originHeight=1218&originWidth=1320&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uc5ab0353-8c4b-4de4-b1cf-8983fe32ba0&title=)
如上图所示，范围从小到大依次为: Campus < Zone < Region < All
其中 Campus、Zone、Region 在服务注册发现领域模型里统一定义为元数据，是一种特殊的位置元数据（Location Metadata）。
以一个实际的部署模型为例：上海机房一、上海机房二、杭州机房一、杭州机房二、北京机房。
三层模型

- Campus(上海机房一) < Zone(上海) < Region（华东）
- 三层模型有时太过于复杂，也可简化成两层模型
- Zone(上海机房一) < Region (上海)

就近路由顾名思义，服务调用时按照 同 Campus、同 Zone、同 Region 的优先级从高到低依次选取目标服务实例。核心是减少服务调用因物理距离增加的网络耗时。
跨机房容灾场景：服务端有广州云、深圳、南京云设备。客户端在深圳。 那么深圳客户端访问这个服务端时，返回实例列表由深圳、广州、南京实例组成，同时设置优先级，让深圳客户端优先访问深圳；然后是广州(同可用区)；最后是南京

### 路由

#### 主调实例地域信息

1. 在初始化治理SDK时用户进行编程指定
2. 治理SDK侧通过相关环境变量信息，自动填充当前进程的地域信息
3. 治理SDK结合云厂商的相关元数据API自动获取地域信息并填充

#### 被调实例地域信息

**SDK侧自动打标**

1. 注册实例时用户进行编程指定
2. 治理SDK通过相关环境变量信息，自动填充注册实例的地域信息
3. 治理SDK结合云厂商的相关元数据API自动获取地域信息并填充

**结合CMDB系统打标**

1. 治理中心从CMDB中将IP的地域信息同步至实例当中


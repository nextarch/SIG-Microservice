# 实例

[English](./instance.md) | 简体中文

---

## 定义

服务实例对应的是通过暴露端口的形式，提供服务接口供网络调用的节点，对应的是部署在虚拟机上的进程，或者是部署在POD中的容器进程。一般通过IP:Port的方式进行唯一标识。

## 属性

- Instance

| 名称                | 类型       | 描述             | 是否必选 |
| ------------------- | ---------- | ---------------- | -------- |
| service             | string     | 所属服务名       | 是       |
| namespace           | string     | 所属命名空间     | 是       |
| host                | string     | IP地址           | 是       |
| port                | int        | 监听端口         | 是       |
| metadata            | map        | 标签列表         | 否       |
| weight              | int        | 权重             | 否       |
| healthy             | bool       | 是否健康         | 否       |
| isolate             | bool       | 是否隔离         | 否       |
| protocol            | string     | 端口对应的协议   | 否       |
| version             | string     | 进程的版本号信息 | 否       |
| location            | Location   | 地域信息         | 否       |
| enable_health_check | bool       | 是否开启健康检查 | 否       |
| health_check        | HeathCheck | 健康检查相关配置 | 否       |

- Location

| 名称       | 类型   | 描述          | 是否必选 |
| ---------- | ------ | ------------- | -------- |
| region     | string | 地域-大区     | 否       |
| zone       | string | 地域-城市     | 否       |
| datacenter | string | 地域-数据中心 | 否       |



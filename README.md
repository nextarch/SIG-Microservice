# SIG-Microservice

English | [简体中文](./README-zh.md) 

---

README：

- [Introduction](#introduction)
- [Specification](#specification)
- [Ecosystem](#ecosystem)
- [SIG Members](#sig-members)
- [SIG Responsibilities](#sig-responsibilities)

## Introduction

SIG-Microservice is committed to providing a standardized service discovery and governance solution for distributed and microservice architecture. It applies to heterogeneous infrastructure such as multi-language, multi-framework, different runtime environments and hybrid cloud scenario.

## Specification

### Service Governance Specification

|                                   |         Latest Release             |  
| :-------------------------------- | :--------------------------------: |
| **Serivce Specification**         |
| namespace       | [v1alpha1](/specification/namespace/namespace.md) |
| service         | [v1alpha1](/specification/service/service.md) |
| instance        | [v1alpha1](/specification/instance/instance.md) |
| health check    | [v1alpha1](/specification/healthcheck/healthcheck.md) |
| **Fault Tolerance Specification** |
| circuit breaker | [v1alpha1](/specification/circuitbreaker/circuitbreaker.md) |
| fault detector  | [v1alpha1](/specification/faultdetector/faultdetector.md) |
| **Traffic Control Specification** |
| router          | [v1alpha1](/specification/router/router.md) |
| load balancer   | [v1alpha1](/specification/loadbalancer/loadbalancer.md) |
| **Access Control Specification**  |
| limiter         | [v1alpha1](/specification/limiter/limiter.md) |
| authentication  | [v1alpha1](/specification/authentication/authentication.md) |

## Ecosystem

## SIG Members

| Name          | Organization         |
| ------------- | -------------------- |
| Hongzhi Wang  | Tencent   |
| Shibai Wei    | Kuaishou  |
| Xiang Yu      | Bluefocus |
| Ke Su         | TAL       |
| Bo Zhang      | BIGO      |
| Jiajun Shan   | [Polarismesh](https://github.com/polarismesh) |
| Junfeng Wan   | [go-zero](https://github.com/zeromicro)   |
| Qiang Guo     | [GoFrame](https://github.com/gogf)        |
| Guangming Luo | [CloudWeGo](https://github.com/cloudwego) |
| Le Zhang      | [Spring Cloud Tencent](https://github.com/Tencent/spring-cloud-tencent) |
| Kaiyuan Li    | [TARS](https://github.com/TarsCloud/Tars) |

## SIG Responsibilities

**End User Input Gathering (Inbound Communication)**

- Gather useful end user input and feedback regarding expectations, pain points, primary use cases etc.
- Compile this into easily consumable reports and/or presentations to assist projects with feature design, prioritization, UX etc.

**End User Education (Outbound Communication)**

Provide up-to-date, high quality, unbiased and easy-to-consume material to help end users to understand and effectively adopt microservice technologies and practises within the SIG’s area, for example: White papers, presentations, videos, or other forms of training clarifying terminology, comparisons of different approaches, available projects or products, common or recommended practises, trends, illustrative successes and failures, etc.

**As Trusted Expert Advisors to the TOC**

- Perform technical due diligence on new and graduating projects, and advise TOC on findings.
- Be involved with, or periodically check in with projects in their area, and advise TOC on health, status and proposed actions (if any) as necessary or on request.

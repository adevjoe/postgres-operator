<h1>概念</h1>

Postgres [operator](https://coreos.com/blog/introducing-operators.html)
用于在 Kubernetes (K8s) 上管理 PostgreSQL 集群:

1. 该 operator 监听 PostgreSQL 集群清单文件的创建、更新、删除并更新运行中的集群。例如, 当用户提交了一个新的清单文件，operator 获取到清单文件并创建一个新的 Postgres 集群和一些 K8s StatefulSets、Postgres 角色等必要的资源实体。查看这些
   [Postgres 清单文件](../manifests/complete-postgres-manifest.yaml) 可以了解涵盖的配置。

2. 该 operator 也监听 [它自身配置文件](../manifests/configmap.yaml) 的更新并且在必要的情况下更新运行中的集群。对于实例，如果 pod 中的镜像修改了，operator 会执行滚动更新，这将意味着它会一个一个地重新生成 StatefulSet 中的 pod，并使用新的镜像。

3. 最终，该 operator 会周期性地同步每个 Postgres 集群的状态，把真实状态同步至清单文件中定义的期望状态。

4. 该 operator 旨在让用户无需操作即可管理集群，且仅仅通过清单文件进行配置相关工作。这样就可以轻松集成到自动部署流水线中，而无需直接访问 K8s。

## 范围

Postgres Operator 的职责在于维护, 更改配置和清理 Postgres 集群则使用 Patroni，
operator 主要让 Patroni 运行在 K8s 集群上变得更简单、方便。operator 一方面维护和
修改 K8s 资源，另一方面在数据库集群启动后或运行中，也维护数据库和数据库角色。我们尝试
将尽可能多的工作留给适合的 K8s 和 Patroni，尤其是集群启动和高可用。但是，operator 
会参与一些总体流程，例如滚动更新以改善用户体验。

当前，监控或调整 Postgres 不在 operator 的范围内。然而，通过全局 sidecars 配置，
我们提供足够的灵活性去集成一些工具，比如 [ZMON](https://opensource.zalando.com/zmon/),
[Prometheus](https://prometheus.io/) 或其它 Postgres 工具。


## 实体关系概览

这幅图总结了在提交新的 Postgres 集群 CRD 时 operator 将创建的内容:

![postgresql-operator](diagrams/operator.png "由 operator 创建的 K8s 资源")

这幅图没有完全地展示单个集群 pod 中组件情况，让我们放大图片看看:

![pod](diagrams/pod.png "数据库 pod 中的组件")

这两幅图可以帮助你了解 operator 提供的主要功能。

## 状态

该项目仍然处于开发阶段。但是 [Zalando 内部]((https://jobs.zalando.com/tech/blog/postgresql-in-a-time-of-kubernetes/)) 已经在
使用了，用来在 K8s 上运行大量的 Postgres 集群测试环境，生产环境也用的越来越多。
在此环境中，operator 被部署到多个 K8s 集群，用户可以在其中通过我们的 CI/CD 
基础架构，或依靠用户界面来创建清单。

请反馈任何任何遇到的问题到 https://github.com/zalando/postgres-operator/issues.

## 演讲

- "PostgreSQL on K8S at Zalando: Two years in production" talk by Alexander Kukushkin, FOSSDEM 2020: [video](https://fosdem.org/2020/schedule/event/postgresql_postgresql_on_k8s_at_zalando_two_years_in_production/) | [slides](https://fosdem.org/2020/schedule/event/postgresql_postgresql_on_k8s_at_zalando_two_years_in_production/attachments/slides/3883/export/events/attachments/postgresql_postgresql_on_k8s_at_zalando_two_years_in_production/slides/3883/PostgreSQL_on_K8s_at_Zalando_Two_years_in_production.pdf)

- "Postgres as a Service at Zalando" talk by Jan Mußler, DevOpsDays Poznań 2019: [video](https://www.youtube.com/watch?v=FiWS5m72XI8)

- "Building your own PostgreSQL-as-a-Service on Kubernetes" talk by Alexander Kukushkin, KubeCon NA 2018: [video](https://www.youtube.com/watch?v=G8MnpkbhClc) | [slides](https://static.sched.com/hosted_files/kccna18/1d/Building%20your%20own%20PostgreSQL-as-a-Service%20on%20Kubernetes.pdf)

- "PostgreSQL and Kubernetes: DBaaS without a vendor-lock" talk by Oleksii Kliukin, PostgreSQL Sessions 2018: [video](https://www.youtube.com/watch?v=q26U2rQcqMw) | [slides](https://speakerdeck.com/alexeyklyukin/postgresql-and-kubernetes-dbaas-without-a-vendor-lock)

- "PostgreSQL High Availability on Kubernetes with Patroni" talk by Oleksii Kliukin, Atmosphere 2018: [video](https://www.youtube.com/watch?v=cFlwQOPPkeg) | [slides](https://speakerdeck.com/alexeyklyukin/postgresql-high-availability-on-kubernetes-with-patroni)

- "Blue elephant on-demand: Postgres + Kubernetes" talk by Oleksii Kliukin and Jan Mussler, FOSDEM 2018: [video](https://fosdem.org/2018/schedule/event/blue_elephant_on_demand_postgres_kubernetes/) | [slides (pdf)](https://www.postgresql.eu/events/fosdem2018/sessions/session/1735/slides/59/FOSDEM%202018_%20Blue_Elephant_On_Demand.pdf)

- "Kube-Native Postgres" talk by Josh Berkus, KubeCon 2017: [video](https://www.youtube.com/watch?v=Zn1vd7sQ_bc)

## 文章

- "How to set up continuous backups and monitoring" by Pål Kristensen on [GitHub](https://github.com/zalando/postgres-operator/issues/858#issuecomment-608136253), Mar. 2020.

- "Postgres on Kubernetes with the Zalando operator" by Vito Botta on [has_many :code](https://vitobotta.com/2020/02/05/postgres-kubernetes-zalando-operator/), Feb. 2020.

- "Running PostgreSQL in Google Kubernetes Engine" by Kenneth Rørvik on [Repill Linpro](https://www.redpill-linpro.com/techblog/2019/09/28/postgres-in-kubernetes.html), Sep. 2019.

- "Zalando Postgres Operator: One Year Later" by Sergey Dudoladov on [Open Source Zalando](https://opensource.zalando.com/blog/2018/11/postgres-operator/), Nov. 2018

## Tekton

### 概念

Tekton 是一个功能强大且灵活的Kubernetes 原生开源框架，用于创建持续集成和交付（CI/CD）系统。

Tekton定义了Task, TaskRun, Pipeline, PipelineRun, PipelineResource 五类核心对象。Tekton通过对Task和Pipeline的抽象，我们可以定义出任意组合的pipeline模板来完成各种各样的CICD任务。通过TaskRun,PipelineRun,PipelineResource可以将这些模板套用到各个实际的项目中。

#### 概念模型

- Step是CI/CD工作流中的操作，例如为Python web应用程序运行一些单元测试，或编译Java程序。Tekton使用您提供的基础镜像执行每个步骤
- Task是按顺序排列的Step集合。Tekton以Kubernetes Pod的形式执行Task，其中每个Step都成为Pod中的运行的容器。这种设计允许Task中的所有Step享用相同环境。例如，在Task中挂载一个Kubernetes卷，该卷可以在Task中的每个Step中访问
- Pineline是按顺序排列的Task集合。Tekton收集所有任务，将它们连接在一个有向无环图形(DAG)中，然后按顺序执行该图形的任务。换句话说，它创建了多个Pod对象，并确保每个Pod按成功的按照预期运行。Tekton允许开发人员完全控制该过程：用户可以设置任务完成的fan-in(扇入)/fan-out(扇出)场景，要求Tekton在存在flaky测试时自动重试，或者指定Task在继续之前必须满足设定的条件
- 资源输入输出 ：每个 Task 和 Pinelines 可能都有自己的输入和输出，在Tekton中称为输入和输出资源。例如，编译任务可能有一个 git 库作为输入，一个镜像作为输出。
- 





### 原理

Tekton利用Kubernetes的List-Watch机制，在启动时初始化了2个Controller， PipelineRunController和TaskRunController。

PipelineRunController监听PipelineRun对象的变化。在它的reconcile逻辑中，将pipeline中所有的Task构建为一张有向无环图(DAG)，通过遍历DAG找到当前可被调度的Task节点创建对应的TaskRun对象。

TaskRunController监听TaskRun对象的变化。在它的reconcile逻辑中将TaskRun和对应Task转化为可执行的Pod，由kubernetes调度执行。利用Kubernetes的OwnerReference机制，pipelinerun own taskrun, taskrun own pod。pod状态变更时触发taskrun的reconcile逻辑，taskrun状态变更时触发pipelinerun的reconcile逻辑。



### Task

workspace : `Task`在执行期间需要的一个或多个卷 ; 主要是工作目录（放置git拉取代码）
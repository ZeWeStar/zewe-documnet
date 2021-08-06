# Flowable 

### 概念

BPMN

Flowable

资料

### 部署

包结构概述





### Flowable API

#### 流程定义

流程模型部署

+ xxxxbpmn20.xml 部署

  ```
  DeploymentBuilder deploymentBuilder = repositoryService
                  .createDeployment()
                  .addClasspathResource("xxxxbpmn20.xml");
  Deployment deployment = deploymentBuilder.deploy();                
  ```

+ 多个流程定义文件 （xml打包zip/bar）

  ```
  Deployment deployment = 
  processEngine.getRepositoryService()//获取流程定义和部署对象相关的Service  
    .createDeployment()//创建部署对象 
    //使用zip方式部署，将approve.bpmn和approve.png压缩成zip格式的文件  
    .addZipInputStream(zipInputStream)
    .deploy();//完成部署
  ```

+ model 方式部署 - 通过act_de_model表进行部署

  ```
  org.flowable.ui.modeler.domain.Model modelData =modelService.getModel(modelId);
  byte[] bpmnBytes = new BpmnXMLConverter().convertToXML(model);
  String processName = modelData.getName() + ".bpmn20.xml";
  Deployment  deploy=   repositoryService.createDeployment()
                  .name(modelData.getName())
                  .addString(processName, new String(bpmnBytes, "UTF-8"))
                  .deploy();
  ```

+ 其他方式部署

流程定义相关表

| 数据表            | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| ACT_RE_DEPLOYMENT | 流程模型部署对象表，每部署一次生成一条，id作为act_re_procdef和act_ge_bytearray 外键 |
| ACT_RE_PROCDEF    | 流程定义表，act_re_deployment只有一条部署信息，但act_re_procdef有多个记录（一个流程定义对应一条，zip包部署），同时act_ge_bytearray也是每一个流程定义对应2条记录。这个表有DEPLOYMENT_ID_外键字段，用它关联act_re_deployment。 |
| ACT_GE_BTYEARRAY  | 流程模型资源文件的真正存放地方，它每部署一次就会产生2条记录，一条是关于bpmn规范的文件内容存放在BYTES字段中，另一条是图片信息，采用二进制格式存储。 |
| ACT_RE_MODEL      |                                                              |
|                   |                                                              |





#### 流程实例

+ 启动流程实例

  ```
  // 部署流程定义
  Deployment deployment = repositoryService.createDeploymentQuery().processDefinitionKey(processKey).singleResult();
  
  // 启动流程实例
  ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(processKey, busiKey, values);
  ```

+ 终止流程实例

  ```
  runtimeService.deleteProcessInstance(processInstanceId,"删除原因");
  ```

+ 我发起的实例

  ```
  List<HistoricProcessInstance> list=processEngine.getHistoryService() // 历史相关Service
                                      .createHistoricProcessInstanceQuery()
                                      // 已完成的 unfinish 未完成的，或者不加表示全部
                                      .finished() 
                                      .startedBy(userId) // 流程实例发起人
                                      .orderByProcessInstanceStartTime().asc()
                                      .list();
  ```

+ 我参与的流程实例

  ```
  // 用户参与的流程实例信息 （历史，用户参与）
  List hpis = historyService
  .createHistoricProcessInstanceQuery().involvedUser(name)
  .orderByProcessInstanceStartTime().desc().listPage(firstResult, maxResults);
  ```

+ 挂起与激活流程实例

  ```
  // 挂起
  runtimeService.suspendProcessInstanceById(processInstanceId);
  // 激活
  runtimeService.activateProcessInstanceById(processInstanceId);
  ```

+ 获取定义流程图

  ```
   ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery()
                  .processDefinitionId(processVo.getProcessDefinitionId())
                  .list()
                  .get(0);
          InputStream imageStream = repositoryService.getProcessDiagram(processDefinition.getId());
  ```

+ 获取动态流程图

  ```
  // 将生成图片放到文件夹下
  String deploymentId = "801";
  // 获取图片资源的名称
  List<String > list = processEngine.getRepositoryService().getDeploymentResourceNames(deploymentId);
  // 定义图片资源的名称
  String resourceName = "";
  if (list !=null && list.size()>0){
      for (String name:list){
           if (name.indexOf(".png")>=0){
                resourceName = name;
           }
      }
  }
   
  // 获取图片的输入流
  InputStream  in = processEngine.getRepositoryService().getResourceAsStream(deploymentId,resourceName);
  ```

+ 运行时实例 或 历史时实例 （互斥）

  ```
  // 运行时实例
  ProcessInstance  pi= runtimeService.createProcessInstanceQuery().processInstanceId(procInstanceId).singleResult();
  
  // 历史时实例
  HistoricProcessInstance  hpi= historyService.createHistoricProcessInstanceQuery().processInstanceId(procInstanceId).singleResult();
  ```

+ 11

#### 任务实例


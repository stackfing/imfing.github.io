---
title: Activiti 入门小案例
date: 2017-11-14 19:12:18
tags:
---

# Getting Started

## spring boot 整合 activiti

## Step 1
添加 Maven 依赖(可能需要 Mysql 驱动)
```
<!-- Activiti -->
<activiti>6.0.0</activiti>
<dependency>
			<groupId>org.activiti</groupId>
			<artifactId>activiti-spring-boot-starter-basic</artifactId>
			<version>${activiti}</version>
</dependency>

<!-- mysql驱动 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>

```

## Step 2

在 `resources` 文件夹新增一个目录 `process`

新建文件 `leave.bpmn` 

![](/images/activiti/diagram.png)

流程的 Id 为：leave

提交任务指定为 zs 处理，所以我们在 Assignee 填写: `zs`

同样的，审批任务指定为 ls 处理， Assignee 填写: `ls`

可以查看 xml 文件，生成出来的 xml 代码如下：

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" xmlns:tns="http://www.activiti.org/test" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" expressionLanguage="http://www.w3.org/1999/XPath" id="m1510372276269" name="" targetNamespace="http://www.activiti.org/test" typeLanguage="http://www.w3.org/2001/XMLSchema">
  <process id="leave" isClosed="false" isExecutable="true" processType="None">
    <startEvent id="_2" name="StartEvent"/>
    <userTask activiti:assignee="zs" activiti:exclusive="true" id="_3" name="提交"/>
    <userTask activiti:assignee="ls" activiti:exclusive="true" id="_4" name="批准"/>
    <endEvent id="_5" name="EndEvent"/>
    <sequenceFlow id="_6" sourceRef="_2" targetRef="_3"/>
    <sequenceFlow id="_7" sourceRef="_3" targetRef="_4"/>
    <sequenceFlow id="_8" sourceRef="_4" targetRef="_5"/>
  </process>
  <bpmndi:BPMNDiagram documentation="background=#3C3F41;count=1;horizontalcount=1;orientation=0;width=842.4;height=1195.2;imageableWidth=832.4;imageableHeight=1185.2;imageableX=5.0;imageableY=5.0" id="Diagram-_1" name="New Diagram">
    <bpmndi:BPMNPlane bpmnElement="leave">
      <bpmndi:BPMNShape bpmnElement="_2" id="Shape-_2">
        <omgdc:Bounds height="32.0" width="32.0" x="420.0" y="60.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="32.0" width="32.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_3" id="Shape-_3">
        <omgdc:Bounds height="55.0" width="85.0" x="390.0" y="175.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_4" id="Shape-_4">
        <omgdc:Bounds height="55.0" width="85.0" x="390.0" y="290.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_5" id="Shape-_5">
        <omgdc:Bounds height="32.0" width="32.0" x="420.0" y="425.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="32.0" width="32.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="_6" id="BPMNEdge__6" sourceElement="_2" targetElement="_3">
        <omgdi:waypoint x="436.0" y="92.0"/>
        <omgdi:waypoint x="436.0" y="175.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_7" id="BPMNEdge__7" sourceElement="_3" targetElement="_4">
        <omgdi:waypoint x="432.5" y="230.0"/>
        <omgdi:waypoint x="432.5" y="290.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_8" id="BPMNEdge__8" sourceElement="_4" targetElement="_5">
        <omgdi:waypoint x="436.0" y="345.0"/>
        <omgdi:waypoint x="436.0" y="425.0"/>
        <bpmndi:BPMNLabel>
          <omgdc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>

```

## Step 3 

配置在 `application.yml` 文件配数据源

```
server:
  port: 8989
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/activiti?autoReconnect=true&useUnicode=true&characterEncoding=UTF-8
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL5Dialect
        format_sql: true
        use_sql_comments: true
```

## Step 3

新建一个 ActivitiTest.java 测试类

第一步：将流程部署

```
@Test
	public void createProcessInstance() {
		repositoryService.createDeployment().addClasspathResource("processes/leave.bpmn").name("请假手续");
	}
```

第二步：启动一个流程实例

```
@Test
	public void startProcessInstance() {
		runtimeService.startProcessInstanceByKey("leave");
	}
```

第三步：查询任务列表
```
@Test
	public void getTasksByName() {
		System.out.println(taskService.createTaskQuery().taskCandidateOrAssigned("zs").list());
	}
```

此时就能显示出 zs 需要处理的列表，而查询 ls 却没有任何任务

第四步：完成任务
```
@Test
	public void completeTask() {
		taskService.complete("5005");
	}
```

第五步：查询 ls 任务列表
```
@Test
	public void getTasksByName() {
		System.out.println(taskService.createTaskQuery().taskCandidateOrAssigned("ls").list());
	}
```

这个时候 zs 的任务已经完成了，ls 有了一个新的任务需要处理
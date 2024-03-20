<center>
    <font size="120%">
        <b>Flowable</b>
    </font>
</center>

# 简介

​		Flowable是一个使用Java编写的轻量级业务流程引擎，可以帮助我们简化类似**请假审批**等工作流业务的开发工作。Flowable流程引擎可用于部署BPMN 2.0流程定义。主要开发步骤有分为两个阶段，一个是流程图的绘制与部署，另一个是实际业务开发的API集成。

# 流程图的绘制

​		BPMN 2.0是一个基于XML格式被广泛接受与支持的，展现流程的流程定义规范，我们也是基于这个规范来使用XML文件描述流程图的结构，大致的文件结构如下：

~~~xml
<definitions
  xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
  xmlns:flowable="http://flowable.org/bpmn"
  targetNamespace="Examples">

  <process id="myProcess" name="My First Process">
    ..
  </process>

</definitions>
~~~

​		不过使用XML文件来绘制流程还是比较麻烦的，所以这里就不展开描述了，改为使用其他工具以图形化界面方式来绘制，最终生成的文件也是BPMN 2.0规范的xml文件，所以不用担心兼容性。

​		绘制工具也有很多，IDEA、Eclipse和VsCode都有相应的插件，这里我们以官方的Flowable UI来演示，当然，工具本身并不重要，重要的是流程绘制过程中各个节点的定义和使用。

## Flowable UI部署

1. 资源下载：https://github.com/flowable/flowable-engine/releases/download/flowable-6.6.0/flowable-6.6.0.zip

2. 将压缩包中的 `flowable-6.6.0\wars\flowable-ui.war` 丢到Tomcat中跑起来

3. 打开`http://localhost:8080/flowable-ui` 用账户：admin/test 登录

   ![img](img\FL01.png)

4. 进入进入APP.MODELER创建流程，之后可以导出流程到项目中使用，两种导出方式

   * 第一种是导出为BPMN规范的XML文件，然后在项目去自行部署

   * 第二种是直接修改当前UI项目的配置文件`apache-tomcat-9.0.37\webapps\flowable-ui\WEB-INF\classes\flowable-default.properties`，将流程直接部署到指定数据库

     ![img](img\FL02.png)

     > 注：流程图的部署其实最终都是要存在数据库中的，方式自行选择

## 流程图绘制

​		![img](img\FL03.png)

根据业务需要在 flowable-ui>APP.MODELER里面绘制流程图，示例如上图。先解释一些概念。

- **事件（event）** 通常用于为流程生命周期中发生的事情建模，图里是【开始、结束】两个圈。
- **顺序流（sequence flow）** 是流程中两个元素间的连接器。图里是【箭头线段】。
- **网关（gateway）** 用于控制执行的流向。图里是【菱形（中间有X）】
- **用户任务（user task）** 用于对需要人工执行的任务进行建模。图里是【矩形】。

简单的工作流大概就这些元素(还有很多这里就不扩展了)。下面描述一下工作流是如何流动的。

首先启动了工作流后，由【开始】节点自动流向【学生】节点，等待该任务执行。任务被分配的学生用户执行后流向 【老师】节点，再次等待该任务执行。被分配的老师用户执行后流向 【网关】，网关以此检查每个出口，流向符合条件的任务，比如这里老师执行任务时是同意，就流向【校长】节点，等待该任务执行。执行后跟老师类似，同意后就流向【结束】节点，整个流程到此结束。
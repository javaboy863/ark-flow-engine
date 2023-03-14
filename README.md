# 1.什么是ark-flow-engine？
&emsp;&emsp;ark-flow-engine是ark系列框架中的流程编排引擎组件。
# 2.ark-flow-engine解决了什么问题？
&emsp;&emsp;ark-flow-engine 用于中台等复杂的业务场景，抽象分解能力后的对能力的编排。
# 3.ark-flow-engine使用场景
- 交易中台，用于编排不同接入中台的业务线，不同交易流程的编排场景；
- 商品中台，不同业务线建品流程的编排场景。
# 4.ark-flow-engine服务如何接入？
1、引入jar依赖
```
<dependency>
    <groupId>com.ark.flow</groupId>
    <artifactId>ark-flow-engine</artifactId>
    <version>1.0</version>
</dependency>
```
2、通过xml方式配置
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:flow="http://code.ark.com/schema/flow"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
         http://code.missfresh.com/schema/flow
         http://code.missfresh.com/schema/flow/flow.xsd">
    <!-- spring bean对象 -->
    <bean id="action1" class="com.ark.flow.demo.action.SimpleAction1" />
    <bean id="action2" class="com.ark.flow.demo.action.SimpleAction2" />
    <bean id="action3" class="com.ark.flow.demo.action.SimpleAction3" />
    <!-- flow对象 -->
    <flow:flow id="flowTest">
        <flow:nodes>
            <flow:node name="node1" action-ref="action1" />
            <flow:node name="node2" action-ref="action2" />
            <flow:node name="node3" action-ref="action3" />
        </flow:nodes>
        <flow:relations>
            <flow:relation pre-node="start" next-node="node1" call-type="syn"></flow:relation>
            <flow:relation pre-node="node1" next-node="node2" call-type="syn"></flow:relation>
            <flow:relation pre-node="node2" next-node="node3" call-type="syn"></flow:relation>
            <flow:relation pre-node="node3" next-node="end" call-type="syn"></flow:relation>
        </flow:relations>
    </flow:flow>
</beans>
```
3、注入流程编排对象并开始流程
```
@Resource(name = "flowTest")
Flow flow;
@Resource
FlowFactory flowFactory;

Flow<SimpleFlowRequest,SimpleFlowResponse> flow = flowFactory.getFlowBean("flowTest");
SimpleFlowRequest flowRequest = SimpleFlowRequest.builder()
        .add("field1",123)
        .add("field2","abc")
        .build();
SimpleFlowResponse simpleFlowResponse = flow.start(flowRequest);
```
# 5.最终一致性说明
- 1.流程开始顺序执行各个节点逻辑，当发生异常时，同步顺序执行回滚操作。
- 2.通常是这样，但当回滚又发生异常时，发出回滚失败的MQ消息，程序抛出异常。
- 3.流程监控，消费MQ消息，再次向同服务器应用请求，异步请求回滚。
再次发生失败？<br/>
总共尝试回滚20次调用<br/>
第一次，立即调用回滚<br/>
第二次～第五次，间隔30秒调用一次<br/>
第六次～第二十次，间隔5分钟调用一次<br/>

# DDD



https://tech.meituan.com/2017/12/22/ddd-in-practice.html



个人理解



**将对象与行为绑定在一起**

理解成go中的方法？

MVC中，主要的业务逻辑都在Service中，对象大部分只有get set equals等非业务方法。

![image-20240715211950753](C:\Users\chenz\OneDrive\桌面\note\else\images\image-20240715211950753.png)

在MVC架构中，简单的CRUD方法使用是没什么问题，但是业务复杂性上来了，并且牵扯到不同的业务逻辑状态。代码意图就会变得散乱。



**抽象、分治、知识**

介绍一下概念：

**分治** 把问题空间分割为规模更小且易于处理的若干子问题。分割后的问题需要足够小，以便一个人单枪匹马就能够解决他们；其次，必须考虑如何将分割后的各个部分装配为整体。分割得越合理越易于理解，在装配成整体时，所需跟踪的细节也就越少。即更容易设计各部分的协作方式。评判什么是分治得好，即高内聚低耦合。

**抽象** 使用抽象能够精简问题空间，而且问题越小越容易理解。举个例子，从北京到上海出差，可以先理解为使用交通工具前往，但不需要一开始就想清楚到底是高铁还是飞机，以及乘坐他们需要注意什么。

**知识** 顾名思义，DDD可以认为是知识的一种。

**限界上下文** 与微服务中同名概念类似，强调中业务角度上将问题以大化小





以业务架构为着眼点  而非系统架构或技术架构





将其分为？

**实体**：与传统意义上实体差别不大，由标识区分

**值对象**：在不含唯一标识情况下 描述对象

**聚合根（根实体）**：一组相关对象的集合，作为一个整体被外界访问，聚合根（Aggregate Root）是这个聚合的根节点。 **也许可理解为一个存储多对象的数据结构？**



**聚合**由根实体，值对象，实体组成



**领域服务**：可以类比成服务向外暴露出的api。对于微服务架构风格，一切领域逻辑的对外暴露均需要通过领域服务来进行。如原本由聚合根暴露的业务逻辑也需要依托于领域服务。



**领域事件**：对领域内发生的活动进行的建模。（事件的建模？）



**领域对象**：根据业务划分的，同一限界上下文内，界定不同操作行为的对象个体。**是同一界限上下文内实体聚合值对象的父集**





# DDD工程实现

类似微服务中的，**以模块作为单领域限界上下文的单位**

以下是不同包之间相互导入的组织结构

```java
import com.company.team.bussiness.lottery.*;//抽奖上下文
import com.company.team.bussiness.riskcontrol.*;//风控上下文
import com.company.team.bussiness.counter.*;//计数上下文
import com.company.team.bussiness.condition.*;//活动准入上下文
import com.company.team.bussiness.stock.*;//库存上下文
```



模块内按照领域对象、领域服务、领域资源库、防腐层等方式定义

```java
import com.company.team.bussiness.lottery.domain.valobj.*;//领域对象-值对象
import com.company.team.bussiness.lottery.domain.entity.*;//领域对象-实体
import com.company.team.bussiness.lottery.domain.aggregate.*;//领域对象-聚合根
import com.company.team.bussiness.lottery.service.*;//领域服务
import com.company.team.bussiness.lottery.repo.*;//领域资源库
import com.company.team.bussiness.lottery.facade.*;//领域防腐层
```







防腐层太屌了

### 防腐层（Anti-Corruption Layer, ACL）

- **作用范围**：防腐层主要用于**领域间**，也就是不同的限界上下文（Bounded Context）之间或系统之间。
- **目的**：保护本地领域模型免受外部系统或不同限界上下文的变化和复杂性的影响。防腐层确保外部系统的变化不会直接影响到本地系统，保持本地领域模型的纯粹性和一致性。
- **实现**：通过数据转换、协议适配和接口封装，将外部系统的数据和操作转换为本地系统可以理解和处理的形式。

### 数据流转（Data Flow）

- **作用范围**：数据流转主要用于**领域内**，即在同一个限界上下文内的数据处理过程。
- **目的**：描述数据在系统内部各个组件和服务之间的流动和处理过程，确保数据能够在系统内有效传输、转换和存



粗略可理解为一个是领域间  一个是领域内

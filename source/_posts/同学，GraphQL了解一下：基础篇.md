---
title: 同学，GraphQL了解一下：基础篇
date: 2018-07-07 22:04:34
tags:
    - GraphQL
---
### 简介 ###
GraphQL由Facebook发起，其手机客户端自2012年起，就全面采用了GraphQL查询语言， 2015年， Facebook全面开源了第一份GraphQL规范。到目前为止，在Twitter，Github，Pinterest，Shopify等大型网站也得到了广泛的实践。语言也从最初的js，扩展到java ,python，Go。且Apollo-client也逐渐做全了graphql生态。
GraphQL是一种用于 API 的查询语言（规范），是一种协议而非存储。GraphQL对你的API中的数据提供了一套易于理解的完整描述，使得客户端能够准确地获得它需要的数据，而且没有任何冗余；
如果想在读下面的知识之前，对GraphQL先有一个粗狂的认识，可以登录[GitHub GraphQL API][1] 站点，体会一下其使用，可点击右侧的Doc文档查看接口定义。  

![clipboard.png][3]


### GraphQL出现的意义 ###
![clipboard.png][4]
上面是一张我们常用的Restful接口交互和引入GraphQL接口中间层的接口交互对比图，在此之前，我们先了解一下当前我们所使用的Restful接口的痛点：
 1. 冗余字段大量返回;
 2. 渲染一个复杂的页面需要发起多个请求;
 3. API定义文档更新不及时；  
相信上面的三点，在前端淌过水的你，一定是会有体会的。一个接口，同时用于h5页面和web页面的渲染源数据，但可能H5页面只需要这些数据源的1/10,冗余数据的返回浪费了流量，消耗了时间；由于现在后端更推崇microservice,提供聚合服务的API被认为没有技术含量，所以我们有时渲染一个关系复杂的页面，需要请求5,6次，才能完整的渲染出一个页面，这期间我们还得去容错，避免因其中某一个请求的失败导致页面无法渲染的问题；API文档更新不及时，相信这是最常见的吐槽，好好的静态页面写完，并自己照着API文档做完mock，等联调时，接口返回的字段完全不按API定义来，奔溃。
![clipboard.png][5]
上图摘自GraphQL官网，其用简单的三句话：描述你的数据（服务端），请求你所要的数据（客户端），得到可预测的结果。其隐藏的意思就是：
 1. 服务端（或则中间层）需要描述所有可能的类型系统（Schema），有一定工作量；
 2. 按你所需请求你需要的数据，这就解决了不同客户端，不同的渲染需求，都请求回同样的数据源的痛点；
 3. 获取多个资源只用一个请求，除了你可以描述你某个请求需要返回那些字段，你还可以定义这个请求需要返回那些数据。对应浏览器来讲，只发送了一个请求，GraphQL服务端会根据请求定义将这些请求拆分，并最后再将数据拼装返回回来；
听起来是不是特别爽，那实现起来呢？
### 零碎的GraphQL语法 ###
#### 基础语法 ####
其实GraphQL所需要学习的语法很少，大部分语法与我们平时的语法一致，可以通过[官网][2]详细了解。首先，GraphQL是一门强类型语言，所以和我们在数据库定义一张表一样，我们需要定义每一个属性的类型.如下图所示：

![clipboard.png][6]

        enum Episode {
          NEWHOPE
          EMPIRE
          JEDI
        }
        type Person{
             id: ID! 
             name: String!
             friends: [Person]
             appearsIn: [Episode]! 
        }   
        
上面是一个简单的类型定义，先是定义了一个枚举，然后我们定义了一个类型，类型中有四个属性：id、 name、 friends、 appearsIn,其中id和name是标量类型，而friends是一个Person类型，这是一个嵌套类型，仔细想想应该没什么毛病，毕竟你的朋友和你一样，都是人；而appearsIn是一个枚举类型，看起来还是很熟悉的；
了解完类型，再了解一下Arguments和resolver,两者都是偏服务端一些，但是了解一下，对graphql的使用原理有进一步的认识；
对于一个Restful API来讲，除了知道接口URL，我们还需要知道接口的传参定义，对于GraphQL其实也一样，虽然URL只有一个，不同的接口通过type来区别，但传参同Restful API一样，体现了客户端与服务端的交互。比如下面，查询的目标是id = 2的用户，获取他的用户名：  

        Query{
            user(id: 2) {
                id
                userName
            }
        }
而在服务端定义一个接口时，我们也需要去定义入参，主要从两个方面，一是类型，二是其是否必填，比如下面这样：
** 接口定义 **  

      user: {
        type: UserType,
        args: {
          id: { type: GraphQLID }
        },
        resolve: (root, args, context, info) => {
          const { id } = args;
          return getUser(id);
        }
      }
** 查询定义 ** 

![clipboard.png][7]

上面的代码只是定义了一个输入属性id,并未定义其是否是必填，所以当查询时，如果没有配置查询id，查询不会报错，只会获取一个为null的空值结果。但是讲道理的话，我们希望这是一个必填项，所以我们需要修改服务端的代码,将***id: { type: GraphQLID }*** 更换为***id: {type: new GraphQLNonNull(GraphQLID)}***，这句代码的含义就表示id是一个类型为ID的必填项，再次执行我们的查询可以得到下面的错误提示，提示id是一个必填的ID类型，同时右侧也没有获取到为空的查询结果；

![clipboard.png][8]
在讲上面Arguments时候，可以零星的看到type中有一个resolve方法，其接收root, args, context, info四个参数。其中root代表这个type上父节点的resolve值（因为GraphQL支持嵌套查询），args就是上面讲的，context表在Resolver解析链中不断传递的中间变量,和react的上下文相似，而info这个概念，是当前Query的AST对象，比较抽象，但是可以通过查看info,获取这个QUERY的编译对象。这个方法也是后端服务编写的重点部分，常常我们可以在这里与已有的Restful API关联起来。
#### 核心概念 ####

Schema可以说是GraphQL最具核心的部分，其描述了整个接口向外暴露的形式；像Restful API，我们会定义一个查询所有人的接口url定义为：/api/v1/user/getUsers，而查询人具体信息的接口url为：/api/v1/user/getUserById,而新增一个人员的接口url定义为：/api/v1/user/createUser，前端人员调用起来很直观。但是graphql是完全不一样的使用方式，其向前端暴露的url就一个像/api/graphql之类的，那这么多接口怎么区分呢？  
![clipboard.png][9]  

奥妙就是上面这张图，一个graphql接口都有一个Schema定义，其定义三种操作方式：query（查询）,mutation（变更）和subscription（监听）。再往下延伸，一个查询中包含多个field，也就是多种不同的查询，比如query user查询人，query message查询消息，query weather查询天气，通过这些就实现了Restful API使用多个url来达到不同操作的效果。看下面一张图（使用最原生的graphql语言编写，有助于理解）：

![clipboard.png][10]

### 总结 ###
这是一篇自己学习GraphQL期间，归纳总结到的信息，自己感觉是同现在的开发方式相比，需要在前端与Restful API之间增加一层GraphQL中间层，这确实增加了工作量，但看上面提到的语法，其编程还是非常直观易懂的，在业务不是那么繁重的时候，还是值得推广的。在下一篇将会讲到：实现一个GraphQL Server与在React应用中引入GraphQL，敬请期待。



  [1]: https://developer.github.com/v4/explorer/
  [2]: http://graphql.cn/learn/
  [3]: https://sfault-image.b0.upaiyun.com/264/039/2640397872-5b40710b46629_articlex
  [4]: https://sfault-image.b0.upaiyun.com/314/810/3148104696-5b4022513330a_articlex
  [5]: https://sfault-image.b0.upaiyun.com/109/513/1095133615-5b40272a2b8bd_articlex
  [6]: https://sfault-image.b0.upaiyun.com/799/440/799440574-5b402acff1ec4_articlex
  [7]: https://sfault-image.b0.upaiyun.com/141/228/1412283152-5b40b461166b7_articlex
  [8]: https://sfault-image.b0.upaiyun.com/134/053/1340537107-5b40b63fe6641_articlex
  [9]: https://sfault-image.b0.upaiyun.com/211/444/2114443209-5b40be02a416e_articlex
  [10]: https://sfault-image.b0.upaiyun.com/533/125/533125430-5b40c2e5b48d7_articlex
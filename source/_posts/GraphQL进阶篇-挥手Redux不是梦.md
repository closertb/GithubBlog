---
title: 'GraphQL进阶篇: 挥手Redux不是梦'
date: 2018-08-05 10:13:41
tags:
    - GraphQL
    - ApolloClient
---
[同学，GraphQL了解一下：基础篇][1]  
[同学，GraphQL了解一下：实践篇][2]
首先，需要澄清，这有点标题党，像Redux， Mobx，Flux这种状态管理库，在日常的开发中的地位还是难以撼动的，但是我们可以试着去了解ApolloClent，它在做本地状态管理所应用的思想，ApolloClient官方有一片文章：[The future of state management][3]。如果对GraphQL还不是很了解的同学，可以看一下开头的两篇文章。作为自己今年下半年学习的重点，如果仅仅去了解好像有点半途而废的感觉，所以我选择如果学，请深钻的道路。
文章所引用的[源码地址][4]

在[实践篇][5]的最后，我在最后一段抛出graphql怎么与现在的redux做集成，而[MagicPig][6]同学在评论里告诉我ApolloClent其实可以不依赖第三方库，自己做状态管理。当时自己入门不深，也是一脸懵逼，后面受其指点，在[ApolloClent官网][7]转悠，发现还有很多宝藏可以挖掘。  

### 用ApolloClent代替Redux ###
在Redux的官方教程中，曾用一个TodoList来介绍Redux的状态管理,看下图：

![clipboard.png][13]

这上面的演示，如果你不是一个react新手，应该不会太陌生。在react应用中，加入redux，实现本地添加list条目与条目状态切换，以及列表的过滤条件切换，如果关于它的实现还不是很了解，可以到[Redux官网][8]重新温习一次。
ApolloClent的[Local state management][9]章节，为了说明怎样用ApoloClient管理应用的本地状态（Learn how to store your local data in Apollo Client），官方提供了一个示例，应用其state功能以及grapql本地查询语法，实现了一个拥有同样功能的TodoList，[CodeSandBox源码地址][10]，不过官方提供的这个在线演示，好像是少了些东西，我并没有完全跑成功，我把东西down下来，改把，改把，在本地还是跑成功了，想了解的，可以通过上方的地址下载。
### 基础知识梳理 ###
在实践篇中创建一个client实例代码是这样的：  

    import { ApolloProvider } from 'react-apollo';
    import ApolloClient from "apollo-boost";
    const client = new ApolloClient({
      uri: 'http://localhost:8080/graphql',  // 服务端接口
      batchInterval: 10,
      opts: {
        credentials: 'cross-origin', // App端单独跑了一个服务，所以涉及到跨域；
      },
    });
上面的代码，就是建立了一个远程的Graphql操作服务，而在这里，我们需要加入本地的状态管理，代码变成了这样：  

    import { ApolloProvider } from 'react-apollo';
    import { ApolloClient } from 'apollo-client';
    import { withClientState } from 'apollo-link-state';
    import { HttpLink } from 'apollo-link-http';
    import { InMemoryCache } from 'apollo-cache-inmemory';
    import { resolvers, typeDefs, defaults } from '../client/index';
    const cache = new InMemoryCache();
    const client = new ApolloClient({
      cache, // 本地数据存储
      link: withClientState({ resolvers, defaults, cache, typeDefs }).concat(
        new HttpLink({
          uri: 'http://localhost:4001/graphql',
          batchInterval: 10,
          opts: {
            credentials: 'cross-origin',
          },
        })
      ),
    });
首先，ApolloClient 这个对象引入的NPM包变了，以前是从apollo-boost引入的，现在是从apollo-client引入的。其次这里加入的本地状态管理，是用withClientState创建了一个link对象，传入了四个参数（resolvers, defaults, cache, typeDefs），cachce很简单，就是上面new InMemoryCache()创建的本地存储，这里简单说明一下resolvers, defaults, typeDefs。
#### 基本定义 ####
首先需要知道的，ApolloClient所建立的状态管理思想与Redux的操作思路基本一致。只是实现上。ApolloClient的本地状态管理，是用Graphql那一套来做的，即query, root, resolver, schema这些概念，建立一套本地的Query（query, mutation, subscrition）。
 - Defaults: 这个和我们写Redux一样，通常需要定义一个initialState, 所以defaults是一个为你应用定义的一个初始化对象，这个对象将会被写入cache，在做客户端查询时，定义一个完整的初始化对象，其有助于减少很多错误，比如，你没有定义，但是去操作它，通常会报一个，you can't read the propery 'xxx' of undefined；
 - Typedefs: typeDefs其实是一个定义本地查询的Schema, 只不过其加入了更多的语法糖，不用像我们在实践篇用原生graphql语言写出的那样冗长, 但其确实就是一个Scehma，定义了查询对象与各做操作；
 - resolvers：这个其实就是描述所有在Schema提到的resolver，总共三类：Query, Mutation, Subscrition，其语法也和服务端的语法一致，每个resolver是一个函数，其包括了四个传参(root，args, context, info)；
其次,由于ApolloClient所建立的本地状态管理，其实建立的是一个本地graphql服务，所以不管是对本地还是远程，我们都是用graphql语言来进行描述，所以，区分本地与远程就显得十分重要。前者在新建查询时多了一个** @client **参数,比如:  

        const query = gql`
          query GetTodos {
            todos @client {
              id
              text
              completed
            }
          }
        `;  
      
#### 更新本地状态 ####
ApolloClient提供了两种方式来更新本地状态：Direct writes 与 resolver。Direct writes就是new出来的这个cache对象，其包含了一些方法，可以直接对state的数据进行操作，它没有采用graphql的突变语法来进行数据操作，所以不会执行数据类型的校验，这种方式只适用于一些简单的状态更新，如果这个状态对你的应用很重要，那就应该用更安全的resolver方式来代替。resolver在前面已经提到，它是Mutation的处理方法，会告诉graphql怎样更新数据。在后面我们做数据状态更新时，其实也有两种实现方式，一种是实践篇用到的那样，用graphql创建一个带变更操作的高阶组件（在实践篇用到的那样），另一种是直接用react-apollo提供的Mutation组件，示例：

    const TOGGLE_TODO = gql`
      mutation ToggleTodo($id: Int!) {
        toggleTodo(id: $id) @client
      }
    `;
    const Todo = ({ id, completed, text }) => (
      <Mutation mutation={TOGGLE_TODO} variables={{ id }}>
        {toggleTodo => (
          <li
            onClick={toggleTodo}
            style={{
              textDecoration: completed ? 'line-through' : 'none',
            }}
          >
            {text}
          </li>
        )}
      </Mutation>
    );  
    
#### 状态查询 ####
状态的查询与读取，是一个最基本的需求，查询语法与服务端语法一致。但不同的是，除了在加载页面的时候需要查询状态，在变更状态时，有时也需要先查询某些关联的状态，然后再做其他操作，比如下面这样：  

    toggleTodo: (_, variables, { cache }) => {
      const id = `TodoItem:${variables.id}`;
      const fragment = gql`
        fragment completeTodo on TodoItem {
          completed
        }
      `;
      const todo = cache.readFragment({ fragment, id });
      const data = { ...todo, completed: !todo.completed };
      cache.writeData({ id, data });
      return null;
    }  
    
上面这一段代码，是关于todoList中的每条List的状态切换，单击条目将其状态从代办变为已办，或从已办变为代办。代码的实现中有一段为cache.readFragment，它的目的就是从cache中的TodoItem属性中获取某个特定id条目的状态，然后取反重新写入。除了cache.readFragment,还有像cache.readQuery这样的方法，因为这是本地的状态管理，所以这个是一个同步的操作，就不涉及promise的概念。更多关于cache方法的操作可查看[官网文档][11]。
### 写一个本地与远程的状态管理应用 ###

![clipboard.png][14] 
接下来，将会与我们的日常实践更加接近，就是用apolloClient代替现有的redux，结合antd做一个中后台最常见的列表查询页面。一个典型的列表查询页，基本由两部分组成，一个Search查询表单头，一个查询结果展示的table。从数据结构以及流程上来说，基本是这样的：

![clipboard.png][15] 

不论是redux还是apolloClient,其实从大体流程来讲，都是这个思路，只是具体的实现有差别，为了实现简便，用了一个tabBar来代替Search,通过tab的切换来改变status，然后发送请求，更新list，来看一下具体实现：  

** index.js **  

    import TabBar from './TabBar';
    import Content from './ContentHoc';
    const GET_STATUS = gql`
      {
        readStatus @client
      }
    `;
    // 每次页面渲染前，从cache中读取status的值
    const BookList = () => (
      <Query query={GET_STATUS}>
        {({ data: { readStatus } }) => {
          return (
            <div>
              <TabBar status={readStatus} />
              <Content status={readStatus} />
            </div>
          );
        }}
      </Query>
    );
每次页面渲染前，从cache中读取status的值，然后将其作为props传递到TabBar与Content组件。  

** TabBar.js **  

    import { Mutation } from 'react-apollo';
    import { Tabs } from 'antd';
    import gql from 'graphql-tag';

    const TabPane = Tabs.TabPane;
    const ReadStatus = [{
      label: '总书单',
      value: '',
    }, {
      label: '已读',
      value: 'read',
    }, {
      label: '期望读',
      value: 'wish',
    }, {
      label: '正在读',
      value: 'reading',
    }];
    const ChangeStatus = gql`
      mutation ChangeStatus($status: String){
        changeStatus(status: $status) @client
      }
    `;
    export default class TabBar extends Component {
      constructor(props) {
        super(props);
        this.state = {};
      }
      render() {
        const { status } = this.props;
        return (
          <Mutation mutation={ChangeStatus} >
            {changeStatus => (
              <Tabs defaultActiveKey={status} onChange={(value) => { changeStatus({ variables: { status: value } }); }}>
                {ReadStatus.map(({ label, value }) =>
                  <TabPane tab={label} key={value} />)}
              </Tabs>
            )}
          </Mutation>
        );
      }
    }
  
tabBar组件根据拿到的status,渲染tab的选中状态，同时给Tabs增加了相应的点击事件，来触发cache中readStatus值的变更。  

** ContentHoc.js **  

    import { Query } from 'react-apollo';
    import { Table } from 'antd';
    import gql from 'graphql-tag';
    
    const columns = [{
      title: '序号',
      dataIndex: 'book_id',
      key: 'id',
    }, {
      title: '书名',
      dataIndex: 'title',
      key: 'title',
    }, {
      title: 'url',
      dataIndex: 'image',
      key: 'image',
    }];
    export const BOOKS_QUERY = gql`
      query($status: String){
        collections(status: $status) {
          total
          collections {
            book_id
            title
            image
          }
        }
      }
    `;
    
    export default class BookList extends Component {
      constructor(props) {
        super(props);
        this.state = {};
      }
      render() {
        const { status } = this.props;
    
        return (
          <Query query={BOOKS_QUERY} variables={{ status }}>
            {({ loading, error, data }) => {
              if (loading) {
                return <div className="loading">Loading...</div>;
              }
              if (error) {
                return <div className="loading error">error</div>;
              }
              const { collections: lists, total } = data.collections;
              const tableProps = {
                dataSource: lists,
                columns,
                rowKey: 'book_id',
              };
              return (
                <div>
                  <p className="total">总共有<span>{total}</span>本图书</p>
                  <Table {...tableProps} />
                </div>
              );
            }}
          </Query>
        );
      }
    }  
    
这一部分应该是与我们使用Redux区别最大的部分，传统的Redux用法会将list的获取与保存放置在容器组件中，然后通过props传递到展示组件。而在这里，利用了apolloClient提供的Query组件，来做以前容器组件干的活。然后以前我们需要在请求的过程中捕获错误或请求状态，而在这里，Query组件提供了一系列的属性（loading，error）,可以直接使用，无需自身维护。
另外，为了调试方便，apolloClient还提供了像React-developer-Tool一样的调试工具（需要梯子）：[Apollo Client Devtools][12]
### 使用总结 ###
通过一个实践，自我感觉其实不管使用Redux还是apolloClient，我们都采用了相同的思路，只是具体的实现方式有差别，或则说Redux与apolloClient用两种不同的手段达到了同一种效果：Redux的dispatch Type 与 apolloClient的query @client查询。另外，就上面这种简单纯粹的中后台系统，使用apolloClient就已足够，不需要再加入Redux家族来帮忙处理。这个月被借调去支撑另一个团队，学习的步伐好像又要放慢了。哎。。。。。。
  [1]: http://closertb.site/2018/07/%E5%90%8C%E5%AD%A6%EF%BC%8CGraphQL%E4%BA%86%E8%A7%A3%E4%B8%80%E4%B8%8B%EF%BC%9A%E5%9F%BA%E7%A1%80%E7%AF%87/
  [2]: http://closertb.site/2018/07/%E5%90%8C%E5%AD%A6%EF%BC%8CGraphQL%E4%BA%86%E8%A7%A3%E4%B8%80%E4%B8%8B%EF%BC%9A%E5%AE%9E%E8%B7%B5%E7%AF%87/
  [3]: https://blog.apollographql.com/the-future-of-state-management-dd410864cae2?_ga=2.224183315.902703595.1532831247-1476755624.1530107243
  [4]: https://github.com/closertb/graphDemo
  [5]: http://closertb.site/2018/07/%E5%90%8C%E5%AD%A6%EF%BC%8CGraphQL%E4%BA%86%E8%A7%A3%E4%B8%80%E4%B8%8B%EF%BC%9A%E5%AE%9E%E8%B7%B5%E7%AF%87/
  [6]: https://segmentfault.com/u/MagicPig
  [7]: https://www.apollographql.com/docs/react/essentials/local-state.html
  [8]: http://www.redux.org.cn/docs/basics/
  [9]: https://www.apollographql.com/docs/react/essentials/local-state.html
  [10]: https://codesandbox.io/s/github/apollographql/apollo-link-state/tree/master/examples/todo
  [11]: https://www.apollographql.com/docs/link/links/state.html
  [12]: https://chrome.google.com/webstore/detail/apollo-client-developer-t/jdkknkkbebbapilgoeccciglkfbmbnfm
  [13]: https://sfault-image.b0.upaiyun.com/281/314/2813145229-5b5bc2de75d9e_articlex
  [14]: https://sfault-image.b0.upaiyun.com/387/015/3870156966-5b655bb55a903_articlex
  [15]: https://sfault-image.b0.upaiyun.com/175/760/1757609678-5b6565679e080_articlex
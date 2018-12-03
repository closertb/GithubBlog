---
title: 同学，GraphQL了解一下：实践篇
date: 2018-07-08 12:24:47
tags:
    - GraphQL
    - Express
    - Apollo
    - React
---
上一篇：[GraphQL了解一下：基础篇][1]
在基础篇主要讲了GraphQL出现的意义与一些基础语法。如果对GraphQL还不是很了解的同学可以点击上方链接了解一下，再来跟进这一篇的实践。本篇主要讲述实现一个GraphQL Server与在React应用中引入GraphQL，代码不对，推荐跟着手敲一遍。
下面文章的代码都可以通过github下载：[地址传送][2]，里面包含两个文件夹：graphqlServer(服务端)与graphqlApp(客户端)。
### 实现一个GraphQL Server ###
#### 技术栈准备 ####
核心依赖（npm包）：
express：Node服务端框架；
apollo-server-express：express graphql中间件，提供graphiqlExpress与graphqlExpress两个方法；
graphql：graphql js实现基础库；
axios：ajax通信，这里用于和已有的Restful API通信；
除了安装以上的核心依赖，你还需要安装babel相关的依赖，并配置babel编译文件，具体可查看上面git下来的文件配置。
#### 搭建服务 ####
怎么引入包这里不再赘述，我们先不带入graphql，启动一个express服务：    

        const PORT = 8080;
        const app = express();  // 创建一个express服务；
        app.use('/graphql',  (req, res) => {
          res.send('Hello GraphQL!');
        });
        app.listen(PORT, () => console.log(`> Listening at port ${PORT}`));
启动这个服务，并在浏览器输入http://localhost:8080/graphql，可以看到Hello GraphQL这段欢迎词，到这里我们的后端服务已经搭建成功，接着我们创建我们的GraphQL服务，删除监听'/graphql'这个路由的代码，添加一下一段代码：  

        import schema from './schema';
        const PORT = 8080;
        const app = express();  // 创建一个express服务；
        app.use(cors()); //这里添加cors，是因为我们后面前端会单独跑一个服务，所以涉及到前后端跨域
        app.use('/graphql', graphqlExpress({ schema }));
        app.use('/graphiql', graphiqlExpress({
        	endpointURL: '/graphql'
        }));
        app.listen(PORT, () => console.log(`> Listening at port ${PORT}`));
至此，我们就添加了GraphQL服务，graphql是用于接收url请求的，而graphiql会呈现一个graphql查询界面，这个界面可以用于查询体验，查看文档定义，这是graphql官方比较推荐的一个技术，就像下面这样：

![clipboard.png][3]
上面我们略过了schema，其实这个在上一篇就花了一定篇幅来讲解，其定义了整个graphql服务所支持的接口定义，照样贴上代码：  

        import {
          GraphQLObjectType,
          GraphQLSchema,
          GraphQLInt,
          GraphQLID,
          GraphQLString,
          GraphQLList,
          GraphQLNonNull,
        } from 'graphql/type';
        import { getUser, getUsers, getUserMixNick } from '../service/index';
        
        // 查询某一个user的详细资料模型
        const UserType = new GraphQLObjectType({
          name: 'User',
          fields: {
            id: { type: GraphQLInt },
            userName: { type: GraphQLString },
            userMixNick: { 
              type: GraphQLString,
              args: {
                id: {
                	type: new GraphQLNonNull(GraphQLID)
                }
              },
              resolve: (root, args, context, info) => {
                const { id } = root;
                console.log(info)
                return getUserMixNick(id);
              }
             },
            military: { type: GraphQLString },
            age: { type: GraphQLInt },
            height: { type: GraphQLInt },
            education: { type: GraphQLString },
            enlistTime: { type: GraphQLString },
            enlistYear: { type: GraphQLInt },
          }
        });
        
        // 查询所有的users
        const PaginationType = new GraphQLObjectType({
          name: 'Pagination',
          fields: {
            pageSize: { type: GraphQLInt },
            pageNum: { type: GraphQLInt },
            total: { type: GraphQLInt },
            data: {
              type: new GraphQLList(UserType)
            }
          }
        });
        // 定义schema
        const schema = new GraphQLSchema({
          query: new GraphQLObjectType({
            name: 'militaryQuery',
            fields: {
              user: {
                type: UserType,
                args: {
                  id: {
                  	type: new GraphQLNonNull(GraphQLID)
                  }
                },
                resolve: (root, args, context, info) => {
                  const { id } = args;
                  return getUser(id);
                }
              },
              users: {
                type: PaginationType,
                args: {
                  pageNum: { type: GraphQLInt },
                  pageSize: { type: GraphQLInt }
                },
                resolve: (root, { filters, pageNum, pageSize }) => {
                  return getUsers(filters, pageNum, pageSize);
                }
              }
            }
          })
        });
        
        export default schema;  
        
这里不想花太大的篇幅去讲解，可以git clone下来自己尝试一下，至此我们就成功的创建了一个GraphQL服务，可以在graphiql界面查询体会一下。
### 在React应用中引入GraphQL ###
核心依赖（npm包）：
react相关： 什么react,webpack，react-router-dom这些;
react-apollo与apollo-boost: 用于在app端创建一个GraphQL服务连接实例;
graphql与graphql-tag: 用于在app端发出一个GraphQL请求;
#### 页面结构 ####

![clipboard.png][4]

页面大致是这样的一个结构，进入App的主页时，会加载兄弟连中主要的战士列表，点击查看详情，可以看到这名展示的一些详细信息，页面结构代码。

      <ApolloProvider client={client}>
        <Router>
          <div>
            <Route exact path="/" component={List} />
            <Switch>
              <Route exact path="/:id/detail" component={Detail} />
            </Switch>
          </div>
        </Router>
      </ApolloProvider>
这里需要强调一下client代码的实现：  

        const client = new ApolloClient({
          uri: 'http://localhost:8080/graphql',  // 服务端接口
          batchInterval: 10,
          opts: {
            credentials: 'cross-origin', // App端单独跑了一个服务，所以涉及到跨域；
          },
        });      
#### 列表页的实现 ####
react-apollo在实现graphql结合react编程的方式上，借鉴了类似react-redux的connect高阶组件的思想，react-apollo提供一个方法graphql用于生成一个容器，这个容器会从远端拉去数据，然后作为props传递给展示组件,直接看代码的实现： 

          // 创建一个查询  
          const USERS_QUERY = gql`
              query UserQuery($pageNum: Int,$pageSize:Int){
                users(pageNum:$pageNum,pageSize:$pageSize ) {
                  pageNum
                  pageSize
                  total
                  data {
                    id
                    userName
                  }
                }
              }
        `;
        // 生成一个graphql容器，会执行USERS_QUERY这个查询；
        const withQuery = graphql(USERS_QUERY, {
          options: () => ({
            variables: {
              pageNum: 3,
              pageSize: 8
            },
          }),
        });
        // 列表展示组件
        class List extends Component {
          constructor(props) {
            super(props);
            this.state = {};
          }
          render() {
            const { data: { loading, users } } = this.props;
            if (loading) {
              return <div className="loading">Loading...</div>;
            }
            const { data: lists, total } = users;
            return (
              <div>
                <p className="total">总共有<span>{total}</span>名军士</p>
                <ul className="list">
                  {
                    lists.map(({ userName, id }, key) =>
                      <li key={key}>
                        <span>姓名：{userName}</span>
                        <Link to={`/${id}/detail`} >详情</Link>
                      </li>
                    )
                  }
                </ul>
              </div>
            );
          }
        }
        // 将数据注入到展示组件中
        const Character = withCharacter(List);
        export default Character;  
        
以上就是对列表展示组件的实现，思想还是比较简单，和我们基于react + redux编程比较像。
#### 详情页的实现 ####
其实现思路和列表页其实是一样的，只是涉及到动态传参的问题，上查询那一段代码说一下：  

        const USER_QUERY = gql`
            query UserQuery($id: ID!){
              user(id:$id) {
                id,
                userName,
                age,
                military,
                height,
                education,
                enlistTime,
                enlistYear,
              }
            }
        `;
        const withQuery = graphql(USER_QUERY, {
          options: (props) => {
            const { match: { params } } = props; // 重点就是从props中获取路由传递的参数
            return {
              variables: {
                id: params.id
              },
            };
          },
        });  
        
withQuery这个高阶组件同样可以从父组件中获取props，然后通过其option方法动态的生成查询参数，至于详细页展示组件的实现，可以具体参考git上面的代码。
到最后输入url可以看到如下的结果：

![clipboard.png][5]

### 思考 ###
从写一个demo的角度来讲，在react中嵌入graphql好像比在在react中嵌入redux还简单，但如何在我们现有的框架中去嵌入graphql呢？比如Dva + graphql，比如react + redux + redux-thunk + graphql，在我的认知范围里，好像还需要时间去评估这一切值不值得，反正技术上肯定是可以实现的。引入graphql有多大价值，这也需要结合具体项目，具体业务，具体团队来说。愿你这两篇文章读完对GraphQL已经有了一个比较完整的认识，happy Ending!!!
  [1]: http://closertb.site/2018/07/%E5%90%8C%E5%AD%A6%EF%BC%8CGraphQL%E4%BA%86%E8%A7%A3%E4%B8%80%E4%B8%8B%EF%BC%9A%E5%9F%BA%E7%A1%80%E7%AF%87/
  [2]: https://github.com/closertb/graphDemo
  [3]: https://sfault-image.b0.upaiyun.com/292/125/2921257029-5b416f8d19145_articlex
  [4]: https://sfault-image.b0.upaiyun.com/407/588/4075889205-5b4183b3532ea_articlex
  [5]: https://sfault-image.b0.upaiyun.com/128/390/1283900726-5b418f5480610_articlex
---
title: 一个javaWEB伪全栈的VUE入坑随笔:从登录开始（VUEX）
date: 2017-04-15 21:28:17
categories: '框架学习'
tags:
---
写于：2017-04-15  
## 扯
此文章用于记录本人VUE学习历程，有共同爱好者可加好友一起分享。
从上周天，由于本周有公司篮球比赛，所以耽误两天晚上，耗时三个晚上勉强做了一个登录功能。中间的曲折只有自己知道，有两天调到晚上1点。闲话少扯，说正经的。  

## 起
想做好一个登录，你需要用到VUE的VUEX（状态存储），VueRouter（路由切换），还有一些零散的VUE知识点，比如计算属性。由于用VUE开发和以前自己JAVA WEB项目有区别，登录状态是存储在浏览器的，和前者存在服务器有一定差别，当然也可以用AJAX和服务器实时交互，获取登录状态与其他信息，所以你这里需要用到浏览器的STORAGE。

- STORAGE，打开CHROME的DEVTOOL，在APPLICATION栏，你可以看到如图一的几种STORAGE类型。具体可以[查看博客][1]；

- VUEX，VUE的状态管理插件，具体[可查看][2]；

## 弯路
分享自己调了一晚上才走出来的弯路
store在VUE中是一个关键字，当我IMPORT我的VUEX组件USERSTORE时，我将其重命名为userstore，而没有用store，然后模仿store使用this.$userstore.dispatch调用我的ACTION，怎么看都感觉对，但是就是API看的不仔细。b、关于VUEX五个属性的关系，[推荐查看][8]，讲的浅显易懂，类比到位，但学深还是看官方API比较靠谱。

- VueRouter，VUE的路由管理插件，用过STRUTS2的人应该知道，他的路由配置作用类似STRUTS.XML，具体[可查看][3]：

- 计算属性，就是根据输入计算输出，自触发的，和METHODS有区别，具体[可查看][4]
- 另外，VUECOMPONENT的灵活应用也很重要。
- 本文重点，我的LOGIN实现原理，[源码地址][5]。  

## 说重点
首先我们应该理清思路，做一个登录功能，我们需要做以下几步：

   - 0. 这里少了一步，用户打开页面时，我们需要检查其浏览器是否保存了他的登录状态，若无，进行a操作；

   - a、输入用户信息，并提交；

   - b、检查用户数据是否为空，是否符合规范，若无，进行下一步，表单验证；

   - c、根据表单信息进行验证，若成功，写浏览器storage，并更新用户登录状态；

   - d、登录成功，跳转到主页或其他；  

  新建登录窗口（Login.vue）：

```
<template>
<div>
  <v-header title='Login'>
  </v-header>
  <div class='hello'>
    <h1 v-if='!logstate'>{{ msg }}</h1>
    <section v-if='!logstate'>
      <form class='loginForm' @submit.prevent='login()'>
      <div class='line'>
        <input type='text' placeholder='输入你的用户名' v-               model='form.name'>
        <div v-show='btn && !form.name'>用户名不能为空</div>
      </div>
      <div class='line'>
        <input type='text' placeholder='输入你的密码' v-model='form.id'>
        <div v-show='btn && !form.id'>密码不能为空</div>
      </div>
      <button>Login the APP</button>
    </form>
    </section>
    <section v-if='logstate'>
      <h2>欢迎你：{{userInfo.name}}</h2>
      <p><button @click='loginout'>退出</button></p>
    </section>
  </div>
  </div>
</template>
```
   代码有一个很简单的逻辑，当用户进入到这个APP时，如果浏览器有用户登录信息，即渲染欢迎页面，否则，渲染登录页面。涉及到表单的提交，那就必须有提交前的检查和提交后一系列的登录验证，当然，学习可以前端做个假登录，自己为了好玩，用Java写了个后台，两者通过AJAX传递数据，进行登录验证。既然提到了方法，那接着我们就需要写SCRIPT文件了

```
<script>
import {mapState} from 'vuex'  //可以暂时忽略他的存在

export default {
  name: 'Login',
  data () {
    return {
      msg: 'Please Login the App First',
      btn:false,
      form: {
					id: '',
					name: ''
				}
    }
  },
  computed: mapState({  //可以暂时忽略他的存在
    logstate:'loginState',
    userInfo:'userstate'
  }),
  methods:{
			  login:function(){
				this.btn = true;
        console.log('start:');
        var user=this.form;
				if(!this.form.id || !this.form.name) {
          return;
        }else{
          this.$store.dispatch('userSignin',user);  //可以暂时忽略他的存在
          this.$router.replace({ path: '/' }) //登录成功，则跳转到主页
        }
			},
      loginout:function(){
          return this.$store.dispatch('userSignout'); //可以暂时忽略他的存在
      }
    }
 }
</script>

```

上面可以看到，除了VUEX的知识，其他的都是挺好懂得。所以现在我们来了解VUEX的状态管理。 还是直接上代码，然后再分析吧，分析见代码；
```javascript
import Vue from 'vue'  //引入VUE和VUEX
import Vuex from 'vuex'
Vue.use(Vuex);  //这一句很重要，不信可以删了试试
const userstore = new Vuex.Store({  //新建一个Vuex.Store对象
  state: {  //原始对象属性之一
    userstate:JSON.parse(sessionStorage.getItem('user')) || {},//用户信息
    loginState:false||(Boolean(JSON.parse(sessionStorage.getItem('user'))))//登录状态
  },
  mutations: {   //原始对象属性之一
    increment:function(state){  //这是哪个博客的测试函数
        state.count  = 2;
    },
    userSignin:function(state, user){
            var userstate=state.userstate;
            sessionStorage.setItem('user', JSON.stringify(user));//向浏览器存储写入用户信息
            state.loginState=true;
            Object.assign(userstate, user);
    },
    userSignout:function(state) {
            sessionStorage.removeItem('user');
            state.loginState=false;
            Object.keys(state.userstate).forEach(k => Vue.delete(state.userstate, k))
    }
  },
  actions:{  //原始对象属性之一
    increment:function({ dispatch, commit }) {   //这是哪个博客的测试函数
      return commit('increment');
    },
    userSignin:function({commit}, user){ //登录
      console.log('new:' user.name '***mima:' user.id);
        var loginFlag=true; //false
              if(loginFlag){
                commit('userSignin', user);
              }

  },
  userSignout:function({commit}) { //退出
      commit('userSignout');
  }
  }
});
export default userstore; //输出模板
```
其实回头再仔细看看，当时自己咋就那么蠢呢，这么简单的弄半天。还有另一个原因，很多VUE相关的实例，大神门都是用ES6语法写的，我这ES5还没嚼透的，根本就还没看过什么箭头函数，let,con这些新语法，不过后面真的好好去学一学。回到正题，这里的执行顺序，大概说一下。  
 - state：这是状态字，APP要用到的全部管理字都需要在这里进行先定义好，这也是向其他组件暴露状态值的最佳捷径，后面，计算属性，直接和他挂上钩。这里的userstate（用户信息）和loginstate（登录状态）都是直接取的SESSIONSTORAGE中的用户信息进行属性值初始化。
 - actions：这里是this.$store.dispatch事件分发的入口，事件是没法直接分发到mutations，这里包含了登录和退出两个ACTIONS，具体实现用ES5写出了，还是很浅显易懂的，commit('Mutations')就是执行MUTATIONS中的相应方法；
 - mutatons:他和ACTIONS的最大区别就在于它会接受 state 作为第一个参数，而ACTION不可以，你自己把state加进去作为输入参数，你用console.log打印出来，可以看到是未定义的，所以官方文档会说：更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。

至此，好像要讲的差不多了，错，我们虽然写了这么多，但怎么把他们结合好，还需要在Main.js中做如下处理：
```
import store from './store/userstore' //引入userstore，别乱起名字了，就用官方的
new Vue({
  el: '#app',
  router,
  store, //全局注册该组件
  template: '<App/>',
  components: { App }
})
```
我们再回到LOGIN.VUE中的方法，现在就可以好好讲讲我们前面忽略的代码了吧
```
import {mapState} from 'vuex'  //引入mapState，方便我们直接索引到状态值

  computed: mapState({   //这是计算属性，实时获取state的值，供页面渲染用
    logstate:'loginState',//这里只所以可以在LOGIN.VUE中获取USERSTORE的STATE值，
    userInfo:'userstate'  //就多亏了mapState
  }),
以下两句，就是调用userstore中的登录方法，成功后，跳转到主页面
this.$store.dispatch('userSignin',user);  //可以暂时忽略他的存在
this.$router.replace({ path: '/' }) //登录成功，则跳转到主页
```
好像该讲的都讲的差不多了，但还差最后一步，就是：用户打开页面时，我们需要检查其浏览器是否保存了他的登录状态，若无，才进行登录操作；
这里需要在APP入口中，就开始检测:这里需要VUEROUTER和VUEX结合着用
```
Vue.use(VueRouter)
router.beforeEach(({meta, path}, from, next) => {
    var { auth = true } = meta
    var isLogin = Boolean(store.state.loginState) //true用户已登录， false用户未登录
    if (auth && !isLogin && path !== '/Login') {
        return router.replace({ path: '/Login' })
    }
    next()
})
```
是不是简单易懂。
没有买自己的服务器，只有上两张自己APP效果截图，供大家点评一下
![登录界面][6]


![图片描述][7]


  [1]: http://www.cnblogs.com/yuzhongwusan/archive/2011/12/19/2293347.html
  [2]: https://vuex.vuejs.org/zh-cn/
  [3]: https://router.vuejs.org/zh-cn/
  [4]: http://cn.vuejs.org/v2/guide/computed.html
  [5]: https://github.com/closertb/MyLogin
  [7]: https://sfault-image.b0.upaiyun.com/256/388/2563887779-58f10196e692f_articlex
  [6]: https://sfault-image.b0.upaiyun.com/239/247/239247719-58f1016f63bb0_articlex
[8]:：https://zhuanlan.zhihu.com/p/24357762
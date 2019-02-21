---
title: 从dist到es：发一个NPM库，我蜕了一层皮
date: 2019-02-21 23:17:45
tags:
---
这并不是自己第一次发npm包， 所以这里没有多少入门的知识。在此之前已经有一篇[前端脚手架，听起来玄乎，实际呢？][1]，但这一次的npm包和上一次的不是一个概念，前者只是一个脚本工具，而这个npm包是日常开发中方法和组件的集合, 是一个库。
在读本文前，假定你已经对npm包有一定概念，熟悉Babel编译和webpack打包的常规用法，知道一些前端工程化的知识。假如你也想自己发布一个npm仓库，但对这一块了解的不是很多，推荐webpack官方的[创建一个 library][2] 

### 打包模式：日常构建与库的构建 ###
在前端日常开发中，引入npm库，执行webpack构建已经是一件不能再平常的事情。但大多数时候，我们不关心这个npm库是怎样构成的，我们只需要知道怎么使用，像antd；在工程化成熟的公司，也不关心webpack的配置到底是怎样的，只需要npm run start或npm run build去启动一次热加载或打包。但是如果你是编写一个npm仓库，这些东西你都需要知道。从那时的无知说起，起初，我用公司的构建工具（类似于roadhog）去打包我的库，没有坎坷，构建出一个2M多的包并成功发布。

![clipboard.png][10]

在测试项目中引入，构建成功。
```
import { EnhanceTable, WithSearch } from 'antd-doddle';  // 引入仓库
```
在浏览器中打开，打击开始到来：
![clipboard.png][11]
提示要引入的对象没有正确导出，就是没有做module.export，所以这是一个打包模式的问题，[output.libraryTarget][3]需要了解一下。

![clipboard.png][12]
webpack的output.libraryTarget决定了打包时对外暴露出来的对象是那种模式，默认是var，用于script标签引入，该模式也是我们日常开发构建最常用的模式，除了这一种，还支持的常用选项有：
 - commonjs(2)：node环境CommonJS规范，关于[commonjs与commonjs2的区别][4]，commonjs 规范只定义了exports，而 module.exports是nodejs对commonjs的实现，实现往往会在满足规范前提下作些扩展，所以把这种实现称为了commonjs2；
 - amd：amd规范，适用于requireJS；
 - this：通过 this 对象访问（libraryTarget:'this'）；
 - window：通过 window 对象访问，在浏览器中（libraryTarget:'window'）。
 - UMD：将你的 library 暴露为所有的模块定义下都可运行的方式。它将在 CommonJS, AMD 环境下运行，或将模块导出到 global 下的变量,（libraryTarget:'umd'）。
 - jsonp：这是一种比较特殊的模式，适用于有extrnals依赖的时候(splitChunks)。将把入口起点的返回值，包裹到一个 jsonp 包装容器中。
所以在这里我们需要设置两个属性来明确打包模式
```
    library: 'antd-doddle',
    libraryTarget: 'umd',
```
![clipboard.png][13]
### 2M到38KB, 这中间发什么了什么 ###
![clipboard.png][14]
上图是用roadhog打包出来的结果，其显示的是开启gzip后可以压缩到的大小，第一次打包的实际大小大概在2M（antd+moment+react+css），后面仔细一想，公司的组件库也才300kb啊，自己是不是哪里搞错了，所以接着就有了下面的探寻之路。
 1. 抽离css（300kb）,由于此npm库是基于antd的，所以就没有再把antd的css打包一次的必要了。基于roadhog给予的提示，配置了disableAntdStyle为false，css文件降到2kb；
 2. 接着上面，虽然是基于antd的，但并没有完全用到antd的所有组件，其官方提供了一个按需打包babel插件babel-plugin-import，并在babelrc中配置, js打包体积由1.6M降为1.2M;
```
    ["import", {
      "libraryName": "antd",
      "libraryDirectory": "lib",
      "style": "css", 
    }]
```
 3. 如果对webpack多了解一下，或者在写一个库之前读过 [创建一个 library][5]，就会发现前面两点都是白扯没有用的，因为对于这个库来说antd就是一个外部依赖（externals），正好roadhog又支持, 打包出来，由1.2M变为38kb, 这是一个质的提升。
```
  externals: {
    react: {
      commonjs: 'react',
      commonjs2: 'react',
        amd: 'react',
    },
    antd: {
      commonjs: 'antd',
      commonjs2: 'antd',
        amd: 'antd',
    },
    moment: {
      commonjs: 'moment',
      commonjs2: 'moment',
        amd: 'moment',
    },
  }
```
打包大小优化至此就搞定了，但后面发现用roadhog打包库有一些很难解决的难题，为了解决还得去了解他源码逻辑，所以后面还是自己写了一个webpack，非常简单的配置。
###ES6之后，光有dist是不够的###
在写这个库之前，我曾想到在我们日常构建时有下面这样一段配置:
```
    rules: [{
      test: /\.js$/,
      exclude: /(node_modules|bower_components)/,
      loader: 'babel-loader',
      query: {
        presets: ['@babel/preset-env', '@babel/preset-react']
      }
    }
```
这段配置是告诉webpack，node_modules中引用的代码不需要再由babel编译一次，但这些代码还是会被打包进dist文件的。在现在前端的主流开发套路中，被引用的库更希望是一个只编译而没有被打包过的，支持按需加载的库。所以dist中被编译打包过的代码再次被打包, 这样就会有不必要的代码出现。所以这样来看，我们只需要将代码编译。babel,没错，就是它，但是多文件编译，还是找个第三方构建工具比较好，我选择了gulp，直接上代码：
```
// 发布打包
gulp.task('lib', gulp.series('clean', () => {
  return gulp.src('./src/**/*.js')
    .pipe(babel())
    .pipe(gulp.dest('./lib'));
}, 'lessToLib')); // lessToLib用于将less文件拷贝贷lib文件夹
```
编译过后大概是下面这样的，确实只编译没打包，函数基本原样：

![clipboard.png][15]

本以为到这就结束了，但是这才开始。只编译不打包消除不必要的代码只是很小的原因，重要的东西我觉得换一行说比较好。
不顾语文老师的责骂换行，那什么才是是最重要的：***按需打包（tree shaking）***，对于这种组件和方法库，作为使用者，我们希望他能支持按需打包，像lodash和antd这样。所以怀着好奇的心理我去看了他们的package.json,然后发现了这样的配置：
```
  "main": "lib/index.js",
  "module": "es/index.js",
  "name": "antd",
```
除了认知中的main入口定义，还多了一个module入口.为什么需要这样呢，和我一样无知的，可以先读webpack官方的[tree shaking][6]，如果不够直观，可以再看一位大佬写的一篇相关文章[聊聊 package.json 文件中的 module 字段][7]。看下面代码：
```
// es6 模块写法 fun.js
export function square(x) {
  return x * x;
}

export function cube(x) {
  return x * x * x;
}

// commonJs写法 fun.js
exports.square = function(x) {
  return x * x;
}

exports.cube = function(x) {
  return x * x * x;
}

// index.js引入
import {
  square
} from './fun.js';

const res = square(10);
console.log('res:', res);
```
简单来说，就是index.js打包编译时，引入commonJs写法的fun.js,打包会将square与cube两个函数同时打进来。而引入es6写法的fun.js，只会将square打包。这样的操作，对于现在的主流趋势，就是必须的优化，特别对于lodash和antd这种庞大的库。而要使我们的库支持这样的操作，我们需要编译时，禁止babel将es6的module引入方式编译，其实只需要在前面的基础上"@babel/preset-react"多配置一个参数["@babel/preset-env", { "modules": false }] 
得到的是下面这样的结果：

![clipboard.png][16]

和上面的lib对比，感觉更接近原始代码。至此，编译已结束，但是我们还需要在package.json中加上相应的配置：
```
  "description": "antd后台项目前端组件封装和方法库",
  "main": "lib/index.js",
  "module": "es/index.js",
  "scripts": {
    "build": "webpack --config webpack.config.js",
    "lib": "gulp lib",
    "es": "gulp es",
    "prepublish": "gulp && webpack --config webpack.config.js"
  },
  "files": [
    "es",
    "dist",
    "lib",
    "utils"
  ],
```
### 怎么让库支持多目录输出 ###

因为我的库主要包括组件和方法，我把方法放到一起，通过utils作为默认输出。然后项目中引入是这样的

```
import { EnhanceTable, WithSearch }, utils from 'antd-doddle'; 
要用里面的方法需要再分解一次或通过utils.xxx
const { DATE_FORMAT, idCodeValid } = utils;
```
虽然感觉上不复杂，但是总感觉别扭，如果你用过dva，就见过下面这样的引入
```
import { routerRedux } from 'dva/router';
import dva from 'dva';
```

所以我去学习了一下，发现要这样实现也不难
分三步，分目录打包，增加一个输出，并增加内部私有映射，package.json增加一个这个映射目录的输出。具体可查看项目源码。实现后，项目引入是这样的
```
import { EnhanceTable, WithSearch }, utils from 'antd-doddle'; 
import { DATE_FORMAT, idCodeValid } from ‘antd-doddle/utils’; // 一步到位
```

### 小技巧分享 ###
 - npx执行本地命令
以前我们很多命令如webpack，gulp命令只有在全局安装（npm install xxx -g）才可以在命令行中直接运行或在项目中安装，通过script定义执行，但在npm5.2以后，我们可以只项目中安装，然后通过新增的npx执行。比如上面scripts中定义的lib打包（"lib": "gulp"），我们可以直接在命令行中用：
```
npx gulp
```
 - 命令行切换npm registry
有可能你和我一样，在到处都是墙的世界，需要在npm,cnpm,公司的npm registry三者之间来回切换，每次都需要这样：
```
npm set registry 'https://registry.npm.taobao.org/'
```

麻烦有没有? 幸好，这世界有很多牛逼的人，nrm registry是个很好用的工具，下面这样：  

```
// 安装
npm install -g nrm
// 设置入口npm，cnpm，company
nrm add npm 'http://registry.npmjs.org'
nrm add cnpm 'https://registry.npm.taobao.org'
nrm add vnpm 'http://npm.company.com'
// 切换入口到淘宝入口
nrm use cnpm
```

### 后续 ###
一个春节自己断断续续就在倒腾这个，收获还是挺大的。后面自己会慢慢去学习怎么加入demo‘，加入单元测试，去建造一个完整的npm库。
源码库：[github][8]
npm仓库地址：[npm][9]


  
 
[1]:http://closertb.site/2018/10/%E8%84%9A%E6%89%8B%E6%9E%B6%EF%BC%8C%E5%90%AC%E8%B5%B7%E6%9D%A5%E7%8E%84%E4%B9%8E%EF%BC%8C%E5%AE%9E%E9%99%85%E5%91%A2%EF%BC%9F/
  [2]: https://webpack.docschina.org/configuration/output/#output-librarytarget
  [3]: https://webpack.docschina.org/configuration/output/#output-librarytarget
  [4]: https://www.sitepoint.com/understanding-module-exports-exports-node-js/
  [5]: https://webpack.docschina.org/configuration/output/#output-librarytarget
  [6]: https://webpack.js.org/guides/tree-shaking/
  [7]: https://loveky.github.io/2018/02/26/tree-shaking-and-pkg.module/
  [8]: https://github.com/closertb/antd-doddle
  [9]: https://www.npmjs.com/package/antd-doddle
  [10]:https://image-static.segmentfault.com/106/119/106119203-5c64ce9d70bc7_articlex
  [11]:https://image-static.segmentfault.com/230/846/2308465126-5c63caabdc0db_articlex
  [12]:https://image-static.segmentfault.com/412/217/4122175979-5c63d4241813a_articlex
  [13]:https://image-static.segmentfault.com/371/148/3711489581-5c4c45d9e91b7_articlex
  [14]:https://image-static.segmentfault.com/875/524/875524389-5c4c68b399f58_articlex
  [15]:https://image-static.segmentfault.com/213/728/213728523-5c6d67fbc3ada_articlex
  [16]:https://image-static.segmentfault.com/200/411/2004119522-5c6d6eec45f26_articlex
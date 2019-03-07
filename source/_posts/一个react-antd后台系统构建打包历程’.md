---
title: 从24m到1m, 一个react+antd后台系统构建打包历程
date: 2019-03-07 21:54:13
tags:
---
虽然在工作中用react+antd写页面写了一年，但从来没自己去认认真真配置一个webpack，去分析去优化自己打出的包。在工程化成熟或者大点的公司，都有自己的打包工具，所以自己工作中很少去琢磨这些。为了试一下写出的组件库（[antd-doddle][1]）性能，就尝试自己去写一个webpack构建，真的是吓了自己一跳，流水账（tu）开始。
## 从npm run dev开始 ##
打包工具：webpack4 + babel6  

开始前，大概说一下项目内容。项目基于react+react-router+antd+antd-doddle,自己日常在这个项目做一些技术验证与demo,就4个页面。组成如下：  
```
  <Content style={{ margin: '24px 16px', padding: 24, background: '#fff', minHeight: 280 }}>
    <Switch>
      <Route exact path={menus.home.path} component={Home} />
      <Route exact path={menus.renderProps.path} component={ExampleTable} />
      <Route exact path={menus.hoc.path} component={Hoc} />
      <Route exact path={menus.learnTest.path} component={LearnTest} />
    </Switch>
  </Content>
```
页面js与公共（node_modules引用）js使用splitChunks分开打包。npm run dev，结果如下图，async.bundle.js大小24M，有点惊人的大。
![clipboard.png][3] 

打包工具：webpack4 + babel7  
![clipboard.png][4]
## 开始正题，npm run build ##
### development与production ###
由于webpack在（mode）模式development与production还是有很大区别，当我直接将mode从development变成production，再运行npm start。打包大小还是有明显的变化，同时也报了三个警告，包体积大于244kb

![clipboard.png][5]
打包大小从24M缩减到17M，由于run start是启用的webpack-dev-server来构建的，所以对应的production模式默认启用UglifyJsPlugin是无效的，但是可以手动去加入这段配置,然后打包大小能看到直线下降到3.25M，见下图
```
  optimization: {
    minimizer: [
      new UglifyJsPlugin()
    ],
    splitChunks: {
      name: 'async',
      minSize: 30000,
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          chunks: 'all'
        },
      }
    }
  }
```

![clipboard.png][6]   

### production模式构建 ###
以上其实都是在用webpack-dev-server构建，当用webpack去构建时，然后npm run build，结果是这样的：  

![clipboard.png][7] 
压缩后大小已经减小到2.6M，还是有实质的进步，然后通过BundleAnalyzerPlugin这个插件看包的构成，会发现antd，antd-doddle，moment占了绝大多数的空间。  

![clipboard.png][8]

恰好我知道moment虽然不支持按需打包，由于支持了多语言，所以省去无用的语言选项，就可以有明显的优化，所以使用ContextReplacementPlugin这个插件，并加上只匹配中文的筛选，如下所示：
```
    new webpack.ContextReplacementPlugin(
      /moment[/\\]locale$/,
      /zh-cn/,
    )
```  
打包体积由2.6减少到2.3M。
![clipboard.png][9]  

当一个一个浏览完包的构成时，大小组成是这样的：
 - atnd(609kb);
 - antd-icon(477kb);
 - antd-doddle(391kb)
 - dragt.js(160kb)
 - immutable(55*2kb)
 - moment(53 *2kb)
 - react-dom(100kb)
 - others(400kb)  
 
我留下了一些疑问：
 - 发现有两个moment.js,两个immutable.js,一个引用路径是正常的，一个是包里的相对路径。  

![clipboard.png][10]  

 - 然后还有一个draft.js（引用来源是antd），一个富文本编辑库，但我并没有这相关的组件应用，所以被打进来也是不讲道理的。
 - 然后看起来antd是按需引入（es module），但是我项目就4个页面，组件最多也就用到他所拥有的1/3，但是上面显示的285个，我的antd-doddle就更吓人了，居然有138个，我库中总共也就30来个导出，所以也是诡异。
为了减少大小，我还使用了babel-plugin-import插件，但没有实质性的改变，然后看到包的路径是es，说明现有的构建本来就已经做了tree-shaking,所以这确实是无用功。但公司项目和网上大多数文章提到的，都是打包都能优化到1M左右，所以应该还有优化的空间。后面我干了什么呢？
### 升级才是硬道理 ###
打包工具：webpack4 + babel7  

babel6升到babel7，修改一下配置。然后再运行npm run build, 结果如下：

![clipboard.png][11]   
当一个一个浏览完包的构成时，大小组成是这样的：
 - atnd(367kb);
 - antd-icon(475kb);
 - react-dom(102kb)
 - antd-doddle(66kb)
 - immutable(55kb)
 - moment(53kb)
 - others(218kb)
不能说是奇迹，但确实解决了一些上面留下的疑问：moment，immutable重复打包；draft无端引入；antd与antd-doddle包数激增。大小也减少了至少1M，所以升级才是自然界应有的规律，更不要说前端。  

![clipboard.png][12]   

## babel6到babel7，到底发生了什么 ##
看到这种优化的效果真的让我惊喜与惊讶，但究竟是什么带来了这种变化更让我更好奇。还没抓到核心的区别，有清楚的，请指点一下，怎么避免js公共函数的重复注入，怎么来优化编译来完美对接后面webpack打包的？想往外去看看大神们都什么意见，可特殊时期，我的梯子被折断了（衰）。

文章提到的项目地址：[webpack打包][2]，master分支是对应babel6，roadhg分支是对应babel7


  [1]: http://closertb.site/2019/02/%E4%BB%8Edist%E5%88%B0es%EF%BC%9A%E5%8F%91%E4%B8%80%E4%B8%AANPM%E5%BA%93%EF%BC%8C%E6%88%91%E8%9C%95%E4%BA%86%E4%B8%80%E5%B1%82%E7%9A%AE/
  [2]: https://github.com/closertb/PromoteMyReact
  [3]: https://image-static.segmentfault.com/227/302/2273026896-5c7b4c1cde854_articlex
  [4]: https://image-static.segmentfault.com/249/477/2494771122-5c7b3e660230b_articlex
  [5]: https://image-static.segmentfault.com/354/151/3541511419-5c7b4ccd32f3e_articlex
  [6]: https://image-static.segmentfault.com/630/864/630864661-5c7b5f5df0d7b_articlex
  [7]: https://image-static.segmentfault.com/840/586/840586160-5c7b614d3cf62_articlex
  [8]: https://image-static.segmentfault.com/332/960/3329601069-5c7bc906cbdfb_articlex
  [9]: https://image-static.segmentfault.com/143/521/1435214377-5c7bca053602b_articlex
  [10]: https://image-static.segmentfault.com/238/780/2387807211-5c7bced33ede6_articlex
  [11]: https://image-static.segmentfault.com/625/248/625248529-5c7bd6c622fe3_articlex
  [12]: https://image-static.segmentfault.com/201/197/201197044-5c7bf3dc81365_articlex
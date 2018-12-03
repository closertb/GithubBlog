---
title: 紧跟时尚，Fetch实践那些事
date: 2017-7-09 21:21:55
categories: '基础知识'
tags:
    - fetch
    - ajax
    - cors
---
今天看到关于[阿里前端面试的提问][1]，看到有一个兄弟说，阿里应该都用fecth了，怎么还在考ajax的底层实现，虽然以前[读ajax已死，fetch永生][2]文章时有了解这个知识，但闲着也是闲着，还是探索一下，才知道他好在那。
老规矩，看官方正式一点的文章，我还是推荐[MDN的][3]，读完个人理解,对比ajax，fetch的优势就是：
    语义化强，调用如ajax第三方库一样简洁；
    支持promise；
来看一个简单的调用：
```
   fetch('http://localhost:8089/StockAnalyse/BlogServlet', fetchInitOption({flag:'getList'}))
    .then(function(response){
        if(response.ok){  //避免404与500这样的响应
                return response.json();
        } else {
            console.error('服务器繁忙，请稍后再试；
Code:'   response.status)
        }
    }).then(function(data){
               that.item =data;//在promise中this指向已经变为指向window对象，所以提前用that保存了this；
               that = null;
            })
    });

```

如果你用过ajax第三方库，如jquery,vue-resource,axios这些，你会发现，fetch调用方法和这些库相似性非常之大，再看option的那些可设置属性：
-    method: GET/POST...
-    headers: 和其他header设置一样，主要设置Content-type和自定义header；
-     body: 要传递的数据
-     mode: cors / no-cors / same-origin，默认为 no-cors
-     credentials: omit / same-origin / include
-     cache: default / no-store / reload / no-cache / force-cache / only-if-cached
-     redirect: follow / error / manual
-     referrer: no-referrer / client /
-     referrerPolicy: no-referrer / no-referrer-when-downgrade / origin /  origin-when-cross-origin / unsafe-urlintegrity:

## 用fetch遇到的新鲜事: ##

1. 不管是404,还是500这些错误，请求仍然有response响应，所以才有response.ok状态值的判断;
2. 当我们使用第三方ajax库发送post请求，数据数据为js对象且设置Content-type为application/x-www-form-urlencoded，服务器都能正常响应(数据读取为request.getPrameter)；但Fetch这样设置就会导致服务器500错误，原因就在于Fetch它如AJAX一样，是一个底层的API，没有封装类似的数据转换,第三方库都自带,关于[post请求常用的Content-type][4]，所以为了不修改服务器端，我在配置post默认请求头时，对发送数据乃做了一定处理，不过仅适用于简单JS对象，目的就是将对象转化为键值对的方式，代码如下：

```
 function fetchInitOption(json){
       let res=new Array();
        for(let item in json){
            res.push(item '=' json[item])
        }
        return {
            method:'post',
            mode:'cors',
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded'
            },
            body:res.join('&')
        };
    }

```

3. 跨域mode的设置，如果项目涉及跨域，fetch在输入option配置项时做了支持，设置mode为cors，当然不跨越就需要设置成no-cors。如果跨域时，mode不为cors，其请求可以正常发出，如果后端做了跨域处理，其响应也可以正常处理，通过dectools看，其http响应完整,响应值200，也发回了想要的数据，但通过打印response的值，其状态为响应失败，这个原因就是由mode没有设置成cors造成的；也算一个奇葩现象；
4. this指向变化（这个不算fetch的坑，这是编程应注意的问题），因为我用了vue,实例中的this默认指向当前vue实例，但是当调用fetch这个方法在promise的响应的匿名函数里，this指向了window对象，所以这里需要提前用一个变量that来保持实例this的引用；
5. 请求前的拦截，就是在请求前想在header中加入自定义请求头,如TOKEN，不过好像解决思路一样,也可以在InitOption时手动设置；

虽路无尽头，但步伐坚定（早上一杯鸡汤，美好的周末即将开始）
  [1]: https://segmentfault.com/q/1010000009611466
  [2]: https://github.com/camsong/blog/issues/2
  [3]: https://developer.mozilla.org/zh-CN/docs/Web/API/GlobalFetch/fetch
  [4]: http://www.topjishu.com/6324.html
---
layout: 
title: fetch实践补充篇，文件上传content-type(multipart/form-data设置)
date: 2017-07-16 21:21:35
categories: '基础知识'
tags:
    - Content-type
    - 文件上传
    - fetch
---
**原谅我是一个标题党，其实这个坑和fetch关系不大**，归根结底是发送http响应的问题，闲话少说，直接说事。前面自己为了好玩，在自己的练手项目里，将vue-resource替换成原生JS的fetch API，使用效果还是极佳的，完全没有其他原生API那种晦涩感。

## 夯实基础 ##

前面一篇[文章fetch][1]已入过门，所以这只说重点，之前使用vue-resource和fetch时，在Conten-type设置上吃过不少亏，所以自己做了大量功课，重要的事情说三遍，post请求content-type,即数据请求的格式主要设置方式：

 - application/x-www-form-urlencoded（大多数请求可用：eg：'name=Denzel'）
 - multipart/form-data（文件上传，这次重点说）
 - application/json（json格式对象，eg：{'name':'Denzel','age':'18'}）
 - text/xml(现在用的很少了，发送xml格式文件或流,webservice请求用的较多)

## 问题描述 ##

我想通过fetch异步上传一张图片到服务器保存，然后返回服务的响应地址（需求很简单，有没有）。于是我这样写的代码：

    let data =new FormData();
    data.append('file',$('#realFile').files[0]);
    data.append('name','denzel'),
    data.append('flag','test')
    const option ={
                method:'post',
                mode:'cors',
                headers: {
                    'Content-Type': 'multipart/form-data'
                },
                body:data
    };
    fetch('http://localhost:8089/Analyse/imgUploadServlet',option)
    .then(function(response){
        if(response.ok){
            console.log('suc')
            return response.text();
        }else{
            console.log('网络错误，请稍后再试')
            return ;
        }
    }).then(function(data){
        console.log('imgUrl',data);
    })



 但在服务器上打印的是这样的错误信息（后端幸好用了try-catch，不然蹦一大堆错误，找不死你）：
 - 错误信息: the request was rejected because no multipart boundary was
   found


 很无厘头有没有，后端代码获取数据前，已经对请求的content-type做了检查，而且没有报错，那说明发送的是文件上传的请求，没毛病啊，而且这个上传文件的后端代码，以前在jsp页面中用过啊，没毛病啊，再在谷歌dev-tools查看一下请求：

```
		if (!ServletFileUpload.isMultipartContent(request)) {
		    // 如果不是则停止
		    PrintWriter writer = response.getWriter();
		    writer.println(Error: 表单必须包含 content-type=multipart/form-data);
		    writer.flush();
		    return;
		}

```


dev-tools请求信息：

```
    Access-Control-Allow-Methods:POST, GET, OPTIONS, DELETE
    Access-Control-Allow-Origin:*
    Content-Length:0
    Content-Type:text/html;charset=UTF-8
    Date:Sun, 16 Jul 2017 01:51:51 GMT
    Server:Apache-Coyote/1.1
    Request Headers
    view source
    Accept:*/*
    Accept-Encoding:gzip, deflate, br
    Accept-Language:zh-CN,zh;q=0.8,en;q=0.6
    Connection:keep-alive
    Content-Length:68172
    content-type:multipart/form-data
    Host:localhost:8089
    Origin:http://localhost
    Referer:http://localhost/
    User-Agent:Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36
    Request Payload
    ------WebKitFormBoundaryJ0rfRWvZ56LNpJ1U
    Content-Disposition: form-data; name='file'; filename='chn.PNG'
    Content-Type: image/png


    ------WebKitFormBoundaryJ0rfRWvZ56LNpJ1U
    Content-Disposition: form-data; name='name'

    denzel
    ------WebKitFormBoundaryJ0rfRWvZ56LNpJ1U
    Content-Disposition: form-data; name='flag'

    test
    ------WebKitFormBoundaryJ0rfRWvZ56LNpJ1U--

```

  好像都没问题,那no multipart boundary到底是个什么鬼，只有百度一下

## 问题症结   ##

原来不是我一个人遇到这样的问题，大家都引用的是这样一句话：

   ***you should never set that header yourself. We set the header properly    with the boundary. If you set that header, we won't and your server    won't know what boundary to expect (since it is added to the header).    Remove your custom Content-Type header and you'll be fine.***

翻译过来就是：

 *** 你不应该自己设置请求头（what ?）,我们会为请求头正确设置边界，但如果你设置了，我们和你的服务器都没法预知你的边界是什么（因为边界是被自动加到请求头的），删除你的自定义Content-Type请求头设置，问题将会解决（翻译渣，虽然英语六级飘过，但那已经是六年前；了）。***

好了，知道问题是啥了，删除请求头相关设置（Content-type），再发送，天啦，真的是耶，怎么会这样，不是说每个http请求都应该正确的设置自己的请求数据类型吗?我以前的书是不是白看了，啊,冷静，别人说的正确,那再通过dev-tools查看一下请求信息吧。

```
    Accept:*/*
    Accept-Encoding:gzip, deflate, br
    Accept-Language:zh-CN,zh;q=0.8,en;q=0.6
    Connection:keep-alive
    Content-Length:88623
    content-type:multipart/form-data; boundary=----WebKitFormBoundaryAnydWsQ1ajKuGoCd
    Host:localhost:8089
    Origin:http://localhost
    Referer:http://localhost/
    User-Agent:Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36
    Request Payload
    ------WebKitFormBoundaryAnydWsQ1ajKuGoCd
    Content-Disposition: form-data; name='file'; filename='Screenshot_2017-05-23-11-41-22-090_com.wacai365.png'
    Content-Type: image/png


    ------WebKitFormBoundaryAnydWsQ1ajKuGoCd
    Content-Disposition: form-data; name='name'

    denzel
    ------WebKitFormBoundaryAnydWsQ1ajKuGoCd
    Content-Disposition: form-data; name='flag'

    test
    ------WebKitFormBoundaryAnydWsQ1ajKuGoCd--

```

与上面失败的请求一比较，发现content-type后面居然多跟了boundary=----WebKitFormBoundaryAnydWsQ1ajKuGoCd这样一串火星字符,再看发送的数据，数据之间都被请求数据类型的那个boundary字符串分割开，好像，我是有点懂，什么叫边界了，就是发http请求规定的数据交换规则.类似于：A发送请求给B，并告诉B，我给你送来了三个快递（但为了好搬运，我将它捆成了一个包裹），包裹拆分的规则在快递单上有说明，于是B就按A说的规则，进行包裹拆分。
## 问题讨论：post请求Content-type到底该不该设置 ##
我得出的结论是，要正确设置。fetch 发送post字符类请求时，

 1. 非文件上传时，无关你发送的数据格式是application/x-www-form-urlencoded或者application/json格式数据，你不设置请求头，fetch会给你默认加上一个Content-type = text/xml类型的请求头，有些第三方JAX可以自己识别发送的数据，并自己转换，但feth绝对不会，不行，你可以试一下；
 2. 文件上传请求时，因为不知道那个boundary的定义方式，所以就如建议的一样，我们不设置Content-type。

如果本文有描述不正确的，欢迎指正，一起讨论，毕竟自己知识有限
 [1]: https://segmentfault.com/a/1190000010107288

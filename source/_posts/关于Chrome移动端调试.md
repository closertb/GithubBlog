---
title: 关于Chrome移动端调试
date: 2017-05-08 21:26:06
categories: '基础知识'
tags:
    - 前端移动端调试
---
关于怎么配置，网上有足够多的文章，但内容基本和下面推荐的这一篇相似，我也不知道谁是鼻祖。
[好文推荐][1]。


但文章中提到了：使用用DevTools*特别重要的一点是*：如果你点击inspect打开的DevTools窗口一片空白，且刷新无效时，那极有可能是由于被墙的缘故，你可以尝试appspot.com是否可以ping的通，如果无法ping通，那你现在就先翻墙吧（goagent 或红杏都不错，将appspot.com加入白名单），当然你还需要有google账户。


其实这一段说的，我怀疑不是解决的办法，因为我个人并没有翻墙，只将其添加到*白名单*就可以了。
添加方法：打开C:\Windows\System32\drivers\etc\文件夹
在hosts文件里加入：
61.91.161.217 chrome-devtools-frontend.appspot.com
61.91.161.217 chrometophone.appspot.com

仅以此文祭奠自己踩过的坑。。。。。


[1]:http://blog.csdn.net/freshlover/article/details/42528643
---

title: 修复Disqus评论功能关闭的问题：一个Python解决方案
categories:
- 编程人生
key: f49c35ea-1ac7-11ed-861d-0242ac120090


---



已经使用Disqus一段时间了，但最近我发现一些页面上的Disqus显示出"此话题的评论和回复功能已被关闭（Comments for this thread are now closed）"，这意味着无法新建帖子或进行回复。

在查阅Disqus的后台设置之后，发现这可能是因为开启了"30天后自动关闭论坛"的设置。

在Disqus后台的"Edit Discussions"界面中，可以找到相关的论坛并将其重新开启，然而，我遇到了一个问题：无法进行批量操作，只能一个讨论组一个讨论组点击鼠标开启。

为了解决这个问题，决定研究一下Disqus的相关API，并利用Python3编程来解决这个问题。

首先，需要获得API权限。为了达到这个目标，需要在Disqus中创建了一个应用程序，这样我就可以获取access_token。
  
#### 在Disqus中创建应用程序的方法：

>1. 登录Disqus账户，进入[https://disqus.com/api/applications/](https://disqus.com/api/applications/)。
>2. 在API页面，选择**Register new Application**。
>3. 提供应用信息，包括名称、描述、组织、网站和回调URL等。
>4. 完成应用创建。在创建后的页面，你将找到access_token，API Key和API Secret。

接下来，开始写python代码，调用API以显示被关闭的讨论区，并把它们重新打开。

#### 获得被关闭的thread列表

首先获得已经关闭的thread的列表，使用api调用 **listThreads**
```text
https://disqus.com/api/3.0/forums/listThreads?limit=100&include=closed&forum=论坛标识&api_key=你的api key
```

在返回的response中就包含了所有的被关闭的thread的列表，将其中的id保存下来，在后面用另外一个api将thread重新打开。

#### 将关闭的thread重新打开

api的调用地址：
```text
https://disqus.com/api/3.0/threads/open.json?access_token=你的access_token
```

对这个地址使用post方法调用，post的数据如下：
```json
{'thread': 板块id, "api_key": 你的api_key, 'access_token': 你的access_token}
```

这样就将被关闭板块重新打开了。

#### API调用限制

还有一点需要注意的是，Disqus的API调用限制是每小时最多1000次。因此，如果你的论坛数量很多，可能需要分多次完成这个操作。
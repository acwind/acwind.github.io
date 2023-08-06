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

在Disqus中创建应用程序的方法：
1. 登录Disqus账户，进入https://disqus.com/api/applications/。
2. 在API页面，选择**Register new Application**。
3. 提供应用信息，包括名称、描述、组织、网站和回调URL等。
4. 完成应用创建。在创建后的页面，你将找到access_token，API Key和API Secret。

接下来，开始写python代码，调用API以显示被关闭的讨论区，并把它们重新打开。

python代码如下

```python
import json
import requests
import sys

forum = "Disqus 站点标识"
api_key = "Disqus的API KEY"
access_token = "Disqus的Acess Token"

base_url = 'https://disqus.com/api/3.0/forums/listThreads?limit=100&include=closed&'
next = 'first'

def open_thread(id):
    url = 'https://disqus.com/api/3.0/threads/open.json?access_token=' + access_token
    data = {'thread': id, "api_key": api_key, 'access_token': access_token}
    response = requests.post(url, data=data)
    print(response.text)
    response = json.loads(response.text)
    if response['code'] != 0:
        print('您已经超过了每小时请求的限制，请过一小时后再来执行。')
        sys.exit(0)

while next:
    params = {
        'limit': 100,
        'include': 'closed',
        'forum': forum, 
        'api_key': api_key,
    }

    if (next != 'first'):
        params['cursor'] = next

    response = requests.get(url, params=params)
    response.raise_for_status()
    data = json.loads(response.text)
    next = data['cursor']['next']
    for response in data['response']:
        thread = response['id']
        print(thread)
        open_thread(thread)
```

还有一点需要注意的是，Disqus的API调用限制是每小时最多1000次。因此，如果你的论坛数量很多，可能需要分多次完成这个操作。



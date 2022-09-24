---

title: Laravel 分页url 静态化实践
categories:
- 代码人生
key: f49c35ea-1ac7-11ed-861d-0242ac120004

---

最近有个Laravel项目，为了提高访问速度，做了全站的静态化，参考了[《Laravel页面静态化最佳实践》](https://learnku.com/articles/51744)这篇文章配置代码和nginx。

但是遇到一个问题，Laravel框架自带的分页链接，采用的是url后缀带上page=xx这样的方式，采用上述方式做静态化不是太理想。于是需要另外的方案。

其实很简单，直接用笨方法，截取page=xx参数的值，将它保存成为xx.html文件，每次程序首先判断文件是否存在，存在则直接读取调用，不存在的话就渲染这个页面，显示并保存html。

```php
    $page = intval($request->input('page', 1));
        
    $path = 'pages/'.$page.'.html'; //分页缓存放到pages目录下
    if (Storage::disk('local')->exists($path)) {
        return Storage::disk('local')->get($path); //有缓存则显示缓存
    }
    
    //没有缓存的话就直接渲染到屏幕，并写入缓存
    $view = view('test');
    Storage::disk('local')->put($path, $view->render());
    return $view;
```



---

title: SB Admin 2 如何让侧边栏在屏幕缩小的时候隐藏
categories:
- 代码人生
key: f49c35ea-1ac7-11ed-861d-0242ac120025


---

我在使用开源的后台管理系统模板 sb-admin2 时发现，如果在手机上查看，侧边栏会缩小但不会消失，需要点击屏幕上的一个按钮（sidebarToggleTop）才能真正收起侧边栏。因此，我开始和 ChatGPT 结对编程，探讨如何让页面加载时判断屏幕大小，如果屏幕小于 768 像素，则自动隐藏侧边栏。

在 sb-admin2 中，sidebarToggleTop 组件是一个按钮，用于控制侧边栏的展开和折叠。默认情况下，该组件会触发一个名为 "sidebarToggle" 的自定义事件，在侧边栏的展开和折叠之间进行切换。

![](https://icdb-images.oss-cn-hangzhou.aliyuncs.com/news/2023/05/SCR-20230517-mx1.png)
<small>就是这个按钮</small>

为了实现自动隐藏侧边栏的功能，我直接模拟用户点击 sidebarToggleTop 按钮，从而触发该按钮所绑定的点击事件。具体步骤如下：

1. 获取 sidebarToggleTop 元素：

```javascript
const sidebarToggleTop = document.querySelector('#sidebarToggleTop');
```

2. 调用 click() 方法触发点击事件：

```javascript
sidebarToggleTop.click();
```

这样，执行完上述代码后，就会模拟用户点击 sidebarToggleTop 元素，触发 sb-admin2 中的 sidebarToggleTop 组件所绑定的点击事件。如果代码中监听了该事件，那么监听函数就会被触发，并执行相应的操作。

为了实现页面加载时自动隐藏侧边栏的功能，我在代码中监听了屏幕大小改变事件，整个代码如下：

```javascript
function handleSidenavToggle() {
  if (document.body.clientWidth < 768) {
    $('#sidebarToggleTop').click();
  }
}

$(window).on('load resize', handleSidenavToggle);
```

需要注意的是，在使用 click() 方法触发点击事件时，必须确保该元素已经被加载到页面上并且可见。如果该元素还没有被加载或者被隐藏了，那么 click() 方法可能会无效。最后，我喊 ChatGPT 把代码缩简到一行：

```javascript
$(window).on("load resize", function() {if (document.body.clientWidth < 768) $('#sidebarToggleTop').click();});
```
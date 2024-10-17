---

title: iOS Admob SDK 报错 Failed to look up symbolic reference at ...
categories:
- 代码人生
key: f49c35ea-1ac7-11ed-861d-0242ac120107


---

## 项目相关
今天在处理 iOS 项目时，对 cocoapods 进行了更新。之后编译运行项目，结果在 Google Admob SDK 中遇到了错误提示：

```
Failed to look up symbolic reference at 0x104fcde57 in xxxxxx
```

### 问题排查过程
花费了大量时间去排查问题，最终发现是因为 `Google-Mobile-Ads-SDK` 库更新到了最新版本（11.10.1）后，与 iOS 18 以及 Mac OS 新系统存在兼容性问题。

### 解决办法
解决方式比较简单，将其更换为低版本的 SDK，选择了 11.1.0 版本。在 `Podfile` 中进行如下修改：

```
pod 'Google-Mobile-Ads-SDK', '11.1.0'
```

然后重新执行 `pod install`，问题得以解决。等待Google的更新，应该会在不久后修复这个问题。

这个问题的解决过程记录在此，以便日后遇到类似问题时可以作为参考备忘。


---












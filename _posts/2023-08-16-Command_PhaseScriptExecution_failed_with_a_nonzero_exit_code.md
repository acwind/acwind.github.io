---

title: 解决Xcode 14.3升级后Archive时的Command PhaseScriptExecution错误
categories:
- 编程人生
key: f49c35ea-1ac7-11ed-861d-0242ac120092


---

最近，我遇到了一个在升级Xcode到14.3版本后的问题。具体来说，我有一个项目，在运行测试时一切正常，但在尝试进行archive打包时，却遇到了`Command PhaseScriptExecution failed with a nonzero exit code`错误。

这个问题主要表现在执行sh脚本中的rsync时，无法找到某些目录和文件。例如，我遇到的一个具体错误信息如下：

```bash
/IntermediateBuildFilesPath/UninstalledProducts/iphoneos/AFNetworking.framework" failed: No such file or directory
```

我尝试过重新卸载和安装库，但问题仍未得到解决。

## 寻找问题的根源

在Github上搜索这个问题后，我发现问题的根源在于我正在使用的cocoapods版本过低。这是因为，从Xcode 14.3开始，Apple在其框架的符号链接中开始使用相对路径，这导致了我以前的脚本在执行时报错。

在GitHub上，这个问题的原始描述是：

> Xcode 14.3 is now using a relative path in its symlink for frameworks. Without the -f flag, this relative path would be evaluated relative to the working directory of the script being executed, instead of relative to the framework symlink itself. With the -f flag, it resolves that relative path and returns the full path to the source.

![Command PhaseScriptExecution failed with a nonzero exit code](https://images.animesdata.com/news/2023/08/16/64dc459d5818f.png)

## 解决方法

解决这个问题的方法有两种。一种是修改引发错误的脚本文件，添加`-f`参数。另一种更简单的方法是将cocoapods升级到1.12版本或更高版本，因为cocoapods在这个版本中已经修复了这个问题。

你可以通过以下命令安装最新版本的cocoapods：

```bash
sudo gem install cocoapods
```

然后，运行`pod install`来更新你的库文件：

```bash
pod install
```

打完收工。
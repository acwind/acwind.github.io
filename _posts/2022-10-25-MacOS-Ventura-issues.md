---

title: 升级Mac OS Ventura 后 Bartender 权限的问题
categories:
- 代码人生
key: f49c35ea-1ac7-11ed-861d-0242ac120006

---
Mac OS  Ventura今日发布，当然是第一时间体验。经过半个小时的下载和更新，第一次启动后，还没有开始体验 Stage Manager，就已经出现了不少的问题。

首先是有一些软件不兼容，如旧版的little snitch，差点让整个机器无法上网。不少软件会提示不支持新的系统，需要更新安装新的版本。

首先出现的问题是在控制台中 ssh 无法登录服务器，解决的方法也比较简单，直接编辑~/.ssh/config文件，加入以下两段即可：

```
HostkeyAlgorithms +ssh-rsa 
PubkeyAcceptedAlgorithms +ssh-rsa
```

然后发现 Bartender 无法正常启动，即使更新到最新的版本也无济于事。表现为：启动时需要申请权限，一是Accessibility，一是Screen recording。但是即使分配了权限， Bartender还是无法检测到，一直在提示没有权限。

于是 Google 大法，在 Reddit 上看到有人给出了解决方案：重置系统的权限设置即可，方法是在控制台键入以下命令：

```
tccutil reset ScreenCapture
tccutil reset Accessibility
```
作用是重置这两个的权限设置，然后重启 Bartender，重新设置一下权限，即可。

这样的问题是，以前所有已经申请过这两个权限的 App，在启动时会需要重新设置一下。虽然有点麻烦，但好歹解决了问题，不是么？

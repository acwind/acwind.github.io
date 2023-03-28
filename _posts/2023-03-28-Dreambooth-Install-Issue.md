---

title: Stable Diffusion 安装 dreambooth 时 project_dir 的问题解决
categories:
- AI之光
key: f49c35ea-1ac7-11ed-861d-0242ac120020

---

![AI](https://cdn.discordapp.com/attachments/1081825402818920500/1089846695422066718/acwind_Advertisement_with_game-style_graphics_featuring_a_femal_6518ec2d-4411-4d5b-80df-b6742aa9d247.png)

近期AI作画真是杀疯了，用自Stable Diffusion画得不过瘾，前几天还充值了Midjourney的会员，就是为了感受一下v5版本作画的震撼。

公司为了方便美工设计，也配了一台高性能的主机加上3080Ti的显卡，几秒钟就出一张高清的图，甚爽。

而后需求又增加了，听说dreambooth训练模型非常厉害，于是捣鼓着安装它在SD的插件，安装和使用过程总是各种各样的问题，无法使用。比如如下的报错信息:


```bash
    Initializing dreambooth training...
    Exception initializing accelerator: Accelerator.__init__() got an unexpected keyword argument 'project_dir'
    提示：Python 运行时抛出了一个异常.请检查疑难解答页面.
    Restored system models.
    Duration: 00:00:13
```

Google了半天，在dreambooth的官方源代码的讨论区里发现了端倪，有个用户指出可能是Accelerator加速器的版本不对。

解决方法很简单，在Stable Diffusion的根目录下，编辑<strong>requirements_versions.txt</strong>文件，将accelerate的版本替换成你当前的版本即可，我这里改成0.17.1。

保存后重新启动webui，dreambooth就正常使用了。

如何查看自己的accelerate的版本呢，直接在命令行运行下列命令即可。

```bash
    pip list
```
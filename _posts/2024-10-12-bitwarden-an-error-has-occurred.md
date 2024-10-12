---

title: Bitwarden an error has occurred 错误的解决
categories:
- 代码人生
key: f49c35ea-1ac7-11ed-861d-0242ac120105


---

<img src="https://images.animesdata.com/news/2024/10/12/eu56rcpirlod1.jpeg" width="200" alt="an error has occurred">

Bitwarden 是一款优秀的密码管理工具，支持多设备使用，能帮助用户安全存储各类账号密码。其自建服务器方案可让用户自己掌控数据，增强隐私安全性。

我长期使用 Bitwarden 管理密码，通过自建服务器的方式来保管所有账号密码，它能在各种设备上使用，非常方便。

### 遇到的问题
前几日在 iOS 上升级 bitwarden 新版本时，出现了“an error has occurred”的提示无法进入。经查询发现很多人都有此问题，根源在于自建的 self-hosted 服务器版本过低，无法适配新客户端。

### 解决办法
我使用的是 `docker` 的 `vaultwarden/server` 服务器。

1. 更新Docker镜像:

   ```bash
   docker pull vaultwarden/server:latest
   ```

2. 停止并删除旧容器
3. 使用新镜像启动新容器

## 总结

记录此次经历,以备将来可能再次遇到类似问题时参考。定期更新自建服务器版本很重要,可以避免类似兼容性问题的发生。

## 相关链接

- [Bitwarden官网](https://bitwarden.com/)
- [Vaultwarden Docker Hub](https://hub.docker.com/r/vaultwarden/server)





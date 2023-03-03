---

title: x509 certificate has expired or is not yet valid
categories:
- 编程笔记
key: f49c35ea-1ac7-11ed-861d-0242ac120016

---

![Laptop](https://icdb-images.oss-cn-hangzhou.aliyuncs.com/other/blog/02594-1510005657.png)

最近在一台Linux上跑一个go的程序,一直提示"certificate has expired or is not yet valid is after 2021-09-30"这样的错误。

Google了老半天,大部分都说是服务器的时间问题,重置了同步了很多次时间,但是问题依旧.

后来想到这是一台很老的CentOS服务器,也许服务器本身的一些根证书过期了.搜索"证书 2021-09-30"关键字,果然发现了端倪:

> Let's Encrypt 品牌 SSL 证书根证书将于2021年9月30日停用旧版根证书（Root CA）。若您的网站已部署 Let's Encrypt 品牌的 SSL 证书并在过期前未及时更新，将导致您的网站面临不受计算机、设备或 Web 浏览器信任，网站兼容性降低，甚至部分网站不能访问的现象，将会影响您的使用。

所以需要更新一下老服务器的证书来解决这个问题,更新方法很简单,几行命令即可:

```bash
 yum install ca-certificates
 update-ca-trust force-enable
 update-ca-trust extract
```

搞定,go程序不再报错.打完收工,记录以备忘.




---
layout: post
title:  "始"
date:   2019-05-03 19:45:31 +0530
categories: GitHub
author: "easonyi"
---
### GitHub Pages是什么
> GitHub Pages is a static site hosting service designed to host your personal, organization, or project pages directly from a GitHub repository.You can create and publish GitHub Pages sites online using the Jekyll Theme Chooser. Or if you prefer to work locally, you can use GitHub Desktop or the command line.GitHub Pages is a static site hosting service and doesn't support server-side code such as, PHP, Ruby, or Python. [What is GitHub Pages?](https://help.github.com/en/articles/what-is-github-pages)

### 使用指南
* 注册GitHub账号
* create or fork一个具有静态博客特征的repository
* 修改repository name为 GitHub用户名.github.io
* 配置域名 or 直接通过 GitHub用户名.github.io访问

### 配置域名步骤
* 前往GitHub Repository Settings
![settings](..\pictures\settings.PNG)

* 修改Custom domain为你的域名
![domain](..\pictures\domain.PNG)

* 在域名管理页面添加记录
![记录](..\pictures\记录.PNG)

* 获取记录值

```shell
C:\Users\Administrator>ping easonyipj.github.io

正在 Ping easonyipj.github.io [185.199.111.153] 具有 32 字节的数据:
来自 185.199.111.153 的回复: 字节=32 时间=89ms TTL=51
来自 185.199.111.153 的回复: 字节=32 时间=96ms TTL=51
来自 185.199.111.153 的回复: 字节=32 时间=93ms TTL=51
来自 185.199.111.153 的回复: 字节=32 时间=79ms TTL=51

185.199.111.153 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 79ms，最长 = 96ms，平均 = 89ms
```



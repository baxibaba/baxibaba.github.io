---
layout: post
title: 群晖 NAS 设置 DDNS
categories: 群晖 NAS DDNS
description: 介绍 群晖 NAS 设置 DDNS 的方法
keywords: 群晖, NAS, DDNS, Synology
---
买了群晖 NAS 也有一段时间了，NAS 在局域网里使用着实方便，功能很强大。如果在互联网上也能访问，那就完美。想要让自己或者别人在外面可以通过互联网来访问 NAS，那么首先就得要知道 NAS 的公网 IP 地址。然而，在国内，一般家用的宽带是没有固定IP地址的，每次连接上网后该地址都会随机改变，这样想要让别人访问就变得非常麻烦。DDNS 就是为了解决这一问题而来！


## 什么是 DDNS ？

&emsp;DDNS 的全名叫做 Dynamic Domain Name Server  (动态域名服务)。它的工作原理大概是这样，用户首先申请一个DDNS的固定域名，每次客户端(这里客户端就是你的NAS)连接上网之后，便会将这次随机获得的IP地址发送到 DDNS 服务商的服务器去，服务器会将该IP与你的域名进行绑定，这样当其他人通过这个域名访问时，DDNS 便可将其解析到你的客户端的IP地址上去。如果客户端的IP地址有变更，那么它会再次告诉DDNS的服务器要求更新。这样用户只需记住DDNS的域名即可，完美替代了每次都会变化的IP地址。

## 群晖 NAS 怎么设置 DDNS ？
&emsp;群晖 NAS 的 DSM 系统默认就已经支持 DDNS 功能，并且群晖自己也提供了免费的 DDNS 服务，你可以注册一个 QuickConnect.to 的域名。当然，它还支持其他的 DDNS 服务提供商，譬如国内比较多人用的花生壳、3322.org 等等。这个服务商的稳定性、免费与否、更新速度等都不一样，大家根据自己喜好进行挑选。

&emsp;群晖官方的 DDNS 设置叫做 QuickConnect ，注册一个 Synology.Account 用户名，在 NAS 的“控制面板”启用“Enable QuickConnect”并填写自己的注册信息，如下图：

![ddns01](/images/posts/ddns/markdown-img-paste-20180117151439733.png)

在“高级”选项里勾选需要外部访问的服务，这个设置类似开启路由器的转发功能
![ddns02](/images/posts/ddns/markdown-img-paste-20180117152129300.png)

通过以上设置，我们就可以打开 http://QuickConnect.to/你的QuickConnectID 从外网来访问自己家里 NAS 上的各种应用了，非常的简单。

## DDNS 进阶，配置自有顶级域名实现公网访问
&emsp;在国内使用群晖免费的 DDNS 服务，设置虽然非常方便，但是速度不是很满意，从外网访问自己家里的 NAS 时，总是一卡一卡的，原因不得而知，可能是因为免费吧。刚好手里有万网购买的自有顶级域名 www.xxx.com 一个，琢磨着怎么弄个类似 nas.xxx.com 这样的子域名出来，给家里的 NAS 专用，关于使用自有顶级域名进行 DDNS 解析和访问的优点很多：

* 方便记忆地址和访问 NAS 服务器

* 比免费的稳定性好，访问速度快

* 不会有免费域名那种超长，很lower的二级域名

这么多的好处，有什么理由拒绝呢，Just do it !

1、查询顶级域名服务商有不有提供 DDNS 服务

网上查到万网其实不提供 DDNS 服务，为啥就不提供呢！亏我还注册购买了它家域名。

2、查询群晖 NAS DDNS 设置处支持哪些服务商提供的 DDNS 服务
当然，群晖的支持列表上也没有万网，我选择了一家 DNSPod （现在归企鹅公司旗下）
![ddns03](/images/posts/ddns/markdown-img-paste-20180117162445454.png)

3、注册 DNSPod 最好能实名认证
![ddns04](/images/posts/ddns/markdown-img-paste-20180117163018304.png)

4、注册成功后登陆，在控制台-域名解析处，添加你要使用的顶级域名，比如 qq.com
![ddns05](/images/posts/ddns/markdown-img-paste-20180117164149921.png)

5、添加域名后按 DNSPod 提供的 DNS 服务器地址，前往域名注册商处（我这里是万网）修改 DNS
![ddns06](/images/posts/ddns/markdown-img-paste-20180117164652122.png)

6、在 DNSPod 的控制台，选中前面添加的域名，添加记录然后如下图进行填写：
![ddns07](/images/posts/ddns/markdown-img-paste-20180117165210309.png)
注：主机记录是你需要绑定的域名，比如这里是 nas.qq.com 这个自定义设置；记录值填写 111.111.111.111 即可,因为后面会自动改，DDNS 修改的就是这个值。

7、生成 API Token,在 DNSPod 的控制台"用户中心"--“安全设置”--“API Token”，生成后要第一时间截图或者复制保存，名称可以自己定义
![ddns08](/images/posts/ddns/markdown-img-paste-20180117170206865.png)

8、前往群晖 NAS 打开“控制面板”--”外部访问”--DDNS--Add
![ddns09](/images/posts/ddns/markdown-img-paste-20180117170900551.png)

9、设置路由器端口转发，群晖 NAS 有提供便捷设置方式，很贴心，勾选相应服务就可以
![ddns10](/images/posts/ddns/markdown-img-paste-20180117171244818.png)

&emsp;到此，可以直接使用自有的顶级域名（形如nas.xxx.com）直接访问家里的nas了，请注意 : 虽是顶级域名，但访问时还要添加你的具体服务端口号的，比如访问 NAS 的 DSM 端口就是 5000

## DNSPod DDNS 解析原理
&emsp;他家的 API 接口允许用户直接通过 API 修改解析值， 群晖 NAS 会定时将公网 IP 推送给 DNSPod。 接着 DNSPod 开始解析域名并将其指向最新提交的 IP 地址，最终我们在访问时实际访问的就是最新提交的 IP。

## 问题

&emsp;基于以上原理，我们可能会碰到访问服务时，刚好碰到IP地址变化正处于解析中的情况，解析是需要时间的（一般五分钟以上吧），这期间我们访问域名会无法访问，一般等一下就好。当然我们还可以用群晖的 QuickConnect ，反正我是两种都设置了，互为备用 ☻。

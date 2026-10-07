---
title: VMISS US.LA.TRI.Basic 测评留档
published: 2026-10-07
description: 'VMISS 洛杉矶三网各自优化 US.LA.TRI.Basic 实测：配置、线路、性能、IP 质量与使用建议'
image: ''
tags: [服务器测评]
category: '服务器测评'
draft: false
lang: 'zh-CN'
---

## 关于 VMISS

VMISS[官网介绍](https://www.vmiss.com/about-us/)是一家 2021 年成立的加拿大技术与 IT 基础设施公司。业务线覆盖从vps到独立服务器，自我定位是"面向全球多个节点提供高性能云服务器、主机与计算服务"，主打的卖点是可预期的价格、易用的控制面板、99% 可用率和可弹性扩容的资源。

主营US/JP/HK/KR的优化线路/落地机器。不过对于个人用户来说，我认为**买 VMISS 值得看的是它的线路。**价格平价，性价比高，有以前经常有20%的常驻优惠券，活动也比较多。也有一部分有竞争力的产品。比如不久前扩容以后，本次测评的机器，US.LA.TRI.Basic的晚高峰带宽有显著提升。这也是这家在 MJJ 圈子里的口碑最好的机器。同时他们家的HK dc3是你可以买到的最便宜的hk cn2gia线路机器（最便宜的basic只需要20r左右一个月），虽然只有带宽100m，但是实际表现还不错，十分有竞争力。

全系列产品的带宽和流量都偏小，适合个人用户。

官网：<a href="https://app.vmiss.com/aff.php?aff=6059" target="_blank" rel="noopener noreferrer">VMISS 官网</a>

## 本文评测机型配置（US.LA.TRI.Basic）

| 配置项 | 详情 |
|--------|------|
| 购买链接 | <a href="https://app.vmiss.com/aff.php?aff=6059&pid=32" target="_blank" rel="noopener noreferrer">直达购买</a> |
| CPU | 1 核 |
| 内存 | 1 GB |
| 硬盘 | 10 GB SSD |
| 带宽/端口 | 200 Mbps |
| 流量 | 500 GB/月（双向计算） |
| IPv4 | 1 个独立 IPv4（另赠 IPv6） |
| 机房位置 | 美国洛杉矶 |
| 线路 | 电信 CN2 GIA / 联通 9929 / 移动 CMIN2 |
| 价格 | \$5.00 CAD/月（约 25 元人民币）；使用优惠码 `10%OFF` 享 9 折，折后 \$4.50 CAD/月（约 23 元人民币） |

### 机型介绍

US.LA.TRI 是 VMISS 在洛杉矶的三网各自优化线路系列，上游zont，电信走 CN2 GIA，联通走 9929，移动走 CMIN2。Basic 是这个系列最便宜的入门款，也是在mjj中最热门的，500G的流量，对于个人用户来说基本上也够用了，一个月不到20r，同时国际互联也不错，是不错的个人用户产品。

还有一个 DC2 机房版本（US.LA.TRI.DC2.Basic），上游kurun，价格相同，但月流量是 400 GB。

机器全天候三网延迟稳定，带宽在商家扩容以后单线程速度稳定，基本可以跑满200m，流畅使用没有什么问题。

## 性能测试

### IP 质量与解锁测试

<details>
<summary><strong>点击查看详细测试数据</strong></summary>

![IP质量测试](https://imgbed.cetaceang.qzz.io/file/测评留档/vmiss_la_tri/1791244346235_vmiss_la_tri_ip.png)

</details>

**总结**：IP 质量很干净的机房ip，甚至有些数据库中是家宽，几乎所有数据库都是低风险（只有 IPQS 这个敏感肌有点风险，这个库有点神经的，忽视就行），原生美国 IP。常见流媒体都能解锁，虽然不知道为什么个别解锁地区显示的hk，25 端口可用，需要的话可以尝试搭建邮局。

### 网络质量测试

<details>
<summary><strong>点击查看详细测试数据</strong></summary>

![网络质量测试](https://imgbed.cetaceang.qzz.io/file/测评留档/vmiss_la_tri/1791244776094_vmiss_la_tri_netquality.webp)

![三网回程路由](https://imgbed.cetaceang.qzz.io/file/测评留档/vmiss_la_tri/1791244937897_vmiss_la_tri_route.png)

**上海电信晚高峰 iperf3 实测**（单线程，10 秒）：

```text
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec  1.62 MBytes  13.6 Mbits/sec
[  5]   1.00-2.00   sec  22.9 MBytes   192 Mbits/sec
[  5]   2.00-3.00   sec  24.4 MBytes   204 Mbits/sec
[  5]   3.00-4.00   sec  25.5 MBytes   214 Mbits/sec
[  5]   4.00-5.00   sec  25.1 MBytes   211 Mbits/sec
[  5]   5.00-6.00   sec  25.1 MBytes   211 Mbits/sec
[  5]   6.00-7.00   sec  24.8 MBytes   208 Mbits/sec
[  5]   7.00-8.00   sec  26.2 MBytes   220 Mbits/sec
[  5]   8.00-9.00   sec  25.1 MBytes   211 Mbits/sec
[  5]   9.00-10.00  sec  25.9 MBytes   217 Mbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.13  sec   230 MBytes   190 Mbits/sec    0             sender
[  5]   0.00-10.00  sec   227 MBytes   190 Mbits/sec                  receiver

iperf Done.
```

</details>

**总结**：三网回程都走了各自的高端线路：电信cn2gia，联通9929，移动cmin2，nq脚本出错了，所有电信全识别成了163，但是看回程路由应该是没问题的，虽然不知道为什么广电走了9929。延迟比较稳定。晚高峰时段上海电信 iperf3 单线程稳定在 190~220 Mbps，0 重传，基本跑满 200m 端口。国际互联方面，到东京约 99ms，到纽约约 60ms，到欧洲约 130ms，总体还不错。

### 硬件性能测试

<details>
<summary><strong>点击查看详细测试数据</strong></summary>

![硬件性能测试](https://imgbed.cetaceang.qzz.io/file/测评留档/vmiss_la_tri/1791244982326_vmiss_la_tri_hardware.png)

</details>

**总结**：CPU Geekbench 5 单核 766、多核 785，对线路鸡来说性能不错了，日常跑轻量服务够用；内存 1 GB，硬盘 10 GB，4K 随机读写不算突出（Q1 约 27.7 / 23.5 MB/s），不适合太重的 IO 场景。其实可以跑数据库，不过这个磁盘大小又让跑数据库不太现实。

## 实际使用场景与体验

个人建议作为纯粹的组网机器使用，也可以跑一些轻量服务。

总体比较稳定。前面网络测试里的 iperf3 就是在**晚高峰**测的



## 总结：值得买吗？

> ✅ **推荐购买**：想要一台三网优化线路的线路鸡 / 组网机，不需要大带宽和流量，预算每月 $4.50~5.00 CAD 的个人用户
>
> ❌ **不推荐**：需要大流量、大带宽、大硬盘，或者要跑数据库、多个 Docker 等重负载的场景

---

*本文测评基于个人实际使用体验，仅供参考。不同地区、不同运营商的网络环境可能会有差异。*

<em>如果你也在关注 VPS 补货，可以查看 <a href="https://stock.cetaceang.de" target="_blank" rel="noopener noreferrer">VPS 库存监控</a>，或用 <a href="https://t.me/vpsstock_notice_bot" target="_blank" rel="noopener noreferrer">Telegram Bot</a> 订阅提醒。</em>

---
title: VPS 库存与补货监控：DMIT、VMISS 等实时库存 + Telegram 提醒
published: 2026-08-25
description: '集中查看 DMIT、VMISS、V.PS 等 VPS 厂商的套餐、地区、线路、配置、价格与库存，并通过 Telegram Bot 订阅补货提醒。'
image: ''
tags: [VPS]
category: '技术'
draft: false
pinned: true
lang: 'zh-CN'
---

买 VPS 时经常会遇到一种情况：看中的套餐长期缺货，偶尔补货又很快售罄。一直开着厂商页面手动刷新并不现实，同时关注多家厂商时更是麻烦。

所以我做了一个公开的 [VPS 库存监控站](https://stock.cetaceang.de)，把多家厂商的套餐和库存放在一起，并提供 Telegram 补货提醒。

## 目前可以做什么

- 集中查看多家 VPS 厂商的库存，无需逐个打开官网
- 查看套餐的地区、线路、CPU、内存、硬盘、流量、带宽和价格
- 按库存状态、地区等条件筛选套餐
- 有货时直接跳转到厂商的购买页面
- 通过 Telegram 精准订阅某一个套餐，或接收全部补货消息

监控任务通常约每分钟检查一次。实际更新时间可能受到厂商接口、页面变化或访问限制影响，最终库存和价格仍以厂商结算页面为准。

## 已接入的厂商

目前已经接入以下 7 家厂商：

| 厂商 | 库存页面 |
|------|----------|
| DMIT | [查看 DMIT 库存](https://stock.cetaceang.de/dmit) |
| SaltyFish | [查看 SaltyFish 库存](https://stock.cetaceang.de/saltyfish) |
| V.PS | [查看 V.PS 库存](https://stock.cetaceang.de/vps) |
| VMISS | [查看 VMISS 库存](https://stock.cetaceang.de/vmiss) |
| AaITR | [查看 AaITR 库存](https://stock.cetaceang.de/aaitr) |
| AkkoCloud | [查看 AkkoCloud 库存](https://stock.cetaceang.de/akkocloud) |
| RFCHOST | [查看 RFCHOST 库存](https://stock.cetaceang.de/rfchost) |

后续还会继续增加厂商，并根据实际使用体验完善筛选和提醒功能。

## 如何查找套餐

进入 [库存监控首页](https://stock.cetaceang.de) 后，先选择厂商，再按库存状态和地区进行筛选。套餐卡片会列出当前采集到的配置与价格；遇到合适的有货套餐，可以直接进入厂商购买页面。

## 用 Telegram 接收补货提醒

如果目标套餐暂时无货，可以通过两种方式接收通知：

- [精准订阅 Bot](https://t.me/vpsstock_notice_bot)：只订阅自己关注的具体套餐，减少无关消息
- [全量补货频道](https://t.me/vpsstock_channel)：接收监控范围内的全部补货消息

在库存站的套餐页面点击“Telegram 订阅下次补货”，也可以直接跳转到 Bot 并带上对应的套餐信息。对于可能很快售罄的低价套餐，这通常比偶尔手动检查更方便。

## 使用说明

库存信息来自自动监控，可能出现延迟、误判或厂商临时改变价格的情况，请以厂商最终展示和结算结果为准。部分“直达购买”链接可能包含推广标记，不会因此改变购买价格；如果不希望使用推广链接，也可以自行进入厂商官网购买。

网站和提醒功能仍在持续完善。如果有希望接入的厂商或更实用的筛选方式，欢迎通过博客中的联系方式告诉我。

<div align="center">

![quark-logo](img/icon.png)

# 智能夸克网盘自动转存

zp优化版本



**已优化内容总结（要点）**

- **智能任务列表**：新增 Smart Task List，支持只填剧名、自动搜索、自动保存。（避免选择的分享链接失效了、更新的比较慢，所以直接用搜索的结果）
- **自动搜索改进**：对搜索结果前 20 个分享解析最新集数，选集数最大的分享保存。
- **过滤条件**：新增“更新时间起始（updated_after）”与“最近几集（recent_episodes）”过滤。
- **执行环境**：WebUI 运行脚本改用当前 Python 解释器，避免依赖找不到。
- **性能**：并发解析分享链接时限制并发数+超时；把“前 20”改为“前 N 可配置”。
  - 智能搜索参数可配置：增加 Top N、并发数、超时的配置（UI + 配置文件 + 运行时读取），解析前 N 结果并发计算最新集数
  - 智能搜索并发限制：workers 默认改为 2，且最大限制 4（后端强制 + 前端输入限制）
  - 配置默认值同步：quark_config.json 更新默认值，WebUI 初始化默认值跟随变更
- **可观测性**：在日志里输出“最终选择的分享链接 + 最新集数 + 过滤前后文件数量”，排错更快。
- **WebUI**：为 Smart Task 增加“测试搜索”按钮，只跑搜索/解析不转存，也可以手动转存，用于那种自动保存错误的情况，比如老季/重制版。

**后续优化的方向：**

- **最新集判定**：对文件名解析加“季数/年份/分辨率过滤”与“分季优先”，避免老季/重制版干扰。
- **配置安全**：敏感字段（cookie、推送）在 WebUI 只显示脱敏/可切换显示。
- **智能搜索稳定性**：给搜索结果加“相似度评分 + 关键词过滤”，避免同名/误匹配；并缓存一次解析结果，减少重复请求。







夸克网盘签到、自动转存、命名整理、发推送提醒和刷新媒体库一条龙。

对于一些持续更新的资源，隔段时间去转存十分麻烦。

定期执行本脚本自动转存、文件名整理，配合 [SmartStrm](https://github.com/Cp0204/SmartStrm) / [OpenList](https://github.com/OpenListTeam/OpenList) , Emby 可达到自动追更的效果。🥳

> [!CAUTION]
> ⛔️⛔️⛔️ 注意！资源不会每时每刻更新，**严禁设定过高的定时运行频率！** 以免账号风控和给夸克服务器造成不必要的压力。雪山崩塌，每一片雪花都有责任！



## 功能

输入电视剧，电影，动漫名，自动搜索保存。

## 部署

### Docker 部署

未完成



管理地址：http://yourhost:5005

| 环境变量         | 默认       | 备注                                     |
| ---------------- | ---------- | ---------------------------------------- |
| `WEBUI_USERNAME` | `admin`    | 管理账号                                 |
| `WEBUI_PASSWORD` | `admin123` | 管理密码                                 |
| `PORT`           | `5005`     | 管理后台端口                             |
| `PLUGIN_FLAGS`   |            | 插件标志，如 `-emby,-aria2` 禁用某些插件 |
| `TASK_TIMEOUT`   | `1800`     | 任务执行超时时间（秒），超时则任务结束   |

<details open>
<summary>WebUI 预览</summary>

![screenshot_webui](img/screenshot_webui-1.png)

![screenshot_webui](img/screenshot_webui-2.png)

</details>

## 使用说明

### 正则处理示例

| pattern                                | replace                 | 效果                                                                   |
| -------------------------------------- | ----------------------- | ---------------------------------------------------------------------- |
| `.*`                                   |                         | 无脑转存所有文件，不整理                                               |
| `\.mp4$`                               |                         | 转存所有 `.mp4` 后缀的文件                                             |
| `^【电影TT】花好月圆(\d+)\.(mp4\|mkv)` | `\1.\2`                 | 【电影TT】花好月圆01.mp4 → 01.mp4<br>【电影TT】花好月圆02.mkv → 02.mkv |
| `^(\d+)\.mp4`                          | `S02E\1.mp4`            | 01.mp4 → S02E01.mp4<br>02.mp4 → S02E02.mp4                             |
| `$TV`                                  |                         | [魔法匹配](#魔法匹配)剧集文件                                          |
| `^(\d+)\.mp4`                          | `{TASKNAME}.S02E\1.mp4` | 01.mp4 → 任务名.S02E01.mp4                                             |

更多正则使用说明：[正则处理教程](https://github.com/Cp0204/quark-auto-save/wiki/正则处理教程)

> [!TIP]
>
> **魔法匹配和魔法变量**：在正则处理中，我们定义了一些“魔法匹配”模式，如果 表达式 的值以 $ 开头且 替换式 留空，程序将自动使用预设的正则表达式进行匹配和替换。
>
> 自 v0.6.0 开始，支持更多以 {} 包裹的我称之为“魔法变量”，可以更灵活地进行重命名。
>
> 更多说明请看[魔法匹配和魔法变量](https://github.com/Cp0204/quark-auto-save/wiki/魔法匹配和魔法变量)

### 刷新媒体库

在有新转存时，可触发完成相应功能，如自动刷新媒体库、生成 .strm 文件等。配置指南：[插件配置](https://github.com/Cp0204/quark-auto-save/wiki/插件配置)

媒体库模块以插件的方式的集成，如果你有兴趣请参考[插件开发指南](https://github.com/Cp0204/quark-auto-save/tree/main/plugins)。

### 更多使用技巧

请参考 Wiki ：[使用技巧集锦](https://github.com/Cp0204/quark-auto-save/wiki/使用技巧集锦)

## 生态项目

以下展示 QAS 生态项目，包括官方项目和第三方项目。

### 官方项目

* [QAS一键推送助手](https://greasyfork.org/zh-CN/scripts/533201-qas一键推送助手)

  油猴脚本，在夸克网盘分享页面添加推送到 QAS 的按钮

* [SmartStrm](https://github.com/Cp0204/SmartStrm)

  STRM 文件生成工具，用于转存后处理，媒体免下载入库播放。

### 第三方开源项目

> [!TIP]
>
> 以下第三方开源项目均由社区开发并保持开源，与 QAS 作者无直接关联。在部署到生产环境前，请自行评估相关风险。
>
> 如果您有新的项目没有在此列出，可以通过 Issues 提交。

* [nonebot-plugin-quark-autosave](https://github.com/fllesser/nonebot-plugin-quark-autosave)

  QAS Telegram 机器人，快速管理自动转存任务

* [Astrbot_plugin_quarksave](https://github.com/lm379/astrbot_plugin_quarksave)

  AstrBot 插件，调用 quark_auto_save 实现自动转存资源到夸克网盘

* [Telegram 媒体资源管理机器人](https://github.com/2beetle/tgbot)

  一个功能丰富的 Telegram 机器人，专注于媒体资源管理、Emby 集成、自动下载和夸克网盘资源管理。

## 打赏

如果这个项目让你受益，你可以无偿赠与我1块钱，让我知道开源有价值。谢谢！

![WeChatPay](https://cdn.jsdelivr.net/gh/Cp0204/Cp0204@main/img/wechat_pay_qrcode.png)

## 声明

本项目为个人兴趣开发，旨在通过程序自动化提高网盘使用效率。

程序没有任何破解行为，只是对于夸克已有的API进行封装，所有数据来自于夸克官方API；本人不对网盘内容负责、不对夸克官方API未来可能的变动导致的影响负责，请自行斟酌使用。

开源仅供学习与交流使用，未盈利也未授权商业使用，严禁用于非法用途。

## Sponsor

CDN acceleration and security protection for this project are sponsored by Tencent EdgeOne.

<a href="https://edgeone.ai/?from=github" target="_blank"><img title="Best Asian CDN, Edge, and Secure Solutions - Tencent EdgeOne" src="https://edgeone.ai/media/34fe3a45-492d-4ea4-ae5d-ea1087ca7b4b.png" width="300"></a>
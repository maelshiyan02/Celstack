# Celstack 对话开场白

> 用于 Trae 新会话的启动模板。直接复制粘贴即可开始对应开发。
> 本文件顶部的「参考项目与骨架」是每次新会话必读的固定上下文——避免模型重新摸索、重复造轮子。

---

# 零、参考项目与骨架（先读这段）

## 0.1 骨架选择：没有现成的可 fork，骨架就是 Scrollforge 的基建

**结论：Celstack 不存在可直接 fork 的成熟 Tauri 追番器，骨架 = 复制 Scrollforge 的 Tauri v2 + React + TypeScript + SQLite 基础工程，再叠播放器与追踪逻辑。**

理由：番剧追踪器领域最成熟的 `erengy/taiga` 是 C++ / Win32 原生程序，与 Tauri 技术栈不同，只能读它的功能设计，不能移植代码；而 `readest` 的 Tauri 工程写法可以直接借鉴（不复制代码）。

| 动作 | 内容 |
| --- | --- |
| 起点 | 从 Scrollforge 复制 Tauri v2 + React 工程骨架（换 app 名、标识符、图标） |
| 换装 | 图标用仓库根目录 `icon.png`；主色蓝紫 `#7365F0` |
| 播放器 | **不要用 WebView2 直接播 MKV**——Chromium 不支持 MKV 容器。走 libmpv 封装，或 FFmpeg 无重编码转封装成 MP4 |
| 数据层 | SQLite：`animes` / `episodes` / `progress` / `marks`（与 Scrollforge 的 db 封装保持同一套写法） |
| 排期 | Scrollforge 先做，Celstack 等它的 `db / ai / config` 跑通后再启动 |

## 0.2 参考项目清单（已核验 star 与许可证）

| 项目 | star | 语言 | 许可证 | 我们拿它做什么 |
| --- | --- | --- | --- | --- |
| `erengy/taiga` | 2.3k | C++ | GPL-3.0 | **追番追踪器的行为蓝本**（Windows 原生）：识别正在播放的播放器、自动记进度、同步 MAL/AniList。**只读设计，不抄代码** |
| `yjl9903/AnimeGarden` | 1.4k | TypeScript | AGPL-3.0 | 番剧资源与字幕组发布的聚合检索：资源发现层的列表与筛选交互参照 |
| `truelockmc/streambert` | 6.4k | JavaScript | GPL-3.0 | Electron 在线流播放器：播放页布局与源切换交互参照（与本地库场景相关度较低） |
| `readest/readest` | 25.7k | TypeScript | AGPL-3.0 | Tauri 端的窗口、文件系统、SQLite、设置面板写法参照 |
| `johnfactotum/foliate-js` | 1.1k | JavaScript | MIT | 不需要——列在此仅用于确认它支持 CBZ 漫画，为后续"电子漫画"分支留接口 |
| `mpv-player/mpv` | — | C | GPL-2.0+ | 播放内核（libmpv 嵌入）：MKV / HEVC / ASS 字幕的正确解 |
| `FFmpeg` | — | C | LGPL/GPL | 剪辑导出（无重编码切段）与容器转封装 |

外部 API（进度同步目标）：AniList GraphQL API v2、Bangumi 开放 API。二者都以 OAuth + 增量同步为设计前提，需要处理"本地改了 → 远端也改"的冲突。

## 0.3 许可证红线

- **只能读、不能抄**：`taiga`（GPL-3.0）、`AnimeGarden`（AGPL-3.0）、`streambert`（GPL-3.0）、`readest`（AGPL-3.0）。
- **可以引入**：`foliate-js`（MIT）、`readium/css`（BSD-3）。
- **尤其注意**：不能把 Scrollforge 里 fork 自 readest 的 AGPL 代码复制进 Celstack。共享层必须是两边各自重写的实现，或整体一起接受 AGPL。
- `mpv` 以 libmpv 动态链接方式使用属于常规做法；`FFmpeg` 若启用 GPL 组件（如 x264），整体分发需按 GPL 处理。

## 0.4 与 Scrollforge 的关系

1. 顺序：Scrollforge 优先，Celstack 排队。
2. 共享：`db` / `ai` / `config` 三个模块抽出共享库，两边引用。
3. 独立：播放器、剪辑、番剧元数据是 Celstack 独有，不进共享层。

---

## 模板 1：环境初始化（复用 Scrollforge 骨架）

```
启动 Celstack 项目开发。骨架不 fork 外部项目，直接从 Scrollforge 复制 Tauri v2 + React + Rust 工程：
1. 复制 Scrollforge 的 src-tauri 与前端工程，改 app 名 Celstack、标识符 com.celstack.app
2. 替换图标为仓库根目录 icon.png，主题主色 #7365F0
3. 只保留 db / ai / config 三个共享模块的引用，播放器与剪辑相关代码不要从 Scrollforge 带过来
4. 确认能编译、能打开空书架窗口

注意：如果 Scrollforge 走的是 fork readest 的路线，它的代码是 AGPL-3.0。Celstack 不要复制其中来自 readest 的部分，只复制我们自己在 Scrollforge 里新写的基建。
```

## 模板 2：Phase 1 功能开发（追番追踪器）

```
Celstack Phase 1：最小可用的追番追踪器。需求：
1. 番剧书架：手动添加番剧名 + 集数 + 封面（本地图片或 URL）
2. 本地视频导入（MP4/MKV/AAC+ASS 内封）——WebView2 不支持 MKV 容器，先做两条路：MP4 直接用 <video>；MKV 走 libmpv 或 FFmpeg 无重编码转封装（`-c copy`）成 MP4 再播
3. 进度追踪：记住集数 + 视频时间点（秒），下次打开自动恢复；退出与切换集时各存一次，避免丢进度
4. 剧集列表：每集状态（未看 / 在看 / 看完），封面用视频首帧或手动指定
5. 主题切换：主色 #7365F0（与图标一致），深色/浅色两套

SQLite Schema 需要：animes 表 + episodes 表 + progress 表（参考《Celstack项目开发计划书-追番追踪剪辑与AI增强规划.md》）

参考 erengy/taiga（GPL-3.0，只读设计不抄代码）：它的进度记录策略是"播放器一开就盯住窗口 + 定时轮询当前时长"，我们可以简化为"播放器内上报 + 关闭时落盘"。
```

## 模板 3：时间点标记（核心功能）

```
Celstack Phase 2：时间点标记功能。需求：
1. 播放视频时按 M 键标记当前时间点
2. 标记存入 SQLite：anime_id + episode + timestamp(秒) + note
3. 播放器下方显示标记时间点列表，点击跳转
4. 导出"精彩片段合集"：把所有标记过的时间段汇总显示
5. 快捷键可调（设置里改）
```

## 模板 4：AI 剧情摘要

```
Celstack Phase 3：AI 剧情摘要。需求：
1. 接入 AI API（复用 Scrollforge 的 ai 模块）
2. 自动生成每集剧情提要（从字幕或手动触发）
3. 角色档案：AI 识别出场角色 + 关系图谱
4. 经典台词收藏：标记的台词自动归档
5. "该看什么"推荐：基于你追过的番
```

## 模板 5：字幕搜索

```
Celstack：字幕功能。需求：
1. 支持导入 SRT / VTT 字幕文件
2. 字幕全文搜索（SQLite FTS5）
3. 视频播放时同步显示字幕
4. 选中字幕 → AI 翻译 → 显示译文
5. 时间点标记自动关联字幕内容
```

## 模板 6：Clip 剪辑导出

```
Celstack Phase 4：Clip 剪辑。需求：
1. 从标记的时间点自动提取片段：`ffmpeg -ss <start> -to <end> -i in -c copy out.mp4`（无重编码，秒出）
2. 注意 `-c copy` 的切点只能落在关键帧上，会前后偏移。需要帧级精确时，改用重编码（H.264 + AAC）或先做关键帧索引再决定策略
3. 片段列表页：缩略图（取片段首帧）+ 时长 + 预览 + 一键导出
4. 合并多个片段为一个合集：用 concat demuxer，前提是所有片段的编码参数一致（分辨率/帧率/编码器/时间基），否则先统一转码
5. 导出进度要能取消，长片段不要阻塞 UI 主线程

参考：`mpv` 用于片段预览的精确 seek；`FFmpeg` 仅在启用 GPL 组件（x264 等）时才需要按 GPL 分发，若只用 `-c copy` 不涉及重编码库。
```

## 模板 7：Anilist API 同步

```
Celstack：接入 AniList（GraphQL API v2）与 Bangumi（开放 API）。需求：
1. OAuth 登录：授权码流程，token 存本地（不要写进代码库），支持刷新
2. 追番列表双向同步：本地 ↔ AniList/Bangumi
3. 自动补全番剧信息：标题（多语言别名）、封面、集数、评分
4. 观看进度同步：集数 + 状态（在看/看完/搁置）
5. 冲突处理：每条记录存 `updated_at`，同步时以时间戳较新的一方为准；同秒冲突以本地为准并标记待复核
6. 请求限流保护：批量操作串行化，失败按指数退避重试

参考 erengy/taiga（GPL-3.0，只读设计）：它同时支持 MAL 与 AniList 双同步，字段映射表与"一次性全量拉取 + 增量推送"的策略值得照搬设计。
```

## 模板 8：项目启动脚手架检查

```
帮我快速审视 Celstack 当前目录结构，检查：
1. Tauri v2 布局是否完整
2. 和 Scrollforge 共用的 db / ai / config 模块是否正确引用
3. 前端播放器组件是否有独立的 Player/ 目录
4. SQLite Schema 是否有 marks 表（时间点标记专用）
5. 给出改进建议
```

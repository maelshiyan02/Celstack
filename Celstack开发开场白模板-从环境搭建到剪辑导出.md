# Celstack 对话开场白

> 用于 Trae 新会话的启动模板。直接复制粘贴即可开始对应开发。

---

## 模板 1：环境初始化（复用 Scrollforge Tauri 基建）

```
启动 Celstack 项目开发。复用 Scrollforge 的 Tauri v2 + React + Rust 基建——如果 Scrollforge 已经跑通了脚手架，直接复制过来改项目名和 Cargo.toml；如果还没有，就用官方脚手架初始化。Celstack 和 Scrollforge 共用 db / ai / config 模块。
```

## 模板 2：Phase 1 功能开发（追番追踪器）

```
Celstack Phase 1：最小可用的追番追踪器。需求：
1. 番剧书架（手动添加：番剧名 + 集数 + 封面 URL 或本地图片）
2. 本地视频导入（MP4/MKV），用 WebView2 <video> 播放
3. 进度追踪：记住集数 + 视频时间点 + 下次打开自动恢复
4. 剧集列表：显示每集状态（未看 / 在看 / 看完）
5. 主题切换（底色沿用图标的蓝紫主色 #5B4BE0）

SQLite Schema 需要：animes 表 + episodes 表 + progress 表（参考《Celstack项目开发计划书-追番追踪剪辑与AI增强规划.md》）
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
1. 从标记的时间点自动提取视频片段
2. FFmpeg 封装：输入视频 + 起止时间 → 输出 MP4
3. 片段列表页面，可预览 + 一键导出
4. 支持合并多个片段成一个合集
```

## 模板 7：Anilist API 同步

```
Celstack：接入 Anilist API。需求：
1. OAuth 登录 Anilist 账号
2. 追番列表双向同步（本地 ↔ Anilist）
3. 自动补全番剧信息（标题 / 封面 / 集数 / 评分）
4. 观看进度同步到 Anilist
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

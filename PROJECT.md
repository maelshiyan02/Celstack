# Celstack

> 动画追番器 · 每一集都是一张赛璐璐片

**官网**：待补充  
**GitHub**：https://github.com/maelshiyan02/Celstack  
**许可**：MIT  
**平台**：Windows 10/11（优先）→ macOS / Linux（后续）

---

## 项目简介

Celstack 是一款面向深度动画观看者的追番与标注工具。名字来自 **Cel**（赛璐璐片，动画逐帧绘制的透明胶片）+ **Stack**（堆叠）——每一帧 Cel 片堆叠起来就是一部动画，每一集追完就往上堆一层进度。和 Scrollforge 同属"加工系"产品：不只是追番，还要标注时间点、剪辑精彩片段、写观后感、AI 总结剧情。

### 核心理念

| 传统追番应用 | Celstack |
|--------------|----------|
| 标记"看到第X集" | 标记"看到第X分X秒" |
| 看完就走 | 边看边标注精彩时间点 |
| 忘了哪段最好看 | 自动生成精彩片段合集 |
| 不知道讲了什么 | AI 实时剧情摘要 |
| 无感知 | 像堆赛璐璐片一样有仪式感 |

### 目标用户

- 一次追好几部番、容易混乱进度的动画爱好者
- 想回头看"那段最精彩"但记不清集数的重番观众
- 想做反应视频/剪辑 UP 主的创作者
- 想写观后感但懒得记时间点的资深观众

---

## 技术栈

| 层 | 选择 | 理由 |
|----|------|------|
| **桌面壳** | Tauri v2 | 和 Scrollforge 统一，开发栈复用 |
| **前端** | React 19 + Vite | 和 Scrollforge 统一 |
| **后端** | Rust | 和 Scrollforge 统一 |
| **视频播放** | WebView2 原生 `<video>` + 扩展 | 标准 HTML5，无需引入 VLC 等重型库 |
| **数据库** | SQLite (rusqlite) | 统一 |
| **时间点标记** | 自定义格式（集数 + timestamp + 备注） | 轻量化 |
| **AI 后端** | 复用 Scrollforge 的 AI API 桥接 | 统一 |

### 和 Scrollforge 的关系

| 项目 | 共用 | 独立 |
|------|------|------|
| Tauri v2 壳 | ✅ | - |
| React + Vite | ✅ | - |
| SQLite 基建 | ✅ | - |
| AI API 桥接 | ✅ | - |
| 播放引擎 | - | ✅（WebView2 video） |
| 时间点标记系统 | - | ✅ |
| 剪辑导出 | - | ✅ |
| 格式解析（EPUB/TXT） | ✅（未来可能用到） | - |

---

## 功能路线图

### Phase 1 — 追番追踪器（2-3 周）

| # | 功能 | 复杂度 |
|---|------|--------|
| 1 | 番剧书架（手动添加 + 封面 + 集数信息） | 低 |
| 2 | 观看进度追踪（集数 + 时间点） | 低 |
| 3 | 本地视频导入（MP4/MKV） | 低 |
| 4 | 基础播放器（播放/暂停/快进/倍速） | 低 |
| 5 | 剧集列表 + 下一集提醒 | 低 |
| 6 | 主题切换 | 低 |

### Phase 2 — 标注与剪辑（3-4 周）

| # | 功能 | 复杂度 |
|---|------|--------|
| 7 | **时间点标记**（播放时按快捷键标记 + 备注） | 中 |
| 8 | **精彩片段合集**（自动提取你标记过的时间段） | 中 |
| 9 | 观后感笔记（和番剧绑定，按集索引） | 低 |
| 10 | 字幕搜索（SRT/VTT 导入 + 全文搜索） | 中 |
| 11 | 倍速播放 / A-B 循环 | 低 |

### Phase 3 — AI 增强（3-4 周）

| # | 功能 | 复杂度 |
|---|------|--------|
| 12 | **AI 剧情摘要**（每集自动生成剧情提要） | 中 |
| 13 | **AI 角色档案**（自动识别角色出场 + 关系） | 高 |
| 14 | **对话翻译**（选中字幕 → AI 翻译） | 中 |
| 15 | 经典台词收藏 | 低 |
| 16 | "该看什么"推荐（基于你追过的番） | 高 |

### Phase 4 — 高级（按需）

| # | 功能 |
|---|------|
| 17 | Clip 剪辑导出（从时间点标记自动剪片段） |
| 18 | 联网追番（Anilist / AniList API 同步） |
| 19 | 实时观看（通过 WebView 聚合平台） |
| 20 | 和 Scrollforge 统一账号 + 云同步 |

---

## 技术细节备忘

### 时间点标记数据结构（SQLite）

```sql
CREATE TABLE marks (
  id INTEGER PRIMARY KEY,
  anime_id INTEGER NOT NULL,       -- 关联的番剧
  episode INTEGER NOT NULL,        -- 第几集
  timestamp INTEGER NOT NULL,      -- 秒数
  note TEXT,                        -- 备注
  created_at DATETIME DEFAULT NOW()
);

CREATE INDEX idx_marks_anime ON marks(anime_id, episode);
```

### 进度记忆（关键需求）

```typescript
// 存集数 + 时间点 + 记住 window position
{
  animeId: 123,
  episode: 8,
  timestamp: 142.5,
  windowX: 100, windowY: 100,
  windowW: 1280, windowH: 800
}
// 下次打开自动恢复到这个位置和时间点
```

### WebView2 video 播放器扩展

```javascript
// 监听 video 元素的 timeupdate 事件
const video = document.querySelector('video');
video.addEventListener('timeupdate', () => {
  // 每秒记录一次当前时间点
  currentTime = video.currentTime;
});
// 快捷键标记
document.addEventListener('keydown', (e) => {
  if (e.key === 'm' || e.key === 'M') {
    invoke('mark_timestamp', { animeId, episode, time: video.currentTime });
  }
});
```

---

## 项目结构

```
Celstack/
├── Cargo.toml
├── src-tauri/
│   ├── src/
│   │   ├── main.rs
│   │   ├── db/           # 番剧/标记/笔记 SQLite 操作
│   │   ├── mark/         # 时间点标记逻辑
│   │   └── ai/           # AI 桥接（和 Scrollforge 共用）
│   └── tauri.conf.json
├── src/
│   ├── components/
│   │   ├── Shelf/        # 番剧书架
│   │   ├── Player/       # 播放器（核心）
│   │   ├── Marker/       # 时间点标记面板
│   │   ├── Notes/        # 观后感笔记
│   │   └── Forge/        # AI 摘要/翻译面板
│   └── App.tsx
├── package.json
└── README.md
```

---

## 命名溯源

**Celstack** = Cel（赛璐璐片）+ Stack（堆叠）

- Cel 是动画行业行话，指传统手绘动画里的一张张透明胶片——懂的人秒懂这个项目是干什么的
- Stack 有两层含义：(1) 每一帧 Cel 片堆叠成一部动画；(2) 你追的番剧像堆栈一样一层一层往上堆
- 和 Scrollforge 的 Forge（锻造炉）呼应，都是"加工系"词根：Scrollforge 锻造文本，Celstack 堆叠进度
- Cel 还是 **C**ontent **E**xtension **L**ayer 的缩写（多媒体扩展层），暗示未来可能扩展到漫画/动画统一管理

---

*文档版本：v1.0 · 2026-10-27*

# Mineradio 文档索引

## 快速导航

### 首次接触项目？按顺序阅读：

1. [`AGENTS.md`](../AGENTS.md) — AI 工作规范，项目入口（第一优先）
2. [`docs/architecture/ARCHITECTURE.md`](architecture/ARCHITECTURE.md) — 系统架构全貌
3. [`docs/history/PROJECT_MEMORY.md`](history/PROJECT_MEMORY.md) — 项目长期记忆与设计偏好
4. [`docs/roadmap/ROADMAP.md`](roadmap/ROADMAP.md) — 路线图与待办事项

### 按场景查找：

| 场景 | 文档 |
|------|------|
| 修改 UI/视觉 | `design/GLASS_SVG_TEXTURE.md` → `design/DESKTOP_LYRICS_VISUAL.md` |
| 修改 3D 歌单架 | `design/3D_PLAYLIST_SHELF_MEMORY.md` |
| 修改播放/登录 | `design/QQ_MUSIC_INTERFACE_NOTES.md` |
| 修改安装器 | `design/INSTALLER_STYLE.md` |
| 发布新版本 | `../RELEASE.md` → `release/RELEASE_NOTES_v1.1.0.md` |
| 安全审计 | `security/SECURITY_REBUILD_2026-06-24.md` |
| 上手交接 | `history/HANDOFF_NEXT_CHAT.md` → `history/AI_HANDOFF.md` |

---

## 目录结构

```
docs/
├── README.md                     # 本文件 — 文档索引
│
├── architecture/                 # 系统架构
│   └── ARCHITECTURE.md           #   完整的系统架构文档
│
├── design/                       # 设计规范与专项记忆
│   ├── GLASS_SVG_TEXTURE.md      #   SVG 玻璃质感黄金版本
│   ├── DESKTOP_LYRICS_VISUAL.md  #   桌面歌词视觉效果规范
│   ├── 3D_PLAYLIST_SHELF_MEMORY.md # 3D 歌单架交互手感边界
│   ├── INSTALLER_STYLE.md        #   安装器中文极简黑白蓝格式
│   └── QQ_MUSIC_INTERFACE_NOTES.md # QQ 音乐接口排障记录
│
├── roadmap/                      # 路线图
│   └── ROADMAP.md                #   版本规划与待办事项
│
├── release/                      # 发布说明
│   └── RELEASE_NOTES_v1.1.0.md   #   v1.1.0 发布说明
│
├── history/                      # 项目记忆与交接
│   ├── PROJECT_MEMORY.md         #   长期设计偏好与关键决策
│   ├── HANDOFF_NEXT_CHAT.md      #   新对话快速交接
│   └── AI_HANDOFF.md             #   AI 工作交接（含完整工作日志）
│
├── security/                     # 安全
│   └── SECURITY_REBUILD_2026-06-24.md # 安全重建记录
│
├── SUPPORT.md                    # 作者支持渠道
│
└── assets/                       # 文档图片资源
    ├── readme/
    │   └── cinema-beat-smoke.png
    └── support/
        └── mineradio-author-support-poster.png
```

---

## 写作约定

- **`docs/design/`** — 视觉规范、交互边界、专项技术排障。记录"怎么做"和"为什么这样做"。
- **`docs/roadmap/`** — 未来规划、待办事项。记录"接下来做什么"。
- **`docs/release/`** — 版本发布说明。一个版本一个文件。
- **`docs/history/`** — 项目记忆与工作日志。追加式更新，不删除旧内容。
- **`docs/security/`** — 安全事件与审计记录。
- **`docs/architecture/`** — 系统架构，反映当前代码结构，随代码变更同步更新。

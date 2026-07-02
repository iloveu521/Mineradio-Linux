# Mineradio Linux 文档索引

## 快速导航

### 首次接触项目？按顺序阅读：

1. [`CLAUDE.md`](../CLAUDE.md) — AI 工作规范，项目入口（第一优先）
2. [`docs/roadmap/ROADMAP.md`](roadmap/ROADMAP.md) — 当前 Sprint 与路线图
3. [`docs/design/DESIGN.md`](design/DESIGN.md) — 当前阶段详细设计文档
4. [`docs/architecture/ARCHITECTURE.md`](architecture/ARCHITECTURE.md) — 系统架构全貌
5. [`docs/history/PROJECT_MEMORY.md`](history/PROJECT_MEMORY.md) — 项目长期记忆与设计偏好

### 按场景查找：

| 场景 | 文档 |
|------|------|
| 当前开发任务 | `design/DESIGN.md` → `roadmap/ROADMAP.md` |
| 修改 UI/视觉 | `design/GLASS_SVG_TEXTURE.md` → `design/DESKTOP_LYRICS_VISUAL.md` |
| 修改 3D 歌单架 | `design/3D_PLAYLIST_SHELF_MEMORY.md` |
| 修改 QQ 音乐接口 | `design/QQ_MUSIC_INTERFACE_NOTES.md` |
| 发布新版本 | `../CHANGELOG.md` → `../RELEASE.md` |
| 上手交接 | `history/AI_HANDOFF.md` → `history/PROJECT_MEMORY.md` |

## 目录结构

```
docs/
├── README.md                            # 本文件 — 文档索引
├── architecture/
│   └── ARCHITECTURE.md                  # 完整系统架构文档
├── design/
│   ├── DESIGN.md                        # 当前阶段设计文档
│   ├── GLASS_SVG_TEXTURE.md             # SVG 玻璃质感黄金版本
│   ├── DESKTOP_LYRICS_VISUAL.md         # 桌面歌词视觉效果规范
│   ├── 3D_PLAYLIST_SHELF_MEMORY.md      # 3D 歌单架交互手感边界
│   └── QQ_MUSIC_INTERFACE_NOTES.md      # QQ 音乐接口排障记录
├── roadmap/
│   └── ROADMAP.md                       # 当前 Sprint + 版本规划
├── history/
│   ├── PROJECT_MEMORY.md                # 长期设计偏好与关键决策
│   └── AI_HANDOFF.md                    # AI 工作交接（含完整工作日志）
└── assets/                              # 文档图片资源
```

## 写作约定

- **`docs/design/DESIGN.md`** — 当前阶段详细设计（Why + How），代码实现前必读
- **`docs/roadmap/ROADMAP.md`** — Current Sprint 进度跟踪，每完成一个 Phase 更新状态
- **`docs/history/PROJECT_MEMORY.md`** — 项目长期记忆，追加式更新
- **`docs/history/AI_HANDOFF.md`** — AI 工作交接，每次任务完成后更新

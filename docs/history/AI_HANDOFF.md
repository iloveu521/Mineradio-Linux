# Mineradio Linux AI Handoff

新对话接管本工作区时，优先读 `docs/README.md`、`docs/roadmap/ROADMAP.md`、`docs/design/LINUX_DESIGN.md`。

## 当前权威入口（2026-07-02）

- 当前项目：Mineradio Linux，独立仓库（非 fork）
- GitHub 仓库：`https://github.com/iloveu521/Mineradio-Linux`
- 当前版本：`v1.2.1`
- Release 地址：`https://github.com/iloveu521/Mineradio-Linux/releases/tag/v1.2.1`
- 上游仓库：`git@github.com:XxHuberrr/Mineradio.git`（已添加为 upstream remote）
- 当前阶段：短期优化计划 v1.3.0（6 个 Phase），详见 `docs/roadmap/ROADMAP.md`

## 用户偏好

- 默认用中文沟通
- 视觉方向：暗色、玻璃质感、舞台感、音乐可视化
- 禁止大块重写 `public/index.html` 视觉系统，用 `rg` 精确定位
- 不要破坏已有的 SVG 玻璃质感、3D 歌单架手感、粒子预设

## 工作区地图

- `desktop/main.js`：Electron 主进程、窗口管理、IPC
- `server.js`：HTTP API 服务、网易云/QQ 音乐代理、更新检查
- `public/index.html`：前端 SPA（~27000 行），修改前用 `rg` 定位
- `dj-analyzer.js`：离线节拍分析
- `public/desktop-lyrics.html`：桌面歌词叠加窗口
- `public/wallpaper.html`：壁纸窗口
- `build/`：图标资源
- `docs/`：项目文档

## 已完成工作日志

### 2026-07-02 — 记录者：Silly ([iloveu521](https://github.com/iloveu521))

- Linux 迁移完成（v0.2.0 → v1.2.1），共 7 个阶段。
- 发布 v1.2.1 Release：AppImage (135MB) + deb (105MB)。
- 仓库脱离 fork → 独立仓库，已添加上游 remote。
- 制定短期优化计划 v1.3.0（6 个 Phase），写入 `docs/roadmap/ROADMAP.md`。
- 清理 Windows 历史记录，Mineradio Linux 作为独立发行版维护。

### 2026-07-01 — 记录者：Silly ([iloveu521](https://github.com/iloveu521))

- `docs/` 从扁平结构重组为分类子目录：architecture/design/roadmap/release/history/security。
- 11 个文件通过 `git mv` 移入对应子目录，零删除，保留 Git 历史。
- 新建 `docs/README.md`（文档索引导航）和 `docs/roadmap/ROADMAP.md`（路线图）。
- 新建 `docs/architecture/ARCHITECTURE.md`（完整系统架构文档）。
- 更新 40+ 处跨文档路径引用。

## 未完成/待确认事项

- 短期优化计划 v1.3.0（6 个 Phase），详见 `docs/roadmap/ROADMAP.md`
- Wayland 完全兼容支持
- 桌面歌词中键锁定替代方案

## 每次任务完成后的固定动作

1. 更新本文件的「已完成工作日志」
2. 更新 `docs/roadmap/ROADMAP.md` 的 current sprint
3. 更新 `CHANGELOG.md`
4. 运行 `node --check server.js` 和 `node --check desktop/main.js`
5. 创建 Git commit + tag

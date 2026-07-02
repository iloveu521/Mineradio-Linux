# Mineradio Linux Project Memory

Mineradio Linux 独立发行版的项目长期记忆。记录者：Silly ([iloveu521](https://github.com/iloveu521))。

## Stable Project Facts

- 项目名称：Mineradio Linux
- GitHub 仓库：`https://github.com/iloveu521/Mineradio-Linux`
- 上游参考：`git@github.com:XxHuberrr/Mineradio.git`（已添加为 upstream remote）
- 当前版本：`v1.2.1`
- 构建产物：AppImage (135MB) + deb (105MB)
- 基于原始 Mineradio v1.1.1，GPL-3.0 授权

## Memory Entries

### 2026-07-02 — Release v1.2.1 + 独立仓库 + 短期优化计划

- **记录者**：Silly
- **GitHub**：[iloveu521](https://github.com/iloveu521)
- **Release v1.2.1**：
  - 已发布到 GitHub Releases：AppImage (135MB) + deb (105MB)
  - Release 地址：https://github.com/iloveu521/Mineradio-Linux/releases/tag/v1.2.1
- **独立仓库**：
  - 仓库脱离 fork → 独立仓库（便于 GitHub 搜索）
  - 已添加上游 remote：`git@github.com:XxHuberrr/Mineradio.git`
- **短期优化计划**（v1.3.0，6 个 phase）：
  1. UI Linux 桌面环境兼容优化（X11/Wayland 检测、卡顿排查）
  2. 左侧歌单交互优化（卡顿、消失延迟）
  3. 桌面歌词显示逻辑优化（窗口可见时关闭、最小化时显示）
  4. 音量按钮滚轮调节
  5. 进度条拖动响应优化
  6. 播放器控制台桌面歌词开关按钮
- **禁止回退**：不要回退到 fork 模式；不要恢复 Windows 专用配置；不要改回 D:\ 路径

### 2026-07-02 — Linux 平台迁移完成

- **记录者**：Silly
- **完成内容**：Mineradio 从 Windows 迁移到 Linux，共 7 个阶段 (v0.1.0 → v1.2.1)
- **代码变更**：
  - `desktop/main.js` — GPU D3D11 条件化 + PNG 图标适配
  - `server.js` — 节拍缓存 XDG 路径 + beatCacheRootInfo 重写
  - `package.json` — Linux 专用配置，AppImage + deb 构建
- **文档新建**：
  - `docs/architecture/ARCHITECTURE.md` — 完整系统架构
  - `docs/design/LINUX_DESIGN.md` — Linux 迁移设计文档
  - `docs/roadmap/ROADMAP.md` — 项目路线图
  - `docs/README.md` — 文档索引导航
  - `README_EN.md` — 英文版项目说明
- **已知限制**：Wayland 不完全兼容、桌面歌词中键锁定不可用、壁纸无 WorkerW、无自动更新
- **禁止回退**：不要恢复 Windows 专用配置覆盖 Linux 适配；不要改回硬编码 D:\ 路径

### 2026-07-01 — 文档目录结构重组

- **记录者**：Silly
- **归档内容**：将 `docs/` 从扁平结构重组为分类子目录（architecture/design/roadmap/release/history/security）
- **新目录结构**：`docs/architecture/` `docs/design/` `docs/roadmap/` `docs/release/` `docs/history/` `docs/security/` `docs/assets/`
- **关键参数**：全部文件移动使用 `git mv` 保留历史；40+ 处跨文档路径引用已同步更新
- **禁止回退**：不要恢复扁平结构

## How To Add New Memory

追加格式：
```markdown
### YYYY-MM-DD — 简短标题

- **记录者**：名字
- **内容**：...
- **涉及文件**：...
- **关键参数/实现**：...
- **禁止回退或改坏的点**：...
```

# Changelog

## v1.3.0-alpha1

- Phase 1a: X11 全兼容优化
- 新增 `detectDisplayServer()` 函数，区分 X11/Wayland 环境
- `windowState` 增加 `displayServer` 字段供前端调优
- X11 下禁用 GPU vsync，避免与 compositor 双重帧同步导致卡顿

## v1.2.1

- 简化为 Linux 专用配置：移除 Windows 构建目标 (win/nsis/rcedit/toolsets)
- 脚本重命名：`build:linux` → `build`，`build:linux:dir` → `build:dir`
- 重写 README.md + 新增 README_EN.md，包含安装/构建/限制说明
- 构建验证：AppImage (135MB) + deb (105MB) 构建成功，GUI 在 X11 上正常启动
- 记录者：Silly (iloveu521)

## v1.2.0

- Linux 平台迁移完成：Mineradio 现支持 Linux (X11)，提供 AppImage 和 deb 包
- GPU 渲染：自动选择 GL/Vulkan 后端，不再强制 D3D11
- 节拍缓存：Linux 上使用 XDG_CACHE_HOME (~/.cache/mineradio/beatmaps)
- 窗口图标：Linux 上使用 PNG 格式图标
- 构建系统：新增 build:linux AppImage/deb 目标
- 已知限制：Wayland 不完全兼容、桌面歌词中键锁定不可用、壁纸模式无 WorkerW 嵌入、无自动更新

## v0.5.0

- Linux 迁移 Phase 4：构建系统配置
- package.json 新增 build:linux / build:linux:dir 脚本
- 新增 linux 构建块，目标 AppImage + deb

## v0.4.0

- Linux 迁移 Phase 3：窗口图标修复
- 新增 getAppIcon() 平台感知图标选择函数
- Linux 使用 build/icon.png 替代 .ico

## v0.3.0

- Linux 迁移 Phase 2：节拍缓存路径修复
- 节拍缓存默认路径从 D:\ 迁移到 XDG 规范 (~/.cache/mineradio/beatmaps)
- beatCacheRootInfo() 重写，Linux 无盘符逻辑
- 缓存状态接口错误信息 Linux 适配

## v0.2.0

- Linux 迁移 Phase 1：GPU 渲染后端修复
- 将 `use-angle d3d11` 从固定 Chromium 启动参数中移除，改为仅在 Windows 平台添加
- Linux 上使用默认 GL/Vulkan 后端，避免 D3D11 不存在导致 Electron 启动崩溃

## v0.1.0

- 文档目录结构重组：docs/ 按 architecture/design/roadmap/release/history/security 分类
- 新增 docs/README.md（文档索引导航）
- 新增 docs/architecture/ARCHITECTURE.md（完整系统架构文档）
- 新增 docs/design/LINUX_DESIGN.md（Linux 迁移设计文档）
- 新增 docs/roadmap/ROADMAP.md（项目路线图）
- 项目记录归档，记录者 Silly (iloveu521)


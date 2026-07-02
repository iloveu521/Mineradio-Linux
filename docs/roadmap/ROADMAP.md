# Mineradio Linux Roadmap

> 最后更新: 2026-07-02 | 当前版本: v1.2.1

---

# Current Sprint

**目标版本**: v1.3.0
**开始日期**: 2026-07-02
**状态**: 进行中

| Phase | 名称 | 状态 | 涉及文件 |
|-------|------|------|----------|
| Phase 1 | UI Linux 桌面环境兼容优化 | ⬜ 待开始 | `desktop/main.js` |
| Phase 2 | 左侧歌单交互优化 | ⬜ 待开始 | `public/index.html` |
| Phase 3 | 桌面歌词显示逻辑优化 | ⬜ 待开始 | `desktop/main.js` |
| Phase 4 | 音量按钮滚轮调节 | ⬜ 待开始 | `public/index.html` |
| Phase 5 | 进度条拖动响应优化 | ⬜ 待开始 | `public/index.html` |
| Phase 6 | 控制台桌面歌词开关按钮 | ⬜ 待开始 | `public/index.html` |

---

## Phase 1 — UI Linux 桌面环境兼容优化

**目标**: 改善 X11 下偶发卡顿，Wayland 下给出兼容性提示

**问题描述**:
- 使用过程中偶尔出现卡顿，疑似 X11 compositor vsync 交互问题
- Wayland 用户不知情下体验可能不佳（透明窗口、全局快捷键受限）
- 当前无任何桌面环境检测逻辑

**方向**:
- 读取 `XDG_SESSION_TYPE` 环境变量区分 X11/Wayland
- Wayland 下首次启动弹窗提示，建议 `--ozone-platform=x11`
- 检查 `requestAnimationFrame` 节流逻辑与 Linux compositor 的兼容性
- 适当调整 Linux 下的渲染像素比预算

**详细设计**: [`docs/design/DESIGN.md`](../design/DESIGN.md#phase-1---ui-linux-桌面环境兼容优化)

---

## Phase 2 — 左侧歌单交互优化

**目标**: 歌单面板出现/消失更流畅，响应更及时

**问题描述**:
- 鼠标移到左侧触发区时歌单出现过程有卡顿
- 鼠标移走后歌单不会及时消失，延迟过长

**方向**:
- 审查 `#playlist-panel` 的 peek/hide 逻辑（CSS transition + JS timeout）
- 缩短 hide timeout，或改用 `pointerleave` 事件即时触发
- 检查 DOM 重排/重绘是否导致卡顿

**详细设计**: [`docs/design/DESIGN.md`](../design/DESIGN.md#phase-2---左侧歌单交互优化)

---

## Phase 3 — 桌面歌词显示逻辑优化

**目标**: 桌面歌词根据窗口状态自动显示/隐藏

**问题描述**:
- 桌面歌词始终显示在最上层，即使应用窗口可见时也遮挡桌面
- 理想行为：打开应用窗口 → 关闭桌面歌词；最小化/关闭应用 → 显示桌面歌词

**方向**:
- `desktop/main.js` 监听窗口 `hide`/`show`/`minimize`/`restore` 事件
- 窗口可见 → 自动 `closeDesktopLyricsWindow()`
- 窗口最小化/不可见 → 自动 `createDesktopLyricsWindow()`
- 保留用户手动开启/关闭的优先级

**详细设计**: [`docs/design/DESIGN.md`](../design/DESIGN.md#phase-3---桌面歌词显示逻辑优化)

---

## Phase 4 — 音量按钮滚轮调节

**目标**: 鼠标悬停音量按钮时，滚轮可调节音量

**问题描述**:
- 当前调整音量只能拖动滑块或点按按钮，不够便捷
- 大多数桌面播放器支持悬停滚轮调节

**方向**:
- `#volume-area` 或音量相关元素添加 `wheel` 事件
- `event.deltaY < 0` → 增加音量，`> 0` → 减小音量
- 滚轮步长和键盘方向键一致（每次 5%）

**详细设计**: [`docs/design/DESIGN.md`](../design/DESIGN.md#phase-4---音量按钮滚轮调节)

---

## Phase 5 — 进度条拖动响应优化

**目标**: 进度条拖动跟手，无延迟感

**问题描述**:
- 拖动进度条时响应慢、不跟手
- seek 操作有明显延迟

**方向**:
- 检查进度条 `pointermove`/`mousedown` 处理链路
- 减少 seek debounce 间隔，或改用 `input` 事件
- 确认渲染循环未阻塞 UI 事件

**详细设计**: [`docs/design/DESIGN.md`](../design/DESIGN.md#phase-5---进度条拖动响应优化)

---

## Phase 6 — 控制台桌面歌词开关按钮

**目标**: 在播放器底部控制台增加桌面歌词开关

**问题描述**:
- 桌面歌词开启需要进入设置面板，路径太深
- 需要一个便捷的开关入口

**方向**:
- `#bottom-bar` 控制区增加桌面歌词图标按钮
- 点击触发 IPC `mineradio-desktop-lyrics-set-enabled`
- 按钮高亮状态反映桌面歌词是否开启

**详细设计**: [`docs/design/DESIGN.md`](../design/DESIGN.md#phase-6---控制台桌面歌词开关按钮)

---

## 已完成

- [x] **v1.2.1** — Linux 专用配置、README 重写、独立仓库、Release 发布
- [x] **v1.2.0** — Linux 平台迁移完成（AppImage + deb）
- [x] **v0.6.0** — 集成测试
- [x] **v0.5.0** — Linux 构建系统配置
- [x] **v0.4.0** — 窗口图标 PNG 适配
- [x] **v0.3.0** — 节拍缓存 XDG 路径
- [x] **v0.2.0** — GPU 渲染 D3D11 条件化
- [x] **v0.1.0** — 文档结构重组

## 未来方向 (Backlog)

- Wayland 完整兼容（透明窗口 + globalShortcut）
- 桌面歌词中键锁定替代方案
- 壁纸模式 Linux 适配
- 自动更新 AppImage 支持

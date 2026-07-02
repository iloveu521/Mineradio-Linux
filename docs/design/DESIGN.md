# Mineradio Linux v1.3.0 — Short-term Optimization Design

> 版本: v1.3.0-draft | 日期: 2026-07-02 | 状态: 设计阶段

---

## Overview

v1.3.0 的目标是修复 Linux 桌面环境下的 6 个用户体验问题。每个 Phase 独立可交付，按优先级排列。

---

## Phase 1 — UI Linux 桌面环境兼容优化

### Why

- 用户反馈使用过程中偶尔卡顿。Linux 下 Electron 的 `requestAnimationFrame` 与 X11 compositor vsync 可能存在不协调，导致帧率不稳定。
- Wayland 用户不知情下可能遇到透明窗口和全局快捷键受限问题。
- 当前代码无任何 `XDG_SESSION_TYPE` 检测，无法针对不同 display server 调优。

### How

**文件**: `desktop/main.js`

1. 新增 `detectDisplayServer()` 函数，读取 `XDG_SESSION_TYPE` / `WAYLAND_DISPLAY` 环境变量
2. 在 `app.whenReady()` 中检测，Wayland 下首次启动弹出 `dialog.showMessageBox` 兼容性提示
3. 记录标记避免重复弹窗
4. `sendWindowState()` 增加 `displayServer` 字段，供前端按需调整渲染策略

---

## Phase 2 — 左侧歌单交互优化

### Why

- 鼠标移入左侧触发区时歌单出现有卡顿，可能是 CSS transition 触发大量重排
- 鼠标移走后消失延迟过长，影响其他操作

### How

**文件**: `public/index.html`

1. 定位 `#playlist-panel` 的 peek/hide 逻辑
2. 缩短 hide timeout 到 ~100-150ms
3. 检查面板出现动画，确认 `will-change: transform` 已设置
4. 如使用 `mouseleave`，改为 `pointerleave`（更可靠的离开检测）

---

## Phase 3 — 桌面歌词显示逻辑优化

### Why

- 桌面歌词始终在最上层，打开主窗口时也显示，遮挡桌面
- 用户期望：打开应用 → 关歌词；最小化/关闭 → 显歌词

### How

**文件**: `desktop/main.js`

1. 注册 `mainWindow` 的 `hide`/`show`/`minimize`/`restore` 事件
2. 新增 `updateDesktopLyricsForWindowState()` 函数：
   - 窗口可见 → 关闭桌面歌词（保留 `userEnabled` 标记）
   - 窗口最小化/不可见 → 恢复显示
3. 新增 `desktopLyricsState.userEnabled` 和 `desktopLyricsState.autoHidden` 字段
4. 用户手动操作优先于自动管理

---

## Phase 4 — 音量按钮滚轮调节

### Why

- 当前只能拖动滑块或点按按钮调整音量，不够便捷
- 大多数桌面播放器支持悬停滚轮调节

### How

**文件**: `public/index.html`

1. 定位音量 DOM 元素（`#volume-area` 或音量图标）
2. 添加 `wheel` 事件：`deltaY < 0` 增加 5%，`> 0` 减小 5%
3. 调用已有的 `setVolume()`/`getVolume()` 函数
4. `preventDefault` 阻止页面滚动

---

## Phase 5 — 进度条拖动响应优化

### Why

- 拖动进度条时不跟手，有明显延迟
- 可能是 seek debounce 或渲染循环覆盖了拖动值

### How

**文件**: `public/index.html`

1. 定位进度条元素（`#progress-bar` 或类似）的 `pointerdown`/`pointermove`/`pointerup` 事件
2. 检查 seek 相关的 debounce，移除或降到 ~16ms
3. 分离 UI 更新和实际 seek：`pointermove` 更新视觉，`pointerup` 执行 `audio.currentTime`
4. 添加 `isDraggingProgress` 标志，拖动期间暂停自动进度更新

---

## Phase 6 — 控制台桌面歌词开关按钮

### Why

- 当前桌面歌词需进入设置面板开关，路径太深
- 需要一个一键快捷切换入口

### How

**文件**: `public/index.html`

1. 定位 `#bottom-bar` 控制台按钮区域（`.control-cluster.modes`）
2. 添加桌面歌词图标按钮 `#desktop-lyrics-toggle`
3. 点击触发 IPC `setDesktopLyricsEnabled`
4. 监听 `onDesktopLyricsEnabledState` 回调，按钮状态同步
5. 复用现有 `.control-btn` 样式，active 态使用高亮色

---

## Verification

每个 Phase 完成后：

| Phase | 验证 |
|-------|------|
| 1 | Wayland → 弹兼容提示；X11 → 不弹；`windowState` 含 `displayServer` |
| 2 | 鼠标移入歌单 ~200ms 滑出；移走 ~150ms 消失 |
| 3 | 播放 → 打开主窗口 → 歌词消失 → 最小化 → 歌词出现 |
| 4 | 悬停音量图标滚轮 → 音量 ±5% → 滑块同步 |
| 5 | 拖动进度条 → 视觉实时跟手 → 松手 seek |
| 6 | 点击按钮 → 桌面歌词开/关 → 按钮状态同步 |

全局: `node --check` 通过 + `npm start` 无崩溃

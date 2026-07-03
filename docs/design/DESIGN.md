# Mineradio Linux v1.3.0 — Short-term Optimization Design

> 版本: v1.3.0-draft | 日期: 2026-07-02 | 状态: 设计阶段

---

## Overview

v1.3.0 的目标是修复 Linux 桌面环境下的 7 个用户体验问题。每个 Phase 独立可交付，按优先级排列。

---

## Phase 1a — X11 全兼容优化

### Why

- 用户反馈 X11 下偶尔卡顿。`requestAnimationFrame` 与 X11 compositor vsync 可能不协调。
- X11 是 Linux 下最主流的 display server，应作为推荐配置稳定运行。
- 当前无 X11 专项性能调优。

### How

**文件**: `desktop/main.js`

1. X11 环境检测：`detectDisplayServer()` 返回 `'x11'` 时，前端可针对 X11 做渲染优化
2. 检查 `CHROMIUM_PERFORMANCE_SWITCHES`：
   - 考虑添加 `--disable-gpu-vsync`（测试后决定）
   - 保留 `ignore-gpu-blocklist`、`enable-zero-copy` 等跨平台优化
3. 检查 `getRenderLoadTier()` 像素预算是否适合 Linux X11
4. `sendWindowState()` 增加 `displayServer: 'x11'` 字段
5. 在 X11 上测试帧率稳定性，记录基准数据

### Verification
- [x] X11 下 `npm start` 启动无崩溃
- [x] 播放歌曲时帧率稳定
- [x] `windowState.displayServer === 'x11'`

---

## Phase 1a-2 — GPU 检测与分级优化

### Why

- 不同显卡性能差异巨大。NVIDIA RTX 3060 可稳定 120fps，Intel HD 核显 30fps 就吃力。
- Chromium 的 ANGLE 后端（GL/Vulkan/EGL）在不同厂商的 Linux 驱动上表现不同：NVIDIA 闭源驱动走 EGL 最稳，AMD Mesa radv 走 Vulkan 最佳，Intel 走默认 GL 最可靠。
- 用户无法知道自己的显卡应该跑多少帧，也没有简单的方式调节帧率上限。
- 当前代码无 GPU 检测逻辑，对所有 Linux 用户一视同仁。

### GPU 分级

| Tier | 默认帧率 | 典型 GPU |
|------|---------|---------|
| `high` | 120 fps | NVIDIA RTX 3060+, AMD RX 6600+, Intel Arc |
| `medium` | 90 fps | NVIDIA GTX 10/16/RTX 2050, AMD RX 5xx, Intel Xe/Iris |
| `low` | 60 fps | Intel UHD/HD 核显, 老旧独显 |
| `min` | 30 fps | 用户手动降级，不自动分配 |

### How

**文件**: `desktop/main.js` + `public/index.html`

**Step 1 — GPU 检测** (`desktop/main.js`):
```js
const { execSync } = require('child_process');

function detectLinuxGpu() {
  try {
    const raw = execSync("lspci -mm -d ::0300", { encoding: 'utf8', timeout: 2000 }).trim();
    const line = raw.split('\n')[0] || '';
    const lower = line.toLowerCase();
    const vendor = lower.includes('nvidia') ? 'nvidia'
      : lower.includes('amd') || lower.includes('ati') || lower.includes('radeon') ? 'amd'
      : lower.includes('intel') ? 'intel'
      : 'unknown';
    return { vendor, raw: line };
  } catch (_) {
    return { vendor: 'unknown', raw: '' };
  }
}

function detectGpuTier(vendor, raw) {
  const lower = raw.toLowerCase();
  if (vendor === 'nvidia') {
    if (/rtx [3-9]\d\d\d/i.test(lower) || /rtx [4-9]\d\d\d/i.test(lower) || /rtx 2[5-9]\d\d/i.test(lower)) return 'high';
    if (/rtx 20\d\d/i.test(lower) || /rtx 2050/i.test(lower)) return 'medium';
    if (/gtx 1[0-6]\d\d/i.test(lower) || /gtx 9\d\d/i.test(lower)) return 'medium';
    return 'low';
  }
  if (vendor === 'amd') {
    if (/rx [67]\d\d\d/i.test(lower) || /radeon rx [67]\d\d\d/i.test(lower)) return 'high';
    if (/rx [45]\d\d/i.test(lower) || /radeon rx [45]\d\d/i.test(lower)) return 'medium';
    return 'low';
  }
  if (vendor === 'intel') {
    if (/arc/i.test(lower)) return 'high';
    if (/xe/i.test(lower) || /iris/i.test(lower)) return 'medium';
    return 'low';
  }
  return 'low';
}

// 默认帧率表
const GPU_TIER_FPS = { high: 120, medium: 90, low: 60, min: 30 };
```

**Step 2 — Chromium 厂商 flags** (`desktop/main.js`):
```js
// 根据 GPU 厂商添加专用优化 flags
if (process.platform === 'linux') {
  if (gpuVendor === 'amd') {
    CHROMIUM_PERFORMANCE_SWITCHES.push(['use-angle', 'vulkan']);
    app.commandLine.appendSwitch('enable-features', 'Vulkan');
  } else if (gpuVendor === 'nvidia') {
    CHROMIUM_PERFORMANCE_SWITCHES.push(['use-gl', 'angle']);
    CHROMIUM_PERFORMANCE_SWITCHES.push(['use-angle', 'gl-egl']);
  }
  // Intel: 默认后端已是最佳，不加额外 flag
}
```

**Step 3 — windowState 携带 GPU 信息** (`desktop/main.js`):
```js
// getWindowState() 返回增加:
gpuVendor: GPU_VENDOR,
gpuTier: GPU_TIER,
maxFps: GPU_DEFAULT_FPS,
```

**Step 4 — 前端帧率 DIY** (`public/index.html`):
- 读取 `fx.maxFps`（用户自定义值）或 `windowState.maxFps`（默认值）
- 在限帧循环中应用
- 视觉控制台添加 `maxFps` 滑块（范围 30-120，步长 10）

### Verification

- [ ] `lspci` 检测到 GPU → `windowState.gpuVendor` 非 `unknown`
- [ ] GPU tier 分级正确 → 对应 `windowState.maxFps`
- [ ] AMD 上启用 Vulkan backend；NVIDIA 上启用 GL-EGL backend
- [ ] 前端帧率受 `maxFps` 限制
- [ ] 视觉控制台滑块可调节帧率（30-120），持久化到 `localStorage`

---

## Phase 1b — Wayland 全兼容优化

### Why

- Wayland 下透明窗口和全局快捷键不完全支持。
- 用户不知情下可能遇到渲染异常或快捷键失效。
- 需要检测 + 提示，让用户做知情选择。

### How

**文件**: `desktop/main.js`

1. 新增 `detectDisplayServer()` 函数：
```js
function detectDisplayServer() {
  if (process.env.WAYLAND_DISPLAY) return 'wayland';
  if ((process.env.XDG_SESSION_TYPE || '').toLowerCase() === 'wayland') return 'wayland';
  return 'x11';
}
```
2. 在 `app.whenReady()` 中检测，Wayland 下弹窗：
   > "检测到 Wayland 环境。Mineradio Linux 在 Wayland 下部分功能受限（透明窗口渲染可能异常、全局快捷键不可用）。建议使用 `--ozone-platform=x11` 启动以获得最佳体验。继续使用 Wayland？"
   - 首次弹窗，用户确认后记录标记不再提示
3. Wayland 下 `sendWindowState()` 返回 `displayServer: 'wayland'`，前端据此调整行为（如避免依赖透明效果）

### Verification
- Wayland 下启动 → 弹出兼容提示
- 确认后不再弹窗
- X11 下不弹窗
- `windowState.displayServer === 'wayland'`

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
| 1a | ✅ X11 下帧率稳定 60fps；`windowState.displayServer === 'x11'` |
| 1b | Wayland → 弹兼容提示；X11 → 不弹；`windowState.displayServer === 'wayland'` |
| 2 | 鼠标移入歌单 ~200ms 滑出；移走 ~150ms 消失 |
| 3 | 播放 → 打开主窗口 → 歌词消失 → 最小化 → 歌词出现 |
| 4 | 悬停音量图标滚轮 → 音量 ±5% → 滑块同步 |
| 5 | 拖动进度条 → 视觉实时跟手 → 松手 seek |
| 6 | 点击按钮 → 桌面歌词开/关 → 按钮状态同步 |

全局: `node --check` 通过 + `npm start` 无崩溃

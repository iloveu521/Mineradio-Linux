# Mineradio Linux Migration Design

> 版本: v0.1.0-draft | 日期: 2026-07-02 | 状态: 待审查

---

## 1. 目标

将 Mineradio 从 **Windows-only Electron 桌面音乐播放器** 完整迁移到 **Linux 平台**。

**约束**:
- 不添加新功能，纯平台迁移
- 保留所有现有功能
- 保留 Windows 平台兼容（一次构建、双平台）
- 前端 SPA 零变更

---

## 2. 变更范围

| 文件 | 变更量 | 阶段 |
|------|--------|------|
| `desktop/main.js` | ~15 行 | Phase 1 + 3 |
| `server.js` | ~35 行 | Phase 2 |
| `package.json` | ~20 行 | Phase 4 |
| `docs/design/DESIGN.md` | 新建 | Phase 0 |
| `docs/roadmap/ROADMAP.md` | ~5 行 | Phase 0 |
| `CHANGELOG.md` | ~10 行 | Phase 6 |
| `README.md` | ~5 行 | Phase 6 |

**不变的文件**:
- `public/index.html` (~27000 行) — 零平台特定代码
- `public/desktop-lyrics.html` — 零平台特定代码
- `public/wallpaper.html` — 零平台特定代码
- `desktop/preload.js` — 纯 IPC 桥接
- `desktop/overlay-preload.js` — 纯 IPC 桥接
- `dj-analyzer.js` — 纯 WebAssembly
- `build/after-pack.js` — 已有 `platform !== 'win32'` 守卫
- `build/installer.nsh` — Windows NSIS 脚本，Linux 构建不引用

---

## 3. 实施阶段

### Phase 0 — 文档与准备 → `v0.1.0`

**依赖**: 无

- [ ] 写入本文件 `docs/design/DESIGN.md`
- [ ] 更新 `docs/roadmap/ROADMAP.md` Linux 条目状态为"进行中"

**验证**:
- [ ] `node --check server.js` 通过
- [ ] `node --check desktop/main.js` 通过
- [ ] `npm install` 在 Linux 上无错误

---

### Phase 1 — GPU 渲染修复 → `v0.2.0`

**文件**: `desktop/main.js`

**问题**: `CHROMIUM_PERFORMANCE_SWITCHES` 数组 (lines 40-52) 包含 `['use-angle', 'd3d11']`，强制 Chromium 使用 Direct3D 11 后端。Linux 上没有 D3D11，Electron 启动会因 GPU 初始化失败而崩溃。

**修复**:

```js
// 将 use-angle d3d11 从固定数组移除，改为平台条件追加
const CHROMIUM_PERFORMANCE_SWITCHES = [
  ['autoplay-policy', 'no-user-gesture-required'],
  ['ignore-gpu-blocklist'],
  ['enable-gpu-rasterization'],
  ['enable-oop-rasterization'],
  ['enable-zero-copy'],
  ['enable-accelerated-2d-canvas'],
  ['disable-background-timer-throttling'],
  ['disable-renderer-backgrounding'],
  ['disable-backgrounding-occluded-windows'],
  ['force_high_performance_gpu'],
  // use-angle d3d11 仅 Windows 添加
];

if (process.platform === 'win32') {
  CHROMIUM_PERFORMANCE_SWITCHES.push(['use-angle', 'd3d11']);
}
```

其余 10 个开关全部跨平台适用（ignore-gpu-blocklist、enable-zero-copy 等在 Mesa/Vulkan 下同样有效）。

**验证**:
- [ ] `node --check desktop/main.js` 通过
- [ ] `npm start` 启动不因 GPU 初始化崩溃

---

### Phase 2 — 节拍缓存路径修复 → `v0.3.0`

**文件**: `server.js`

#### 2a — 硬编码 Windows 路径 (line 65)

**当前**:
```js
const BEATMAP_CACHE_DIR = process.env.MINERADIO_BEAT_CACHE_DIR || 'D:\\MineradioCache\\beatmaps';
```

**修复**: 引入 `os` 模块，按平台选择默认路径。

```js
const os = require('os');

function defaultBeatCacheDir() {
  if (process.platform === 'win32') return 'D:\\MineradioCache\\beatmaps';
  // Linux: 遵循 XDG Base Directory 规范
  const xdgCache = process.env.XDG_CACHE_HOME || path.join(os.homedir(), '.cache');
  return path.join(xdgCache, 'mineradio', 'beatmaps');
}
const BEATMAP_CACHE_DIR = process.env.MINERADIO_BEAT_CACHE_DIR || defaultBeatCacheDir();
```

#### 2b — C 盘检测逻辑 (lines 507-531)

**当前** `beatCacheRootInfo()`:
```js
function beatCacheRootInfo() {
  const dir = path.resolve(BEATMAP_CACHE_DIR);
  const root = path.parse(dir).root;
  const drive = root ? root.replace(/[\\\/]+$/, '').toUpperCase() : '';
  const allowed = !!root && !/^C:$/i.test(drive);
  const available = allowed && fs.existsSync(root);
  return { dir, root, drive, allowed, available };
}
```

这个函数用 Windows 盘符模型判断"是否为 C 盘"。Linux 没有盘符，需要不同的验证逻辑。

**修复**: 按 `process.platform` 分流。

```js
function beatCacheRootInfo() {
  const dir = path.resolve(BEATMAP_CACHE_DIR);
  if (process.platform === 'win32') {
    const root = path.parse(dir).root;
    const drive = root ? root.replace(/[\\\/]+$/, '').toUpperCase() : '';
    const allowed = !!root && !/^C:$/i.test(drive);
    const available = allowed && fs.existsSync(root);
    return { dir, root, drive, allowed, available };
  }
  // Linux: 检查父目录是否可写
  const parentDir = path.dirname(dir);
  let available = false;
  try {
    if (!fs.existsSync(parentDir)) {
      fs.mkdirSync(parentDir, { recursive: true });
    }
    fs.accessSync(parentDir, fs.constants.W_OK);
    available = true;
  } catch (_) {
    available = false;
  }
  return { dir, root: '/', drive: 'unix', allowed: true, available };
}
```

#### 2c — 状态接口错误信息 (line ~3324)

缓存状态接口 `/api/beatmap/cache/status` 返回的 `reason` 字段硬编码 `"C_DRIVE_DISABLED"`。在 Linux 上应返回语义正确的错误原因。

**修复**: 在 `beatCacheRootInfo()` 返回中添加 `reason` 字段，或在状态端点判断 `process.platform`。Linux 上不可用时返回 `"CACHE_DIR_UNAVAILABLE"`。

**验证**:
- [ ] `node --check server.js` 通过
- [ ] 单独启动 server: `node server.js`，检查缓存目录初始化日志
- [ ] `/api/beatmap/cache/status` 返回 `{"mode":"disk"}` 且 dir 为 `~/.cache/mineradio/beatmaps`
- [ ] 播客 DJ 节拍分析后缓存文件写入正确路径

---

### Phase 3 — 窗口图标修复 → `v0.4.0`

**文件**: `desktop/main.js`

**问题**: `APP_ICON_ICO` (line 34) 固定指向 `build/icon.ico`。Electron 在 Linux 上不支持 ICO 格式作为窗口图标——需要 PNG。

好消息：`build/icon.png` (28KB) 已存在。

**图标使用位置**:

| 行号 | 上下文 | 当前代码 |
|------|--------|----------|
| 34 | 常量定义 | `const APP_ICON_ICO = path.join(__dirname, '..', 'build', 'icon.ico');` |
| 291 | 桌面快捷方式 | `icon: fs.existsSync(APP_ICON_ICO) ? APP_ICON_ICO : target` — 已有 `platform !== 'win32'` 守卫 |
| 420 | 网易云登录窗口 | `icon: APP_ICON_ICO` — **无守卫** |
| 522 | QQ 登录窗口 | `icon: APP_ICON_ICO` — **无守卫** |
| 1360 | 主窗口 | `icon: APP_ICON_ICO` — **无守卫** |

Line 291 已守卫，不变。Lines 420/522/1360 需要改为平台感知。

**修复**:

```js
const APP_ICON_ICO = path.join(__dirname, '..', 'build', 'icon.ico');
const APP_ICON_PNG = path.join(__dirname, '..', 'build', 'icon.png');

function getAppIcon() {
  if (process.platform === 'win32') {
    return fs.existsSync(APP_ICON_ICO) ? APP_ICON_ICO : APP_ICON_PNG;
  }
  return APP_ICON_PNG;
}
```

然后将 `icon: APP_ICON_ICO` (lines 420, 522, 1360) 替换为 `icon: getAppIcon()`。

**验证**:
- [ ] `node --check desktop/main.js` 通过
- [ ] 主窗口标题栏/任务栏显示正确图标
- [ ] 登录子窗口显示正确图标

---

### Phase 4 — 构建系统配置 → `v0.5.0`

**文件**: `package.json`

#### 4a — 新增构建脚本

```json
"scripts": {
  "start": "electron .",
  "build:win": "electron-builder --win nsis",
  "build:win:dir": "electron-builder --win dir",
  "build:linux": "electron-builder --linux appimage deb",
  "build:linux:dir": "electron-builder --linux dir"
}
```

#### 4b — 新增 Linux 构建块

```json
"linux": {
  "executableName": "mineradio",
  "icon": "build/icon.png",
  "category": "AudioVideo",
  "target": [
    { "target": "AppImage", "arch": ["x64"] },
    { "target": "deb", "arch": ["x64"] }
  ]
}
```

**构建目标选择**:
- **AppImage** — 通用 Linux 格式，单文件，任何 glibc 发行版直接运行。推荐作为主要分发格式。
- **deb** — Debian/Ubuntu 原生包，提供 `.desktop` 入口和 MIME 关联。

**不变**: Windows 构建块 (`win`、`nsis`、`toolsets.winCodeSign`、`devDependencies.rcedit`) 全部保留。

**验证**:
- [ ] `npm run build:linux:dir` 生成 `dist/linux-unpacked/`
- [ ] `npm run build:linux` 生成 `.AppImage` 和 `.deb`

---

### Phase 5 — 集成测试 → `v0.6.0`

无代码变更。在 Ubuntu 22.04/24.04 X11 桌面环境下系统性验证。

**测试清单**:

1. **应用生命周期**
   - 启动 → 主窗口正常显示（无框透明）
   - 单实例锁：重复启动聚焦已有窗口
   - 最小化/最大化/全屏/关闭

2. **音乐播放**
   - 网易云搜索 → 播放 → 歌词 → 封面
   - QQ 音乐搜索 → 播放 → 歌词
   - 播客/DJ 播放 → 节拍分析（验证缓存路径）
   - 音量控制、音质切换

3. **3D 视觉**
   - 5 种视觉预设全部渲染正常（Silk/Tunnel/Orbit/Void/Vinyl）
   - 3D 歌单架交互正常
   - 电影镜头运镜正常

4. **登录**
   - 网易云 QR 登录 → Cookie 持久化
   - QQ 音乐 Cookie 登录 → 持久化

5. **桌面歌词**
   - 叠加窗口创建正常（透明）
   - 锁定态鼠标穿透正常
   - 中键锁定 **不可用**（已知限制）

6. **壁纸模式**
   - 普通全屏覆盖窗口（非 WorkerW 嵌入）

7. **全局快捷键** (X11)
   - 注册/注销正常
   - Wayland 下不可用（已知限制）

8. **更新通知**
   - GitHub Releases 检查正常
   - 手动下载入口可用

---

### Phase 6 — 文档收尾 → `v1.2.0`

- [ ] 更新 `CHANGELOG.md`：Linux 兼容性说明
- [ ] 更新 `README.md`：Linux 下载说明
- [ ] 更新 `docs/roadmap/ROADMAP.md`：标记 Linux 迁移完成，Wayland 支持移至后续

---

## 4. 不修改的功能（已有平台守卫）

以下 Windows 专属功能已有 `process.platform` 守卫，Linux 上自动跳过或优雅降级：

| 功能 | 文件 | 守卫 | Linux 行为 |
|------|------|------|-----------|
| 桌面歌词中键锁定 | main.js:814 | `process.platform !== 'win32'` return | PowerShell 轮询不启动，UI 手动切换正常 |
| WorkerW 壁纸嵌入 | main.js:986 | `process.platform !== 'win32'` return | 壁纸为普通全屏窗口 |
| .lnk 桌面快捷方式 | main.js:276 | `process.platform !== 'win32'` return | deb 包提供 .desktop |
| AppUserModelId | main.js:1430 | `process.platform === 'win32'` guard | 自动跳过 |
| NSIS 安装器 | build/installer.nsh | 仅 win32 构建引用 | 不参与 Linux 构建 |
| rcedit .exe 注入 | build/after-pack.js:44 | `platform !== 'win32'` return | 自动跳过 |

---

## 5. 已知限制

### 5.1 Wayland 不完全兼容

- `globalShortcut` 在 Wayland 下不可用（Electron/Chromium 限制）
- 透明窗口（`transparent: true`）在某些 Wayland 合成器上有渲染问题
- **首期目标 X11**，Wayland 用户可使用 `--ozone-platform=x11` 环境变量

### 5.2 桌面歌词中键锁定

- Windows 通过 PowerShell + GetAsyncKeyState(4) 实现全局中键检测
- Linux 无跨平台等价方案
- 用户需通过应用内 UI 切换锁定/解锁

### 5.3 壁纸模式

- WorkerW/Progman 嵌入是 Windows 独有机制
- Linux 上壁纸窗口为普通全屏覆盖，无法嵌入桌面背景层

### 5.4 自动更新

- Windows 的 NSIS 安装器支持 `shell.openPath()` 自动打开 .exe
- AppImage 无等价机制，用户需手动下载替换
- 更新通知（GitHub Releases 检查）跨平台正常

---

## 6. 依赖关系图

```
Phase 0 (v0.1.0) ── 文档准备
    │
Phase 1 (v0.2.0) ── GPU 渲染修复 (main.js)
    │
Phase 2 (v0.3.0) ── 节拍缓存修复 (server.js)
    │
Phase 3 (v0.4.0) ── 窗口图标修复 (main.js)
    │
Phase 4 (v0.5.0) ── 构建系统 (package.json)
    │
Phase 5 (v0.6.0) ── 集成测试 (无代码变更)
    │
Phase 6 (v1.2.0) ── 文档收尾
```

Phase 1-3 修改不同文件（或同一文件的不同区域），技术上可并行，但按顺序执行更安全。

---

## 7. 风险矩阵

| 风险 | 概率 | 影响 | 缓解措施 |
|------|------|------|----------|
| 特定 GPU/驱动下 GL 渲染异常 | 中 | 高 | `ignore-gpu-blocklist` 标志 + 多驱动测试 |
| Wayland 透明窗口失效 | 高 | 中 | 文档化限制，提供 `--ozone-platform=x11` 方案 |
| globalShortcut 在非 GNOME/KDE DE 注册失败 | 中 | 低 | 已有失败回退（返回 conflict 信息给前端） |
| AppImage 在旧 glibc 上无法运行 | 低 | 中 | 在 Ubuntu 20.04 上构建以兼容更老的 glibc |
| 节拍缓存目录权限不足 | 低 | 低 | 已有 memory-only 降级模式 |

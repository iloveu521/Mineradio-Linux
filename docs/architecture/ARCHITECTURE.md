# Mineradio 系统架构文档

> 版本: v1.1.1 | 更新日期: 2026-07-01

---

## 1. 项目概述

Mineradio 是一款 **Windows Electron 桌面沉浸式音乐播放器**，将天气电台、搜索播放、歌词舞台、粒子视觉、3D 歌单架和 DJ 节奏分析整合为一个私人音乐空间。

- **应用 ID**: `com.mineradio.desktop`
- **许可证**: GPL-3.0
- **作者**: XxHuberrr
- **GitHub**: https://github.com/XxHuberrr/Mineradio

核心体验链路：搜索歌曲 → 播放 → 实时音频频谱分析 → 粒子视觉渲染 + 3D 歌词舞台 + 电影镜头运镜。

---

## 2. 技术栈

| 层级 | 技术 | 说明 |
|------|------|------|
| 桌面壳 | Electron 42 | 无框窗口、系统托盘、全局快捷键、单实例锁 |
| 后端服务 | Node.js HTTP Server | 纯 `http.createServer`，无 Express，手动路由分发 |
| 前端 UI | 原生 HTML/CSS/JS SPA | 单文件 `index.html`（~27,000 行），无框架，无构建工具 |
| 3D 渲染 | Three.js r128 | WebGL 粒子系统、自定义 GLSL Shader、相机系统 |
| 音频分析 | Web Audio API + mpg123-decoder | 实时频谱 + 离线节拍预解析 |
| 动画 | GSAP 3.x | UI 过渡动画、Modal 弹窗、预设切换 |
| BPM 检测 | music-tempo.js | BPM 估算（备选方案） |
| 打包分发 | electron-builder | NSIS 安装器、自动更新、快速补丁 |

---

## 3. 系统架构总览

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Mineradio 系统架构                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                   Electron Main Process                       │   │
│  │                   (desktop/main.js)                           │   │
│  │  ┌──────────┐  ┌───────────┐  ┌──────────┐  ┌────────────┐  │   │
│  │  │ 窗口管理  │  │ 登录窗口   │  │ IPC 通信  │  │ 系统集成   │  │   │
│  │  │ 无框/全屏 │  │ 网易云/QQ  │  │ preload   │  │ 全局快捷键 │  │   │
│  │  │ 桌面歌词  │  │ Cookie持久│  │ bridge    │  │ 单实例锁   │  │   │
│  │  │ 壁纸窗口  │  │ 会话分区  │  │ 双向通道  │  │ 快捷方式   │  │   │
│  │  └──────────┘  └───────────┘  └──────────┘  └────────────┘  │   │
│  └──────────────────────┬───────────────────────────────────────┘   │
│                         │ 启动本地 HTTP 服务                          │
│  ┌──────────────────────▼───────────────────────────────────────┐   │
│  │                  Node.js HTTP Server                          │   │
│  │                    (server.js)                                │   │
│  │  ┌──────────┐  ┌───────────┐  ┌──────────┐  ┌────────────┐  │   │
│  │  │ 网易云API │  │ QQ音乐API  │  │ 更新管理  │  │ 天气电台   │  │   │
│  │  │ 搜索/播放 │  │ 搜索/播放  │  │ GitHub    │  │ Open-Meteo │  │   │
│  │  │ 登录/歌单 │  │ 登录/歌单  │  │ 补丁/下载 │  │ ip-api     │  │   │
│  │  │ 播客/评论 │  │ 歌词/评论  │  │ 版本检查  │  │ 城市定位   │  │   │
│  │  └──────────┘  └───────────┘  └──────────┘  └────────────┘  │   │
│  │  ┌──────────────────────────────────────────────────────┐    │   │
│  │  │              DJ Beat Analyzer (dj-analyzer.js)        │    │   │
│  │  │  音频流下载 → MP3解码 → 高低通滤波 → 能量检测         │    │   │
│  │  │  → Onset检测 → 节拍网格构建 → 相机节拍映射            │    │   │
│  │  └──────────────────────────────────────────────────────┘    │   │
│  └──────────────────────┬───────────────────────────────────────┘   │
│                         │ HTTP 127.0.0.1:<port>                      │
│  ┌──────────────────────▼───────────────────────────────────────┐   │
│  │                     Web 前端 (public/)                        │   │
│  │  ┌──────────────────────────────────────────────────────┐    │   │
│  │  │              index.html (SPA, ~27,000行)               │    │   │
│  │  │  ┌────────┐  ┌────────┐  ┌────────┐  ┌──────────┐   │    │   │
│  │  │  │Three.js│  │WebAudio│  │Canvas2D│  │MediaPipe │   │    │   │
│  │  │  │3D场景  │  │频谱分析│  │纹理生成│  │手势追踪  │   │    │   │
│  │  │  │粒子系统│  │节拍检测│  │歌词渲染│  │捏合旋转  │   │    │   │
│  │  │  │相机系统│  │音量控制│  │封面处理│  │握拳检测  │   │    │   │
│  │  │  │歌词舞台│  │淡入淡出│  │UI组件  │  │          │   │    │   │
│  │  │  │3D歌单架│  │        │  │        │  │          │   │    │   │
│  │  │  └────────┘  └────────┘  └────────┘  └──────────┘   │    │   │
│  │  └──────────────────────────────────────────────────────┘    │   │
│  │  ┌─────────────────────┐  ┌─────────────────────────────┐    │   │
│  │  │  desktop-lyrics.html│  │    wallpaper.html            │    │   │
│  │  │  桌面歌词叠加窗口    │  │    壁纸/WorkerW 嵌入窗口     │    │   │
│  │  │  透明/置顶/穿透     │  │    银河背景/桌面融合         │    │   │
│  │  └─────────────────────┘  └─────────────────────────────┘    │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. 核心模块详解

### 4.1 Electron 主进程 (`desktop/main.js`)

**文件规模**: 1,467 行

**职责**:
- **窗口生命周期管理**: 创建无框透明主窗口（960×540 最小，16:9 比例），窗口状态跟踪（最大化/最小化/全屏/聚焦/可见性）
- **桌面歌词窗口**: 独立透明置顶窗口，支持鼠标穿透、中键锁定/解锁、拖拽移动、位置记忆
- **壁纸窗口**: WorkerW 嵌入模式，将 Electron 窗口注入 Windows 壁纸层
- **第三方登录窗口**: 为网易云音乐和 QQ 音乐分别创建独立 `BrowserWindow`，使用隔离的 `session` 分区持久化 Cookie
- **IPC 通信桥**: 通过 `ipcMain.handle` 注册 20+ 个 IPC 通道，处理前端与主进程的通信
- **全局快捷键**: 可配置的全局快捷键注册/注销系统
- **单实例锁**: `app.requestSingleInstanceLock()` 确保只有一个应用实例运行
- **Chromium 性能开关**: 启用 GPU 光栅化、零拷贝、ANGLE D3D11 后端、禁用后台节流
- **应用更新**: 启动本地 HTTP 服务后加载 `http://127.0.0.1:<port>`

**关键 IPC 通道**:

| IPC 通道 | 方向 | 用途 |
|----------|------|------|
| `desktop-window-*` | 双向 | 窗口最小化/最大化/全屏/关闭/状态查询 |
| `netease-music-open-login` | 前端→主进程 | 打开网易云登录窗口 |
| `qq-music-open-login` | 前端→主进程 | 打开 QQ 音乐登录窗口 |
| `mineradio-desktop-lyrics-*` | 双向 | 桌面歌词启用/更新/拖拽/锁定/位置 |
| `mineradio-wallpaper-*` | 双向 | 壁纸模式启用/更新 |
| `mineradio-open-update-installer` | 前端→主进程 | 打开下载好的安装包 |
| `mineradio-hotkeys-configure-global` | 双向 | 全局快捷键配置 |
| `mineradio-export/import-json-file` | 前端→主进程 | 存档导出/导入 |
| `mineradio-restart-app` | 前端→主进程 | 应用重启 |

**窗口状态广播机制**: 主进程在所有窗口事件（maximize/minimize/restore/show/hide/focus/blur/move/resize）时通过 `sendWindowState()` 向前端推送即时状态，前端据此调整渲染策略（如最小化时降帧、失焦时释放资源）。

---

### 4.2 Preload 桥接层

#### `desktop/preload.js` (主窗口)

通过 `contextBridge.exposeInMainWorld('desktopWindow', {...})` 暴露安全的 API：

- 窗口控制（最小化/最大化/全屏/关闭）
- 音乐平台登录/登出
- 全局快捷键配置
- 桌面歌词/壁纸模式控制
- 状态变化事件监听
- JSON 文件导入/导出

#### `desktop/overlay-preload.js` (叠加窗口)

通过 `contextBridge.exposeInMainWorld('desktopOverlay', {...})` 为桌面歌词和壁纸窗口暴露：

- 歌词状态监听
- 壁纸状态监听
- 拖拽/指针捕获控制
- 热区边界设置
- 锁定状态切换

---

### 4.3 Backend HTTP Server (`server.js`)

**文件规模**: 4,203 行

**架构模式**: 纯 `http.createServer` 手动路由，无 Express 框架。

**API 路由清单** (53 个端点):

#### 应用与更新
| 端点 | 方法 | 功能 |
|------|------|------|
| `/api/app/version` | GET | 返回应用版本和更新配置 |
| `/api/update/latest` | GET | 从 GitHub Releases 拉取最新版本信息 |
| `/api/update/download` | POST | 启动完整安装包下载任务 |
| `/api/update/download/status` | GET | 轮询下载任务状态 |
| `/api/update/patch` | POST | 启动增量补丁任务 |
| `/api/update/patch/status` | GET | 轮询补丁任务状态 |

#### 节拍缓存
| 端点 | 方法 | 功能 |
|------|------|------|
| `/api/beatmap/cache/status` | GET | 磁盘缓存状态查询 |
| `/api/beatmap/cache` | GET/POST | 读取/写入节拍缓存 |

#### 发现与天气
| 端点 | 方法 | 功能 |
|------|------|------|
| `/api/discover/home` | GET | 首页个性化推荐（日推+歌单+播客） |
| `/api/weather/radio` | GET | 天气电台（Open-Meteo + 网易云搜索） |
| `/api/weather/ip-location` | GET | IP 地理位置定位 |

#### 搜索
| 端点 | 方法 | 功能 |
|------|------|------|
| `/api/search` | GET | 网易云音乐搜索 |
| `/api/qq/search` | GET | QQ 音乐搜索 |

#### QQ 音乐
| 端点 | 方法 | 功能 |
|------|------|------|
| `/api/qq/song/url` | GET | QQ 歌曲播放 URL |
| `/api/qq/lyric` | GET | QQ 歌曲歌词 |
| `/api/qq/login/status` | GET | QQ 登录状态 |
| `/api/qq/login/cookie` | POST | 提交 QQ Cookie 认证 |
| `/api/qq/logout` | POST | QQ 登出 |
| `/api/qq/user/playlists` | GET | QQ 用户歌单 |
| `/api/qq/playlist/tracks` | GET | QQ 歌单歌曲列表 |
| `/api/qq/artist/detail` | GET | QQ 歌手详情 |
| `/api/qq/song/comments` | GET | QQ 歌曲评论 |

#### 网易云音乐
| 端点 | 方法 | 功能 |
|------|------|------|
| `/api/song/url` | GET | 歌曲播放 URL（含音质探测） |
| `/api/lyric` | GET | 歌词获取 |
| `/api/song/comments` | GET | 歌曲评论 |
| `/api/song/like/check` | GET | 红心状态查询 |
| `/api/song/like` | POST | 喜欢/取消喜欢 |
| `/api/login/qr/key` | GET | 获取扫码登录 Key |
| `/api/login/qr/create` | GET | 生成登录二维码 |
| `/api/login/qr/check` | GET | 轮询扫码状态 |
| `/api/login/cookie` | POST | 提交 Cookie 登录 |
| `/api/login/status` | GET | 当前登录状态 |
| `/api/logout` | POST | 登出 |
| `/api/user/playlists` | GET | 用户歌单列表 |
| `/api/playlist/create` | POST | 创建歌单 |
| `/api/playlist/add-song` | POST | 添加歌曲到歌单 |
| `/api/playlist/tracks` | GET | 歌单歌曲列表 |
| `/api/artist/detail` | GET | 歌手详情 |

#### 播客/DJ
| 端点 | 方法 | 功能 |
|------|------|------|
| `/api/podcast/search` | GET | 播客搜索 |
| `/api/podcast/hot` | GET | 热门播客 |
| `/api/podcast/detail` | GET | 播客详情 |
| `/api/podcast/programs` | GET | 播客节目列表 |
| `/api/podcast/my` | GET | 我的播客收藏 |
| `/api/podcast/my/items` | GET | 播客收藏分页 |
| `/api/podcast/dj-beatmap` | GET | DJ 节拍分析（离线） |

#### 代理与静态
| 端点 | 方法 | 功能 |
|------|------|------|
| `/api/cover` | GET | 专辑封面代理（含 CORS） |
| `/api/audio` | GET | 音频流代理（支持 Range） |
| `/` | GET | 静态文件服务（`public/` 目录） |

**关键设计**:
- **Cookie 持久化**: 网易云和 QQ 的 Cookie 分别保存到 `.cookie` 和 `.qq-cookie` 文件，启动时自动加载
- **音质探测**: `handleSongUrl()` 会遍历多个音质等级，检测 `freeTrialInfo` 判断 VIP 限制
- **更新系统**: 完整的 OTA 更新框架，支持镜像下载分流（国内 ghproxy 等）、sha256 校验、增量快速补丁（JSON patch 格式）
- **节拍缓存**: 支持磁盘缓存目录 `D:\MineradioCache\beatmaps`，避免重复分析长音频
- **TLS 增强**: 合并系统 CA 证书到 Node.js 信任链

---

### 4.4 DJ Beat Analyzer (`dj-analyzer.js`)

**文件规模**: 865 行

**职责**: 离线音频节拍分析引擎，为播客和长音频生成节拍图（beatmap），驱动电影镜头运镜。

**分析流水线**:

```
音频 URL → fetch 流式下载 → mpg123-decoder 解码
  → 双二阶滤波（高通 32Hz + 低通 178Hz）
  → 短时能量帧计算（hop 10-12.5ms）
  → Onset 检测（滑动窗口 + 自适应阈值）
  → 候选节拍筛选（power/score 排序 + minGap 去重）
  → 节拍网格估计（直方图投票 + 中位数）
  → 网格相位对齐（相位评分 + 最优锚点）
  → 分段节拍调整（每 72-96 秒局部重估计）
  → 节拍属性计算（impact/strength/combo/low/body/snap/mass/sharpness）
  → 输出 beatmap JSON
```

**三种分析策略**:

| 策略 | 适用场景 | 函数 |
|------|----------|------|
| 全量流分析 | 音频 ≤ 2 小时 | `analyzePodcastDjStreamFull()` → 完整下载+解码+分析 |
| 采样范围分析 | 音频 > 2 小时 | `analyzePodcastDjRangeSamples()` → 8-12 个采样窗口 + 拼接 |
| 前奏分析 | 仅需开头节拍 | `analyzePodcastDjIntro()` → 前 90-240 秒分析 |

**输出节拍网格属性**:

| 属性 | 含义 |
|------|------|
| `time` | 节拍时间戳 |
| `impact` | 冲击力 (0-1) |
| `strength` | 节拍强度 (0-1) |
| `combo` | 网格位置 (downbeat/push/drop/rebound/accent) |
| `low` / `body` / `snap` | 低频/中频/瞬态分量 |
| `mass` / `sharpness` | 质量感/锐度 |
| `camera` | 是否触发相机动作 |

---

### 4.5 前端 SPA (`public/index.html`)

**文件规模**: ~27,000 行，~1.3 MB

**架构特点**: 无框架、无构建工具、无模块系统的纯手工 SPA。所有 HTML/CSS/JS 内联于单一文件，以注释分隔区块。

#### 4.5.1 HTML 层级结构 (z-index 分层)

```
z-0     #custom-bg, #album-bg          背景层（自定义壁纸/模糊封面）
z-1     #canvas-container               Three.js WebGL 粒子画布
z-10    #search-area                    顶部搜索栏
z-50    #thumb-wrap, #empty-home       封面信息/首页壁纸
z-60    #fx-panel, #playlist-panel     视觉控制面板/歌单面板（左右侧滑）
z-60    #bottom-bar                     底部播放器控制台
z-70    #stage-lyrics                   3D 歌词舞台
z-100   #toast, #visual-guide          通知/新手引导
z-200   .modal-mask                     模态弹窗层
z-300   #splash                         启动动画
z-500   #desktop-titlebar              桌面壳标题栏
```

#### 4.5.2 Three.js 3D 渲染管道

```
┌─────────────────────────────────────────────────────┐
│                  requestAnimationFrame               │
│                    animate() 主循环                   │
├─────────────────────────────────────────────────────┤
│  1. 读取音频频谱 (AnalyserNode)                      │
│     ├── bass (60-150Hz, bins 3-7)                   │
│     ├── mid (300-6000Hz)                            │
│     └── treble (6000Hz+)                            │
│                                                      │
│  2. 更新相机                                         │
│     ├── orbit 模式（鼠标拖拽旋转/缩放）               │
│     ├── cinema 模式（节拍驱动电影运镜）               │
│     └── freeCamera 模式（手势/键盘自由飞行）          │
│                                                      │
│  3. 更新粒子系统                                     │
│     ├── 主粒子层 (particles): 5 种视觉预设           │
│     │   ├── Silk (0):  丝绸/流体                     │
│     │   ├── Tunnel (1): 隧道/穿梭                    │
│     │   ├── Orbit (2):  行星/轨道                    │
│     │   ├── Void (3):   虚空/黑洞                    │
│     │   └── Vinyl (4):  黑胶唱片                     │
│     ├── 溢光粒子层 (bloomParticles)                  │
│     ├── 漂浮粒子层 (floatGroup, ~1,300 粒子)         │
│     ├── 背面封面层 (backCoverGroup, ~3,000 粒子)     │
│     └── 骷髅点云 (skullParticleGroup)                │
│                                                      │
│  4. 更新 3D 歌单架                                   │
│     ├── 侧面模式 (side shelf)                        │
│     ├── 舞台模式 (stage shelf)                       │
│     ├── 常驻/选中/悬浮动画                           │
│     └── 详情页列表                                   │
│                                                      │
│  5. 更新 3D 歌词舞台                                 │
│     ├── Canvas → Texture → THREE.Plane               │
│     ├── 歌词进度跟踪                                  │
│     └── 世界轴绑定（跟随封面粒子旋转）                │
│                                                      │
│  6. 自适应渲染质量                                    │
│     ├── 像素比 (DPR): 0.56 - 1.75                    │
│     ├── 帧率等级: 1fps(深度休眠) / 30fps / 60fps     │
│     └── 渲染分辨率: 根据视口 + 性能预算动态调整       │
└─────────────────────────────────────────────────────┘
```

#### 4.5.3 5 种视觉预设 (单一 GLSL Vertex Shader)

所有预设共享一个顶点着色器，通过 `uPreset` uniform 分支：

```glsl
// 简化示意 (~250行完整shader)
if (uPreset == 0) { /* Silk - 3D simplex noise + 丝绸流体位移 */ }
if (uPreset == 1) { /* Tunnel - 圆柱投影 + 隧道穿梭 */ }
if (uPreset == 2) { /* Orbit - 球面轨道 + 行星环 */ }
if (uPreset == 3) { /* Void - 径向扭曲 + 黑洞吸积 */ }
if (uPreset == 4) { /* Vinyl - 平面旋转 + 唱片纹理 */ }
```

#### 4.5.4 相机系统

```
orbit 对象
├── userTheta / userPhi / userRadius   用户手动控制参数
├── cineTheta / cinePhi / cineRadius   电影镜头自动参数
├── focus                              焦点偏移
├── shake                              震动强度
└── 模式切换
    ├── 静态模式 (static): 固定角度 -15°
    ├── 动态模式 (dynamic): 自动微动 0°
    ├── 电影模式 (cinema): 节拍驱动运镜
    └── 自由模式 (freeCamera): 键盘/手势全自由
```

#### 4.5.5 前端核心功能模块

| 模块 | 行号范围 | 功能 |
|------|----------|------|
| 全局状态声明 | ~2674 | 所有 var 全局变量 |
| Three.js 场景初始化 | ~3716 | scene/camera/renderer |
| 相机系统 v7.1 | ~3784 | orbit/freeCamera/cinema |
| 指针/拖拽控制 | ~5495 | 鼠标交互/射线检测 |
| 粒子系统 (5 预设) | ~5696 | 顶点着色器/粒子几何体 |
| 漂浮粒子层 | ~6401 | 背景漂浮粒子 |
| 背面封面层 | ~7059 | 封面背粒子 |
| 舞台歌词 v9 | ~7201 | 3D 歌词渲染 |
| 波纹触发系统 | ~9360 | 低音波纹效果 |
| 离线节拍预解析 v7.2 | ~9948 | OfflineAudioContext 节拍检测 |
| 3D 歌单架 | ~12755 | 3D 卡片布局/交互 |
| 歌单详情管理器 | ~13921 | 歌单曲目列表 |
| API 辅助 | ~15079 | HTTP 请求/工具函数 |
| 搜索 | ~17181 | 搜索栏/历史/结果 |
| 音频上下文/频谱 | ~17725 | Web Audio 初始化/分析 |
| 播放控制 | ~18217 | 播放/暂停/切歌 |
| 歌词系统 | ~19014 | YRC/LRC 解析/同步 |
| 文件拖放 | ~19985 | 本地文件播放 |
| 用户存档 | ~20081 | 视觉预设存取 |
| 登录系统 | ~23157 | QR 登录/状态管理 |
| 首页/闲置引导 | ~23950 | 壁纸/空闲动画 |
| 动态库加载 | ~24735 | 按需加载脚本 |
| 手势追踪 v8 | ~24748 | MediaPipe 手势 |
| 快捷键系统 | ~25149 | 键盘快捷键 |
| UI Peek/Hide v8 | ~25233 | 面板滑入滑出 |
| 启动动画 | ~25516 | WebGL Splash |
| 主循环 | ~26496 | requestAnimationFrame |

#### 4.5.6 状态管理

**localStorage 键** (所有以 `mineradio-` 为前缀):

| Key | 类型 | 用途 |
|-----|------|------|
| `diy-player-mode-v1` | string | DIY 模式开关 |
| `custom-covers` | JSON | 自定义专辑封面 |
| `custom-lyrics-v1` | JSON | 自定义歌词 |
| `lyric-layout-v1` | JSON | 3D 歌词布局持久化 |
| `playback-quality-v1` | string | 播放音质 |
| `local-beatmaps-v1` | JSON | 本地节拍缓存 |
| `listen-stats-v1` | JSON | 听歌统计 |
| `search-history` | JSON | 搜索历史 |
| `user-fx-archives-v1` | JSON | 用户视觉存档 |
| `hotkey-settings-v1` | JSON | 自定义快捷键 |
| `free-camera-v1` | JSON | 自由相机位置 |
| `weather-city` | string | 天气电台城市 |

**CSS Body Class 状态机**: 通过切换 `<body>` 的 class 控制 UI 模式，包括 `splash-active`, `diy-mode`, `desktop-shell`, `immersive-mode`, `idle-guide-on`, `visual-guide-active`, `empty-home-active` 等。

---

### 4.6 叠加窗口

#### `desktop-lyrics.html` (桌面歌词)

- 无框透明置顶窗口
- 鼠标穿透（锁定态）+ 中键切换锁定
- PowerShell 全局中键轮询（`GetAsyncKeyState(4)`）
- 歌词投影效果：`drop-shadow` + `-webkit-text-stroke` 细白描边

#### `wallpaper.html` (壁纸窗口)

- 全屏覆盖 WorkerW 层
- 银河/星空背景渲染
- 鼠标事件全部穿透到桌面

---

## 5. 数据流

### 5.1 音乐播放流程

```
用户搜索歌曲
  → /api/search?keywords=xxx (网易云)
  → /api/qq/search?keywords=xxx (QQ音乐)
  → 返回歌曲列表 (id, name, artist, album, cover)
  → 用户点击播放
  → /api/song/url?id=xxx&quality=xxx (获取播放URL)
  → 创建/复用 <Audio> 元素
  → MediaElementAudioSourceNode → AnalyserNode → GainNode → destination
  → 频谱数据 → Three.js uniform 更新 → 粒子动画
  → 同时: 歌词获取 /api/lyric → 舞台歌词渲染
  → 同时: 节拍检测 → 相机电影镜头运镜
```

### 5.2 登录状态流

```
网易云登录:
  前端请求 → ipcRenderer.invoke('netease-music-open-login')
  → Electron 主进程: 创建独立 BrowserWindow(session partition)
  → 加载 music.163.com → 用户扫码
  → Cookie 轮询(1.2s间隔) → 检测到 MUSIC_U
  → 写入 .cookie 文件 + IPC 返回
  → 前端更新 loginStatus

QQ 登录:
  同上流程，检测 qm_keyst/qqmusic_key 等播放票据
  → 写入 .qq-cookie 文件
```

### 5.3 更新流程

```
定时检查 → /api/update/latest
  → GitHub Releases API → 比较版本号
  → 有新版本: 前端显示更新入口
  → 用户点击更新
  → 优先尝试快速补丁: /api/update/patch
  → 补丁失败或不可用: /api/update/download (完整安装包)
  → 多镜像下载(ghproxy等国内分流)
  → sha256/digest 校验
  → ipcRenderer.invoke('mineradio-open-update-installer')
  → shell.openPath() 打开安装包
```

---

## 6. 外部依赖与服务

| 服务 | 用途 | 通信方式 |
|------|------|----------|
| 网易云音乐 API | 搜索/播放/歌单/登录/播客/评论 | `NeteaseCloudMusicApi` npm 包 (HTTP) |
| QQ 音乐 API | 搜索/播放/歌单/登录 | 自定义 HTTP 请求 |
| GitHub Releases API | 版本检查/更新下载 | HTTPS REST API |
| Open-Meteo | 天气预报/地理编码 | HTTPS REST API (免费, 无需 Key) |
| ip-api.com | IP 地理位置 | HTTP REST API (免费) |
| 专辑封面 CDN | 封面图片代理 | /api/cover HTTP 代理 |
| 音频 CDN | 音频流代理 | /api/audio HTTP 代理 (Range 支持) |
| GitHub 镜像 | 国内更新下载加速 | gh.llkk.cc / ghfast.top / gh-proxy.com |

---

## 7. 构建与分发

### 7.1 开发命令

```bash
npm install          # 安装依赖
npm start            # 启动 Electron 开发模式
node --check server.js  # 语法检查
```

### 7.2 构建命令

```bash
npm run build:win        # 完整 NSIS 安装包构建
npm run build:win:dir    # 仅打包到 win-unpacked 目录
```

### 7.3 构建产物

- `dist/Mineradio-{version}-Setup.exe` — NSIS 安装包
- `dist/Mineradio-{version}-Setup.exe.blockmap` — 增量更新块映射
- `dist/latest.yml` — electron-builder 更新清单
- `dist/Mineradio-{from}-to-{to}.patch.json` — 快速补丁

### 7.4 打包配置

- **asar: false** — 不打包为 asar 归档，保持文件可直接访问
- **安装器**: NSIS，自定义深色 UI（中文极简黑白蓝风格）
- **默认安装路径**: 优先 `D:\Mineradio`
- **自动更新**: GitHub Releases + latest.yml + 镜像分流

### 7.5 安装器安全规则

- 安装路径强制规范化到独立 `Mineradio` 子目录
- 非 Mineradio-owned 目录阻止安装（`.mineradio-install-root` 标记）
- 卸载器仅删除已知 Mineradio/Electron 文件，不递归删除

---

## 8. 关键设计决策

### 8.1 为什么是单文件 SPA

- **无构建工具链**: 避免 webpack/vite 配置复杂度
- **快速迭代**: 修改后直接刷新，无需等待编译
- **Electron 友好**: `asar: false` 下文件直接可读，方便运行时补丁更新
- **代价**: 文件巨大 (~27,000 行)，需要 `rg` 精确定位修改

### 8.2 为什么用纯 http.createServer 而非 Express

- **最小依赖**: 避免额外的框架依赖
- **完全控制**: 手动路由分发无中间件开销
- **Electron 内嵌**: 服务器仅服务于本地单用户，无需 Express 的企业级功能

### 8.3 性能策略

- **自适应像素比**: 根据视口大小动态调整 DPR，保持 ~5.2M 像素预算
- **帧率等级**: 正常 60fps / 省电 30fps / 后台 1fps / 深度休眠 0fps (隐藏 canvas)
- **窗口状态驱动**: 最小化/不可见时释放 GPU 资源，但失焦可见窗口保持全帧率
- **节拍缓存**: 磁盘持久化避免重复分析长音频
- **直播后台保持**: 可选开关，阻止后台降载

### 8.4 玻璃质感设计

播放器控制台使用 SVG `feColorMatrix` + `feComponentTransfer` 生成位移贴图，配合 `backdrop-filter: blur()` 实现独特的玻璃质感。相关参数保存在 `docs/design/GLASS_SVG_TEXTURE.md`，标记为"黄金版本"，不可回退到普通毛玻璃。

---

## 9. 文件清单

| 文件 | 大小 | 行数 | 职责 |
|------|------|------|------|
| `server.js` | 164 KB | 4,203 | 后端 API 服务 |
| `public/index.html` | 1.3 MB | ~27,000 | 前端 SPA |
| `desktop/main.js` | 50 KB | 1,467 | Electron 主进程 |
| `dj-analyzer.js` | 34 KB | 865 | 离线节拍分析 |
| `desktop/preload.js` | 3.2 KB | 53 | 主窗口 preload |
| `desktop/overlay-preload.js` | 1.2 KB | 19 | 叠加窗口 preload |
| `public/desktop-lyrics.html` | 55 KB | — | 桌面歌词页 |
| `public/wallpaper.html` | 6.0 KB | — | 壁纸页 |
| `build/installer.nsh` | 24 KB | — | NSIS 安装脚本 |
| `build/after-pack.js` | 2.7 KB | — | 打包后处理 |
| `package.json` | 2.4 KB | 96 | 项目配置 |

---

## 10. 扩展点与未来方向

根据 `docs/history/PROJECT_MEMORY.md` 中的规划：

1. **多音乐接口热插拔方案** — 将网易云/QQ 抽为 Provider 注册表，支持酷狗/汽水/Apple Music/Spotify
2. **情绪节奏音效大师** — 自研本地音频处理引擎（EQ/压缩/饱和/宽度），输出情绪参数驱动电影镜头
3. **壁纸模式重构** — 透明玻璃模式 → WorkerW 壁纸视觉层 → MyDockFinder 避让 → Wallpaper Engine 联动
4. **Ctrl 缩放修复** — 处理 Electron `per_host_zoom_levels` 残留问题
5. **搜索排序优化** — 优先原唱/官方版本

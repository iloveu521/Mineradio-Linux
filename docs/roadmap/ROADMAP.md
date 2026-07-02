# Mineradio 路线图

> 最后更新: 2026-07-01 | 当前版本: v1.1.1

---
# Current Sprint

Current Sprint

v0.1.0-draft

Current Phase

Phase0 Documentation and enviromnet prep

Status

In Progress

Next

Phase1

---

## 近期计划 (P0-P1)

### P0 — 安装器/卸载器安全

- [x] 安装器路径强制规范化到独立 `Mineradio` 子目录
- [x] 卸载器仅删除已知 Mineradio/Electron 文件，不递归删除
- [x] 默认安装路径优先 `D:\Mineradio`，D 不存在再 E/F/.../Z
- [x] 非 Mineradio-owned 目录阻止安装（`.mineradio-install-root` 标记）
- [ ] 同版本覆盖安装规则完善（v1.1.1 已部分完成）

### P0 — QQ 音乐登录与播放

- [ ] QQ-only 登录时弹"未登录，仅试听"问题修复（前置 P0）
- [ ] QQ 音乐播放票据续期与失效处理优化

### P1 — 搜索排序优化

- [ ] 搜索结果优先原唱/官方版本，避免翻唱排第一
- [ ] 示例："日落大道"应优先梁博，"Beauty and a Beat"应优先 Justin Bieber

### P1 — Ctrl 缩放卡住修复

- [ ] 处理 Electron `per_host_zoom_levels` 残留问题
- [ ] 覆盖 `Ctrl+=`、`Ctrl+Shift+=`、`Ctrl+NumpadAdd`、`Ctrl+NumpadSubtract`、`Ctrl+0`
- [ ] 清理旧用户 Preferences 中的 `per_host_zoom_levels` 残留键值

---

## 中期计划 (P2-P3)

### P2 — 多音乐接口热插拔方案

相关设计文档: `docs/design/MUSIC_PROVIDER_PLUGIN_PLAN.md` (待创建)

- [ ] 将当前网易云/QQ 音乐硬编码分支抽为 Provider 注册表
- [ ] Provider 类型: `direct-url` (返回可播放 URL) / `sdk-player` (需内嵌 SDK)
- [ ] 优先接入: 酷狗 → 汽水
- [ ] 后期接入: Apple Music (SDK) / Spotify (SDK)
- [ ] 不承诺所有平台都能返回可播放直链 URL
- [ ] 不要让网易云登录态成为其他 Provider 的播放前置条件

### P2 — 情绪节奏音效大师

相关设计文档: `docs/design/MOOD_RHYTHM_SOUND_MASTER.md` (待创建)

- [ ] Phase 1: 分析层 + UI 状态展示 + 保守 EQ/压缩
  - 自研本地引擎，分析 BPM、鼓点置信度、kick/snare/onset、能量曲线、段落变化
  - 输出情绪节奏参数: energy/aggression/groove/space/brightness/warmth/stability
  - WebAudio EQ/动态压缩/限幅，默认"自动·轻微"
  - 原声 A/B 对比、一键关闭、音量匹配、防削波
- [ ] Phase 2: 情绪参数驱动电影镜头
  - 电子歌偏 kick 锁拍，摇滚偏军鼓/段落爆发，阴郁歌偏慢推镜和粒子呼吸
- [ ] 不依赖网易云私有音效模型
- [ ] CPU 上限保护，失败回退原声

### P3 — 壁纸模式与桌面融合

相关设计文档: `docs/design/WALLPAPER_ENGINE_DESKTOP_FUSION_PLAN.md` (待创建)

- [ ] Phase 1: 透明玻璃模式 MVP
  - 主窗口透明穿透，桌面图标/任务栏不被遮挡
- [ ] Phase 2: WorkerW 壁纸视觉层重构
  - 独立控制台浮层，不可拖动隐藏
- [ ] Phase 3: MyDockFinder 自动探测/手动安全区
- [ ] Phase 4: Wallpaper Engine 轻联动 + 本地桥接深联动
  - 输出独立 Web 壁纸包
- [ ] 不要把 Wallpaper Engine 当作 Electron 容器
- [ ] 不要把播放器黄金版 SVG 玻璃质感改成普通毛玻璃

---

## 长期计划 (P4+)

### 3D 歌单架交互优化

- [ ] 悬停展开和点击后可用状态之间更丝滑
- [ ] Home 页面与后方 3D 歌单架的交互穿透问题
- [ ] 详情页打开后的位置/层级精确控制

### Linux 平台适配 ✅ 已完成

> 详细设计文档: [`docs/design/LINUX_DESIGN.md`](../design/LINUX_DESIGN.md)
> 完成版本: `v1.2.0`

- [x] Phase 0 — 文档与准备 → `v0.1.0`
- [x] Phase 1 — GPU 渲染修复 (use-angle d3d11) → `v0.2.0`
- [x] Phase 2 — 节拍缓存路径修复 (XDG) → `v0.3.0`
- [x] Phase 3 — 窗口图标修复 (ICO→PNG) → `v0.4.0`
- [x] Phase 4 — 构建系统配置 (AppImage + deb) → `v0.5.0`
- [x] Phase 5 — 集成测试 (X11) → `v0.6.0`
- [x] Phase 6 — 文档收尾 → `v1.2.0`

已知限制: Wayland 不完全兼容、桌面歌词中键锁定不可用、壁纸模式无 WorkerW 等价物、无自动更新。

---

## 短期优化计划 🚧 v1.3.0

> 目标: 修复 Linux 桌面环境下的体验问题，优化交互细节

- [ ] **Item 1** — UI Linux 桌面环境兼容优化
  - X11/Wayland 环境检测（`XDG_SESSION_TYPE`）
  - Wayland 下给出兼容性提示
  - 排查 `requestAnimationFrame` 与 X11 compositor vsync 的卡顿原因
  - 文件：`desktop/main.js`
- [ ] **Item 2** — 左侧歌单交互优化
  - 歌单出现时卡顿排查
  - 鼠标移走后延迟消失问题（缩短 hide timeout）
  - 文件：`public/index.html`
- [ ] **Item 3** — 桌面歌词显示逻辑优化
  - 窗口可见 → 自动关闭桌面歌词
  - 窗口最小化/关闭 → 自动显示桌面歌词
  - 其余时刻显示在最上层
  - 文件：`desktop/main.js`
- [ ] **Item 4** — 音量按钮滚轮调节
  - 鼠标悬停音量按钮时，滚轮上下调节音量
  - 文件：`public/index.html`
- [ ] **Item 5** — 进度条拖动响应优化
  - 拖动进度条时响应慢、不跟手
  - 减少 seek debounce 或检查事件处理链路
  - 文件：`public/index.html`
- [ ] **Item 6** — 播放器控制台桌面歌词按钮
  - 在 `#bottom-bar` 控制区添加桌面歌词开关按钮
  - 按钮状态反映桌面歌词开/关
  - 文件：`public/index.html`

---

## 已完成

- [x] v1.1.1 — P0 安装器/卸载器安全规则完善（`build/installer.nsh`）
- [x] v1.1.0 — 纯净安装发布版，旧安装包隔离
- [x] 默认测试作为首次启动默认用户存档
- [x] 3D 歌单架: 控制台模式、常驻/静态镜头、详情页层级、歌词避让
- [x] 3D 歌单架: 滚动选择音、滚轮热区、内容开关、动态/静态绑定边界
- [x] 歌词: 绑定封面粒子世界轴、歌单详情页透明度边界
- [x] 高级性能设置: 进入本地存档、重启保留
- [x] 用户存档应用: 必须提交播放态视觉预设
- [x] 桌面歌词: 白底/黑底可读视觉效果
- [x] 封面清晰度控制台滑块 + 粒子割裂线修复
- [x] Splash WebGL 开场动画 + "点击进入"交互
- [x] Home 页面视觉升级（唱片/封面套/频谱块）
- [x] 安装器中文极简黑白蓝格式
- [x] 更新系统: 完整安装包 + 快速补丁 + 国内镜像分流
- [x] QQ 音乐: 搜索、登录态、播放票据、歌单、歌词
- [x] 桌面歌词: 透明置顶、鼠标穿透、中键锁定
- [x] 播客/DJ 离线节拍分析（`dj-analyzer.js`）

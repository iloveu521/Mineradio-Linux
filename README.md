# Mineradio Linux

基于 [Mineradio](https://github.com/XxHuberrr/Mineradio) 的 Linux 桌面音乐播放器——融合天气电台、歌词舞台、粒子视觉和 3D 歌单架。

English version: [README_EN.md](./README_EN.md)

## System Requirements

| 依赖 | 最低版本 |
|------|---------|
| Node.js | >= 22.12.0 |
| npm | >= 10.0.0 |
| 桌面环境 | X11 (Wayland 部分兼容) |

## Quick Start

```bash
# 安装依赖
npm install

# 启动应用
npm start

# 无桌面环境测试（可选）
npm start -- --no-sandbox
```

## Build

```bash
# 构建 AppImage + deb 安装包
npm run build

# 仅构建未打包目录（调试用）
npm run build:dir
```

构建产物位于 `dist/`：
- `dist/Mineradio-1.2.0.AppImage` — 通用 Linux 可执行文件
- `dist/mineradio_1.2.0_amd64.deb` — Debian/Ubuntu 安装包

## Install

### AppImage

```bash
chmod +x Mineradio-*.AppImage
./Mineradio-*.AppImage
```

### Debian/Ubuntu

```bash
sudo dpkg -i mineradio_*_amd64.deb
```

## Sandbox Configuration

Linux 下 Electron 需要配置 chrome-sandbox 权限：

```bash
sudo chown root:root node_modules/electron/dist/chrome-sandbox
sudo chmod 4755 node_modules/electron/dist/chrome-sandbox
```

开发调试时可临时使用 `--no-sandbox`（不推荐生产环境）。

## Features

- Open-Meteo 天气电台，根据当前位置和天气生成播放列表
- 网易云音乐搜索、播放、歌单、登录（QR 扫码）
- QQ 音乐搜索、播放、登录
- 5 种 3D 粒子视觉预设（Silk / Tunnel / Orbit / Void / Vinyl）
- 基于音频节拍的实时电影镜头运镜
- 3D 歌词舞台
- 3D 歌单架（播放列表 3D 可视化）
- 桌面歌词叠加窗口
- 自定义专辑封面与歌词
- 用户视觉存档导入/导出

## Known Limitations on Linux

- **Wayland**: 透明窗口渲染和 `globalShortcut` 不完全支持，推荐 X11 或 `--ozone-platform=x11`
- **桌面歌词中键锁定**: Linux 无全局中键检测方案，需通过 UI 切换
- **壁纸模式**: 无 Windows WorkerW 等价机制，壁纸为普通全屏窗口
- **自动更新**: AppImage 不支持 electron-builder 自动更新，需手动下载新版本

## Project Structure

```
Mineradio-Linux/
├── desktop/               # Electron main/preload
├── public/                # Web 前端 (Three.js SPA)
├── server.js              # HTTP API 服务（网易云/QQ音乐代理）
├── dj-analyzer.js         # 离线音频节拍分析
├── build/                 # 图标与构建资源
├── docs/                  # 项目文档
│   ├── architecture/      # 系统架构
│   ├── design/            # 设计规范
│   ├── roadmap/           # 路线图
│   ├── history/           # 项目记忆
│   └── security/          # 安全记录
└── package.json
```

## Development

遵循 `CLAUDE.md` 规范。新对话优先阅读：

1. `AGENTS.md` — 项目规则
2. `docs/architecture/ARCHITECTURE.md` — 系统架构
3. `docs/roadmap/ROADMAP.md` — 当前进度

## License

GPL-3.0. 详见 [LICENSE](./LICENSE).

## Credits

原始项目 [Mineradio](https://github.com/XxHuberrr/Mineradio) 由 XxHuberrr 设计与开发。

Linux 迁移: [Silly](https://github.com/iloveu521)

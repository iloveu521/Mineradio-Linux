# Mineradio Linux

A Linux desktop music player based on [Mineradio](https://github.com/XxHuberrr/Mineradio), combining weather radio, lyric stage, particle visuals, and 3D playlist shelf into an immersive music experience.

[中文版](./README.md)

## System Requirements

| Dependency | Minimum Version |
|------------|----------------|
| Node.js | >= 22.12.0 |
| npm | >= 10.0.0 |
| Desktop | X11 (Wayland partially supported) |

## Quick Start

```bash
# Install dependencies
npm install

# Launch app
npm start

# Headless environment (optional)
npm start -- --no-sandbox
```

## Build

```bash
# Build AppImage + deb packages
npm run build

# Build unpacked directory only (debug)
npm run build:dir
```

Build outputs in `dist/`:
- `dist/Mineradio-1.2.0.AppImage` — Universal Linux executable
- `dist/mineradio_1.2.0_amd64.deb` — Debian/Ubuntu package

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

Electron on Linux requires the chrome-sandbox helper:

```bash
sudo chown root:root node_modules/electron/dist/chrome-sandbox
sudo chmod 4755 node_modules/electron/dist/chrome-sandbox
```

For development, use `--no-sandbox` temporarily (not recommended for production).

## Features

- Open-Meteo weather radio — generates playlists based on location and weather
- Netease Cloud Music — search, playback, playlists, QR login
- QQ Music — search, playback, login
- 5 particle visual presets (Silk / Tunnel / Orbit / Void / Vinyl)
- Real-time beat-driven cinematic camera
- 3D stage lyrics
- 3D playlist shelf (playlist visualization)
- Desktop lyrics overlay
- Custom album covers and lyrics
- Visual effect preset import/export

## Known Limitations on Linux

- **Wayland**: transparent window rendering and `globalShortcut` have limited support; use X11 or `--ozone-platform=x11`
- **Desktop lyrics middle-click lock**: no cross-platform global middle-click detection on Linux
- **Wallpaper mode**: no WorkerW equivalent on Linux; wallpaper renders as a regular fullscreen window
- **Auto-update**: AppImage does not support electron-builder auto-update; manual download required

## Project Structure

```
Mineradio-Linux/
├── desktop/               # Electron main/preload
├── public/                # Web frontend (Three.js SPA)
├── server.js              # HTTP API server (Netease/QQ proxy)
├── dj-analyzer.js         # Offline beat analysis
├── build/                 # Icons and build resources
├── docs/                  # Documentation
│   ├── architecture/      # System architecture
│   ├── design/            # Design specifications
│   ├── roadmap/           # Roadmap
│   ├── history/           # Project memory
│   └── security/          # Security records
└── package.json
```

## Development

Follow `CLAUDE.md` conventions. Start by reading:

1. `AGENTS.md` — Project rules
2. `docs/architecture/ARCHITECTURE.md` — System architecture
3. `docs/roadmap/ROADMAP.md` — Current progress

## License

GPL-3.0. See [LICENSE](./LICENSE).

## Credits

Original [Mineradio](https://github.com/XxHuberrr/Mineradio) designed and developed by XxHuberrr.

Linux port by [Silly](https://github.com/iloveu521)

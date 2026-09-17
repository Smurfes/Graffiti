# Graffiti (formerly InteractWall) - Project Context

## Overview
Graffiti is a high-performance interactive wallpaper engine for Windows. It consists of a React/Tauri frontend (UI) and a C++/DirectX11 backend (Renderer) that draws directly to the Windows desktop behind icons.

## Current State (Commit 4fd5fba + Fixes)
The project is currently in a stable state with **6 working C++ effect plugins**:
1. **Cursor Reveal**
2. **Gravity Lens**
3. **Gravity Lens - Transparent**
4. **Depth Parallax**
5. **Brick Outline**
6. **Space Ball (Stone Press V2)**

*Note: The Web Wallpaper (Three.js/WebView2) integration was temporarily reverted and backed up to `C:\My_Proj\WebWallpaper_Backup` to isolate and fix stability issues with the core C++ effects.*

## Recent Fixes & Adjustments
1. **PowerManager Idle Bug Fix**: Fixed an issue where effects would render for half a second and disappear (0% GPU). The PowerManager's idle timer was not being reset when interacting with the Tauri UI. We added `PowerManager::OnMouseMove()` to the `set_effect` and `apply_wallpaper` IPC handlers in `main.cpp` to ensure the renderer wakes up.
2. **Space Ball Tuning**: Removed the default color-fade/dome highlight by setting `depthDarkening` and `directionalShading` defaults to `0.0` in both `Effects.tsx` and `StonePressV2Plugin.cpp`.
3. **Gravity Lens Tuning**: Fixed the default `lensRadius` to `0.08` (matching the Transparent version) and adjusted the UI slider range to `0.02 - 0.25` so it can be properly scaled down.

## Architecture
- **Frontend (`ui/`)**: React + Vite + Tauri. Handles user settings, effect selection, and wallpaper gallery management. Communicates with the renderer via a local IPC pipe (`\\.\pipe\GraffitiIPC`).
- **Backend (`renderer/`)**: C++ / DirectX 11. 
  - `main.cpp`: Manages the application lifecycle, IPC server, and DirectX rendering loop. Uses `WorkerW` to draw behind desktop icons.
  - `PluginLoader.cpp`: Dynamically loads `.dll` effect plugins.
  - `PowerManager.cpp`: Monitors system state (fullscreen apps, battery, idle time) to pause rendering and save resources.
  - `QualityManager.cpp`: Controls FPS caps (30/60/Uncapped) and internal resolution scaling based on hardware detection and user settings.

## Build Instructions
- **Renderer**: Run `build.bat` in the `C:\My_Proj\InteractWall` directory from a standard command prompt (it automatically sets up the MSVC environment).
- **UI / Setup Files**: Run `npm run tauri build` in the `ui` directory to generate the `.exe` and `.msi` installers in `ui/src-tauri/target/release/bundle/`.

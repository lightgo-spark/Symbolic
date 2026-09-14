# InfinityFlow

**A local-first Mermaid diagram editor for Windows.** Write diagrams as text and see them render live, in a single lightweight desktop app built with Rust + WebView2.

English 

- **Version:** 0.6.0
- **License:** MIT
- **Platform:** Windows 10/11 (x64)

## Overview

InfinityFlow is a desktop editor for [Mermaid.js](https://mermaid.js.org/) diagrams. Mermaid v11 is bundled inside the executable, so the app works fully offline and your files never leave your machine. The only network requests the app can make are the optional AI-assistant calls you explicitly trigger.

## Features

### Editor & live preview

- Mermaid code editor with syntax highlighting, line numbers, and configurable indentation (1-8 spaces)
- Live preview that re-renders automatically (~0.4 s after typing) with a render-time indicator
- Error panel with the failing line's syntax message
- Zoom 25-400%, fit-to-width, `Ctrl`+wheel zoom, middle-button pan
- Undo/redo, word wrap, bracket auto-pairing, duplicate/delete/comment line
- Find & replace, go-to-line, and cursor position (Ln/Col) in the status bar

### Documents

- Multi-tab editing, each tab with its own undo history
- Draft restore: unsaved work is restored on the next launch
- Optional auto-save to the opened file
- Recent files (up to 10) and drag & drop file opening
- Version history: a `.bak` snapshot is written before each save (last 20 per file); auto-save snapshots are throttled to once per 10 minutes

### Templates & snippets

- 9 starter templates: Flowchart, Sequence, Class, State, Gantt, Pie, ER, GitGraph, Timeline
- Template gallery with search
- 170+ built-in examples covering 30+ diagram types
- 280+ insertable syntax snippets across 30+ categories
- Custom snippets with JSON import/export, favorites, and recents

### Export

- SVG, PNG (2x), PDF, and standalone HTML
- Copy PNG to clipboard, print
- All exports are produced locally

### Optional AI assistant

Generate or refine diagrams from a natural-language prompt (`Ctrl+Shift+A`).

| Provider  | Default model      | Notes                |
| --------- | ------------------ | -------------------- |
| DeepSeek  | `deepseek-chat`    | API key required     |
| OpenAI    | `gpt-4o-mini`      | API key required     |
| Anthropic | `claude-sonnet-4-5`| API key required     |
| Gemini    | `gemini-2.5-flash` | API key required     |
| Ollama    | `llama3.2`         | local, no key needed |
| LM Studio | `local-model`      | local, no key needed |
| Custom    | -                  | any OpenAI-compatible endpoint |

- Per-provider API keys, custom model, temperature, max tokens, and system prompt
- Diagram-type hint (30 types) and "use current code as context"
- Requests are capped at 20,000 characters per prompt and 4 concurrent calls, and only run when you press Generate/Test

### Windows integration

- NSIS installer with Start menu and desktop shortcuts
- `.mmd` / `.mermaid` file associations
- Open a file from Explorer or the command line: `InfinityFlow.exe diagram.mmd`

## Requirements

- Windows 10/11
- [Microsoft Edge WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/) - preinstalled on Windows 11 and most up-to-date Windows 10 systems. If it is missing, the app shows a download link, and the installer can bundle the offline bootstrapper.
- **Building from source:** Rust 1.85+ (edition 2024) with the MSVC toolchain

## Build & run

```powershell
cargo run --release        # build and launch
cargo test                 # run the test suite
cargo build --release      # produce target\release\InfinityFlow.exe
```

The UI (`assets/index.html`, `assets/app.js`) and Mermaid (`assets/mermaid.min.js`) are embedded into the executable at compile time, so the release build is a single self-contained `.exe`.

### Installer

```powershell
powershell -ExecutionPolicy Bypass -File installer\build.ps1
```

The script runs the tests, builds the release binary, optionally signs it, and packages `dist\InfinityFlow-<version>-setup.exe` with NSIS 3.x (`makensis` must be on `PATH`; if it is not found, the app exe is still produced).

- Code signing (optional): set `MF_CERT_PATH` and `MF_CERT_PASSWORD`.
- Offline WebView2 bootstrap (optional): place `MicrosoftEdgeWebView2Setup.exe` in `installer\` and build with `makensis /DWEBVIEW2_BOOTSTRAPPER installer\infinity-flow.nsi`.

## Keyboard shortcuts

| Shortcut                                        | Action                                  |
| ----------------------------------------------- | --------------------------------------- |
| `Ctrl+Enter`                                    | Render now                              |
| `Ctrl+O` / `Ctrl+S` / `Ctrl+Shift+S`            | Open / Save / Save as                   |
| `Ctrl+T` / `Ctrl+W` / `Ctrl+Tab` / `Ctrl+1...9` | New tab / Close tab / Next tab / Tab N  |
| `Ctrl+F` / `Ctrl+H` / `Ctrl+G`                  | Find / Replace / Go to line             |
| `Ctrl+K`                                        | Command palette                         |
| `Ctrl+/`                                        | Examples helper                         |
| `Ctrl+I`                                        | Insert syntax                           |
| `Ctrl+Shift+A`                                  | AI generate                             |
| `Ctrl+B` / `Ctrl+Shift+F`                       | Toggle sidebar / Fit to width           |
| `Ctrl+=` / `Ctrl+-` / `Ctrl+0`                  | Zoom in / Zoom out / Reset zoom         |
| `Alt+Z` / `Alt+C`                               | Toggle word wrap / Toggle comment       |
| `Ctrl+Shift+D` / `Ctrl+Shift+K`                 | Duplicate line / Delete line            |
| `Ctrl+E` / `Ctrl+Shift+E` / `Ctrl+Shift+P`      | Export SVG / PNG (2x) / PDF             |
| `Ctrl+P`                                        | Print                                   |
| `Esc`                                           | Close dialog                            |

## Data & privacy

InfinityFlow is local-first: there is no telemetry, and no network request is made unless you run an AI feature.

| What                                                  | Where                                                          |
| ----------------------------------------------------- | -------------------------------------------------------------- |
| AI configuration and API keys                         | `%LOCALAPPDATA%\InfinityFlow\llm.json` (plain JSON - protect it) |
| Version-history backups                               | `%LOCALAPPDATA%\InfinityFlow\history\<file-hash>\*.bak`        |
| Diagnostics log                                       | `%LOCALAPPDATA%\InfinityFlow\ipc.log` (capped at 1 MB)         |
| UI preferences (theme, fonts, tabs, snippets, recents)| WebView `localStorage`                                         |

The WebView only loads embedded assets over a private `app://` protocol, navigation and new windows are restricted, and every IPC call is authorized with a per-launch token.

## Project layout

```
src/main.rs            Window, WebView host, IPC, file I/O, exports, history
src/llm.rs             LLM providers: config, requests, responses, sanitization
assets/index.html      UI markup and styles (embedded in the exe)
assets/app.js          Editor, preview, tabs, palette, AI panel (embedded in the exe)
assets/mermaid.min.js  Mermaid.js v11.17.2 runtime (embedded in the exe)
build.rs               Windows resources and icon
installer/             NSIS script and release build script
```


## License

InfinityFlow is released under the [MIT License](LICENSE). Bundled Mermaid.js is MIT licensed (© Knut Sveidqvist and contributors). See [THIRD_PARTY_NOTICES.txt](THIRD_PARTY_NOTICES.txt) and the `LICENSES/` directory for details.

© 2026 lightgo · lightgo1230@gmail.com

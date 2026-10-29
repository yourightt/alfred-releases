# alfred

**Alfred — your AI butler for the PC**  
v0.1.0 · Electron 44 + React 19 + TypeScript · built with electron-vite

Alfred is a desktop AI assistant that runs locally on your machine. It chats with you using Anthropic (Claude), OpenAI, or a local Ollama model, remembers conversations and facts about you, and keeps itself up to date.

---

## What it does

**Chat**
- New conversations, plus a sidebar history list (open / rename / delete)
- Per-conversation provider and model choice: Anthropic, OpenAI, or local Ollama
- Streaming assistant responses with an auto-growing composer, Enter/Send (Shift+Enter for newline), a Stop button, markdown (GFM) rendering, and copy-to-clipboard on messages

**Setup & settings (first-run onboarding → Settings)**
- Choose your "brain": paste an Anthropic or OpenAI API key, or point at a local Ollama instance. Skippable — you can finish setup later in Settings.
- Save / replace / remove API keys per provider. Keys are **encrypted at rest** (Windows DPAPI / macOS Keychain / libsecret) and the renderer only ever sees a `…<last4>` hint — keys never cross IPC in plaintext.
- Ollama status check / refresh with an install link
- Preferences: send-on-enter toggle, default provider/model, history message limit, personality on/off + system prompt
- Memory management: add / remove facts Alfred remembers about you
- About page showing the app version

**Reliability & UX**
- Boot-error card ("Unable to start Alfred" + "Try again") instead of an endless spinner
- React error boundary with "Try again" reset around onboarding and the shell
- Per-message error banners; raw errors are logged to console only, user-safe text shown in the UI

**Updates**
- Auto-updater pointed at GitHub releases (`yourightt/alfred-releases`). Checks / downloads / installs on user action (not auto-download or auto-install).

---

## Security posture

The renderer window is sandboxed (`sandbox: true`, `contextIsolation: true`, `nodeIntegration: false`). It has **no direct filesystem, network, or Electron access** — everything goes through a typed preload bridge (`window.alfred`) that forwards calls to the main process over validated IPC channels. External links open in your default browser, and the app uses a single-instance lock.

---

## Tech stack

- **Runtime:** Electron 44 (main + preload)
- **UI:** React 19, ReactDOM, Tailwind CSS 4
- **State:** Zustand 5 (`useChatStore`, `useSettingsStore`)
- **Validation:** Zod 4 (shared IPC contract)
- **Markdown:** react-markdown 10 + remark-gfm 4
- **Build:** electron-vite 6, electron-builder 26, electron-updater 6
- **Tests:** vitest 5

See `package.json` for the full dependency list.

---

## Project structure

```
alfred/
├── src/
│   ├── main/            # Electron main process (index.ts, ipc/*.ipc.ts, providers/, services/, windows/)
│   ├── preload/         # preload bridge (index.ts + index.d.ts) → window.alfred
│   ├── renderer/        # React renderer (App.tsx, components/, stores/, lib/, assets/)
│   └── shared/          # Shared types (ipc-contract.ts, settings.ts, providers.ts, chat.ts)
├── build/               # Electron builder resources (icons, entitlements)
├── dist/                # Build output (e.g. alfred-0.1.0-setup.exe)
├── out/                 # Compiled output (package.json `main` → ./out/main/index.js)
├── electron-builder.yml # App ID com.alfred.app, publishes to yourightt/alfred-releases
├── electron.vite.config.ts
├── package.json
├── tsconfig.json / tsconfig.node.json / tsconfig.web.json
├── vitest.config.ts
├── eslint.config.mjs
├── .prettierrc.yaml
└── HANDOFF.md           # Dev handoff notes (recent changes + remaining work)
```

---

## Install & run

```bash
# Install dependencies (also installs Electron via postinstall)
npm install

# Start in development mode (HMR)
npm run dev

# Preview the built app
npm run start
```

### Build installers

```bash
# Full build (typecheck + electron-vite)
npm run build

# Platform-specific packages
npm run build:win   # Windows NSIS installer → dist/alfred-0.1.0-setup.exe
npm run build:mac   # macOS
npm run build:linux # Linux (AppImage / snap / deb)
```

A prebuilt Windows installer already exists at `dist/alfred-0.1.0-setup.exe`.

### Other scripts

| Script | Purpose |
|--------|---------|
| `npm run typecheck` | TypeScript check (node + web tsconfigs) |
| `npm test` | Run vitest suite |
| `npm run lint` | ESLint |
| `npm run format` | Prettier |

---

## License

See `LICENSE`.

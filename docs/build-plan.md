# Iris MCP — Build Plan

## North Star

> A dev or agent installs this MCP, drops a `.iris-mcp/` folder in their RN project, and immediately has eyes + hands on a running simulator — no extra servers, no heavy deps.

---

## Constraints

| Constraint | Decision |
|---|---|
| Runtime | Bun |
| Transport | stdio only |
| Dependencies | `adb` + `xcrun` (already on any RN machine) |
| Device targets | Simulators only (no physical devices) |
| Concurrency | Single device per platform, sequential queue |
| CI | Out of scope (emulator must already be running) |

---

## Project Structure

```
iris-cli/                           # standalone repo
├── src/
│   ├── index.ts                   # MCP server entry, tool registration
│   ├── config/
│   │   ├── loader.ts              # reads .iris-mcp/config.json
│   │   └── schema.ts              # config type definitions + validation
│   ├── drivers/
│   │   ├── interface.ts           # DeviceDriver interface
│   │   ├── android.ts             # adb implementation
│   │   └── ios.ts                 # xcrun simctl implementation
│   ├── tools/
│   │   ├── shell.ts               # shell_exec
│   │   ├── server.ts              # server_start, server_stop
│   │   ├── scripts.ts             # run_script
│   │   ├── workflows.ts           # run_workflow (all three tiers)
│   │   └── device.ts              # screenshot, ui_tree, tap, type, swipe, key
│   ├── queue/
│   │   └── index.ts               # sequential async queue
│   └── workflows/
│       ├── resolver.ts            # sh > yml/json > md priority logic
│       ├── executor.ts            # deterministic YAML/JSON runner
│       └── interpreter.ts         # markdown → LLM → steps
├── cli/
│   └── init.ts                    # `iris-cli init` scaffolds .iris-mcp/
├── package.json
└── README.md
```

---

## `.iris-mcp/` Config Schema

```json
{
  "version": 1,
  "platform": "android",           // "android" | "ios" | "both"
  "android": {
    "appId": "com.kite"            // used for targeted adb commands
  },
  "ios": {
    "bundleId": "co.codeleap.kite",
    "simulator": "auto"            // "auto" = first booted, or explicit UDID
  },
  "model": {
    "provider": "anthropic",
    "model": "claude-sonnet-4-20250514",
    "apiKeyEnv": "ANTHROPIC_API_KEY"
  },
  "env": {
    "LOGIN_EMAIL": "test@codeleap.co.uk",
    "LOGIN_PASSWORD": "123456"
  },
  "workflowPriority": ["sh", "yml", "md"]   // optional override
}
```

---

## Tool Surface

### Utility
| Tool | Description |
|---|---|
| `shell_exec` | Run any shell command in project root. Escape hatch for anything not covered. |

### Server lifecycle
| Tool | Description |
|---|---|
| `server_start` | Runs `scripts/start.sh`, manages child process |
| `server_stop` | Kills the managed dev server process |

### Scripts & workflows
| Tool | Description |
|---|---|
| `run_script` | Executes a named script from `scripts/` |
| `run_workflow` | Resolves and executes a named workflow (sh → yml/json → md) |

### Device — read
| Tool | Android | iOS |
|---|---|---|
| `device_screenshot` | `adb exec-out screencap -p` | `xcrun simctl io booted screenshot` |
| `device_ui_tree` | `adb shell uiautomator dump` | `xcrun simctl io booted enumerate` |

### Device — write
| Tool | Android | iOS |
|---|---|---|
| `device_tap` | `adb shell input tap x y` | `xcrun simctl io booted tap x y` |
| `device_find_element` | uiautomator dump → parse XML | simctl enumerate → parse |
| `device_type` | `adb shell input text` | `xcrun simctl io booted keyboard` |
| `device_swipe` | `adb shell input swipe` | `xcrun simctl io booted swipe` |
| `device_key` | `adb shell input keyevent` | `xcrun simctl io booted button` |
| `device_back` | `adb shell input keyevent 4` | n/a (Android only) |

---

## Phase 0 — Scaffold
**Goal:** working MCP server that connects cleanly, does nothing useful yet.

- [ ] Bun project init, MCP SDK dependency
- [ ] `src/index.ts` — server entry, stdio transport, tool registration stub
- [ ] `src/config/loader.ts` — reads `.iris-mcp/config.json`, validates schema
- [ ] `src/tools/shell.ts` — `shell_exec` wired end to end
- [ ] Manual test: connect via Claude Desktop, call `shell_exec "echo hello"`

**Exit criteria:** MCP connects, `shell_exec` works, config loads without errors.

---

## Phase 1 — Device Layer
**Goal:** agent can see the screen and interact with it.

- [ ] `src/drivers/interface.ts` — `DeviceDriver` interface (screenshot, uiTree, tap, type, swipe, key)
- [ ] `src/queue/index.ts` — simple sequential async queue, all device commands go through it
- [ ] `src/drivers/android.ts` — implement interface via `adb shell`
- [ ] `src/drivers/ios.ts` — implement interface via `xcrun simctl`
- [ ] `src/tools/device.ts` — all device tools wired to driver + queue
- [ ] Driver selected at startup based on `config.platform`

**Exit criteria:** call `device_screenshot` and get a PNG back; call `device_tap` and see a tap on the simulator.

---

## Phase 2 — Project Scripts
**Goal:** agent can boot the app and run setup scripts.

- [ ] `src/tools/server.ts` — `server_start` spawns `scripts/start.sh` as child process, `server_stop` kills it
- [ ] `src/tools/scripts.ts` — `run_script` resolves and shells out to `scripts/<name>.sh`
- [ ] Env vars from `config.json` injected into script environment
- [ ] Child process stdout/stderr streamed back as tool response

**Exit criteria:** `server_start` boots Metro, `run_script seed` runs the seed script, output visible in tool response.

---

## Phase 3 — Workflows
**Goal:** agent can call `run_workflow login` and the MCP handles the rest.

- [ ] `src/workflows/resolver.ts` — finds workflow file by name, applies priority (sh > yml/json > md)
- [ ] `src/workflows/executor.ts` — deterministic step runner for YAML/JSON workflows
- [ ] `src/workflows/interpreter.ts` — passes markdown content + screenshot + UI tree to configured LLM, executes returned steps
- [ ] `src/tools/workflows.ts` — `run_workflow` tool wired to resolver

**Exit criteria:** `run_workflow login` with a `.md` file navigates to the home screen autonomously; same with a `.yml` file runs deterministically.

---

## Phase 4 — Polish + Publish
**Goal:** something a stranger can install and use in 5 minutes.

- [ ] `cli/init.ts` — `iris-cli init` scaffolds `.iris-mcp/` with config template and example scripts
- [ ] Config validation with human-readable errors (not just type errors)
- [ ] README: prerequisites, installation, configuration, tool reference
- [ ] npm publish as `iris-cli`
- [ ] Example `.iris-mcp/` for a generic Expo project in the repo

**Exit criteria:** `npx iris-cli init` works in a fresh Expo project, README covers the full setup.

---

## Explicitly Out of Scope (v1)

- Physical device support
- Multi-device targeting
- CI bootstrap (AVD/simulator lifecycle management)
- Orchestrator integration
- Build triggering (Expo MCP's job)
- Video capture / session recording
- Network interception

---

## Dependencies

```json
{
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0"
  },
  "devDependencies": {
    "@types/bun": "latest",
    "typescript": "^5.0.0"
  }
}
```

System requirements (must be present on the machine, not installed by this package):
- `adb` — Android SDK Platform Tools
- `xcrun` — Xcode Command Line Tools (macOS only, for iOS)
- Bun >= 1.0

---

## Notes for Claude Code

- Start with Phase 0 — don't skip to the interesting parts
- The `DeviceDriver` interface is the critical abstraction — get it right before implementing Android or iOS
- The queue must wrap **all** device tool calls, not just the ones that seem concurrent
- Markdown workflow interpretation should be a thin layer — pass content + context to the LLM, get back a list of tool calls, execute them using the same executor as YAML workflows
- Config loading happens once at server startup, not per-request

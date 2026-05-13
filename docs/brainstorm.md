# Iris MCP — Brainstorm

## Core Idea

A standalone MCP server that gives any dev or agent **eyes and hands on a running RN simulator** — without a human in the loop. Install once, configure per-project, callable from Claude, Cursor, CI, or any MCP-compatible agent.

The primary use case is **ambient context gathering**: an agent working on a feature can see what the app looks like right now, verify a UI change rendered correctly, or run a login workflow to reach a specific screen — all without asking a human to do it.

---

## Problem It Solves

Today, when an agent is helping with a React Native feature, it has no way to:
- Verify the UI actually rendered correctly
- Navigate to a specific screen to get context
- Run a login flow autonomously before doing something else
- Know what's on screen without a human describing it

The existing solutions (Expo MCP, Appium, Detox) are either cloud-dependent, too heavy, or not designed for agent-to-agent communication.

---

## What Makes This Different

- **Light**: only `adb` and `xcrun` as dependencies — both already on any RN dev machine
- **Project-local config**: a `.iris-mcp/` folder lives in the project, versioned with the code
- **Three tiers of workflow expressiveness**: shell scripts, structured YAML/JSON, or plain markdown
- **Agent-first**: designed to be called by other agents, not just humans
- **Standalone**: no orchestrator required, no cloud dependency, no paid plan

---

## The `.iris-mcp/` Knowledge Layer

Inspired by Relic's `.relic/` spec layer. Each project defines its own device interaction knowledge:

```
.iris-mcp/
├── config.json          # platform targets, model config, env vars
├── scripts/
│   ├── start.sh         # how to boot the app
│   ├── seed.sh          # seed test data
│   └── reset.sh         # reset app state
└── workflows/
    ├── login.md          # "use email x and password y to login"
    ├── onboarding.yml    # explicit step-by-step flow
    └── deep-link.sh      # needs raw adb commands
```

This folder is the "how does this specific app work" layer. Any agent that knows the MCP protocol can use it without knowing anything about the app internals.

---

## Workflow Tiers

### Tier 1 — Markdown (natural language)
```markdown
# login
Use email `test@codeleap.co.uk` and password `123456` to login.
Navigate through any onboarding screens until you reach the home screen.
```
Agent receives this + current screenshot + UI tree and figures out the steps.
Best for: simple, obvious flows. Easy for devs to write.

### Tier 2 — YAML/JSON (structured, deterministic)
```yaml
name: login
steps:
  - device_tap:
      text: "Email"
  - device_type:
      text: "test@codeleap.co.uk"
  - device_tap:
      text: "Password"
  - device_type:
      text: "123456"
  - device_tap:
      text: "Sign In"
```
Executes deterministically regardless of which model runs it.
Best for: reliable, repeatable flows that can't afford LLM interpretation variability.

### Tier 3 — Shell scripts (raw, maximum control)
```bash
#!/bin/bash
# Handles OTP edge case not expressible in tool calls
adb shell am start -n com.kite/.MainActivity
adb shell input text "$LOGIN_EMAIL"
```
Best for: flows needing env vars, conditionals, or things outside the tool surface.

### Resolution priority
When `run_workflow login` is called, the MCP looks for:
1. `login.sh` → shells out directly
2. `login.yml` or `login.json` → deterministic executor
3. `login.md` → LLM-interpreted execution

Priority can be overridden in `config.json`.

---

## Device Layer Philosophy

Agents don't need video streaming (that's for humans using scrcpy). Agents need:

1. **Screenshot** — one frame, base64 PNG
2. **UI tree** — what's on screen, what's tappable, what has what text
3. **Input** — tap, swipe, type, key events

Android: `adb shell` + `uiautomator dump` for UI tree
iOS: `xcrun simctl` for everything

Both are already installed on any RN development machine. No extra servers, no Java, no Appium.

---

## Sequential Queue

Device interaction is inherently sequential — you can't tap and screenshot simultaneously. The MCP maintains an internal async queue so tool calls from concurrent agents are serialized automatically. Callers don't need to think about this.

This queue is intentionally simple for now. In a future version, when the orchestrator project matures, this queue could be replaced by the orchestrator's primitive.

---

## On-Demand Server Lifecycle

Metro (the RN dev server) is **not auto-started**. It only starts when:
- A dev explicitly calls `server_start`
- An agent calls `server_start` as part of a workflow

The MCP process itself runs continuously (managed by the agent host via stdio), but the app's dev server is a child process the MCP manages separately. This keeps the MCP cheap to run when device interaction isn't needed.

---

## Relationship to Expo MCP

The official Expo MCP handles: docs search, EAS builds, SDK version management.
This MCP handles: what's on screen right now, how to interact with it, app-specific workflows.

They're complementary. A sophisticated agent setup could use both simultaneously.

---

## Future Scope (explicitly not v1)

- Physical device support (Android via ADB already works, iOS needs developer mode + Mac)
- Multi-device targeting (run same workflow on two devices)
- CI bootstrap (spin up AVD/simulator from scratch)
- Orchestrator integration (replace internal queue + model config with orchestrator primitives)
- Video capture / session recording
- Network interception (proxy device traffic for API mocking)
- Build triggering (out of scope, Expo MCP's job)

---

## Open Design Questions

- **Name**: `iris-cli` (npm package name), `.iris-mcp/` (project folder)
- **Config schema versioning**: how do we handle `.iris-mcp/config.json` evolving without breaking existing projects?
- **Markdown workflow LLM**: should the model config be per-workflow or global in `config.json`?
- **Error surface**: when a `device_tap` can't find the element, what does the agent receive? Structured error with UI tree snapshot, or just a message?
- **iOS simulator UDID**: auto-detect booted simulator, or require explicit UDID in config?

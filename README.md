# opencode-opensky

**Give OpenCode hands on macOS.** This plugin wires [OpenSky](https://github.com/pkyanam/OpenSky)
into [OpenCode 2](https://opencode.ai) as native custom tools: any OpenCode agent can list apps,
read accessibility trees, click, type, drag, scroll, paste — 1:1 with the ChatGPT/Codex
Computer-Use surface, on-device, clean-room.

## Requirements
- macOS (Apple Silicon), OpenCode 2.x
- The `opensky` binary on PATH (or `OPENSKY_BIN` env var):
  ```sh
  git clone https://github.com/pkyanam/OpenSky.git
  cd OpenSky && swift build -c release
  mkdir -p ~/.local/bin && cp .build/release/opensky ~/.local/bin/
  ```
- First tool use triggers the macOS **Accessibility** permission prompt (and **Screen Recording**
  for screenshots). Approve once for the terminal that spawned OpenCode.

## Install
Drop the plugin file into OpenCode's global plugin directory:
```sh
curl -fsSL https://raw.githubusercontent.com/pkyanam/opencode-opensky/main/index.ts   -o ~/.config/opencode/plugins/opensky.ts
```
Restart OpenCode. Verify with any model:
> "List your tools whose names contain opensky" → 13 tools.

## Tools (13)
| Tool | What it does |
|---|---|
| `opensky_list_apps` | running + launchable apps (id/bundle/pid/frontmost) |
| `opensky_get_app_state` | AX tree with `[N]` element indices + screenshot path |
| `opensky_click` | click by element index (preferred) or x/y |
| `opensky_type` | type text into the focused element |
| `opensky_press_key` | key chords (`Control_L+a`, `Return`, …) |
| `opensky_scroll` | scroll up/down/left/right by pages |
| `opensky_drag` | drag from→to in window coordinates |
| `opensky_paste` | clipboard paste (restores user clipboard) |
| `opensky_set_value` | set an element's value directly (form fields) |
| `opensky_select_text` | select text by needle/prefix/suffix |
| `opensky_action` | any named AX action (AXPress, AXRaise…) |
| `opensky_policy` | the app's approval decision |
| `opensky_skill` | full OpenSky agent manual, printed inline |

## The one rule agents must know
Element indices are **snapshot-scoped**. After any UI mutation, re-run
`opensky_get_app_state` before the next indexed action. Every tool description
repeats this; `opensky_skill` prints the full manual.

## Safety
- Policy engine: system-critical apps (Finder, loginwindow, Settings, keychain…) are
  **forbidden**; high-risk apps (Mail, Messages…) require approval for writes.
- `forbidden` is permanent; agents are taught to surface it, never retry.
- Nothing leaves the machine.

## Verified end-to-end (2026-09-26, opencode 2.0.16)
- 13 tools registered and listed by a live model session
- `opensky_list_apps` → 26 real apps
- `opensky_get_app_state` on Helium → real AX tree (model reported a real element title)
- `opensky_press_key` Escape → delivered to Helium (input write path)
- Policy loop: Finder correctly returned `forbidden`, agent surfaced it instead of retrying

## Note on providers
Verified with `cloudflare-workers-ai/@cf/zai-org/glm-5.3-flash` (user's configured provider).
The `belweave` router currently errors ("upstream model error") on any tool-result round-trip —
router-side, unrelated to this plugin; text-only sessions work on the same router.

MIT — not affiliated with or endorsed by OpenAI or OpenCode.

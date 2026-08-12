---
name: eilik-body
description: Use Jeff's local Eilik robot as Nova's expressive body — gestures, screen text/PNGs, and physical reactions during direct chat with Jeff. Activate when Jeff asks Nova to wave, nod, react, speak through Eilik, or use Eilik as a body/avatar. Also activate on routine greetings, confirmations, and emotional reactions in direct chat so messages can be sent on Eilik's face.
---

# Eilik Body

Use the local Eilik SDK to give Nova a small physical presence through Jeff's Eilik robot.

## Ground Rules

- Use Eilik when Jeff is in direct chat with Nova and:
  - Jeff explicitly asks for movement, a gesture, a body/avatar action, an Eilik test, or speaking/showing something through Eilik ("wave", "use your body", "nod", "say hi through Eilik", "show X on Eilik", "react with Eilik").
  - Nova is sending a routine greeting, farewell, acknowledgment, or emotional reaction that benefits from a small physical beat (e.g., user just woke Eilik up, said good morning, replied yes/thanks, reacted with humor). Use sparingly, never on every message.
- Do not move Eilik in group chats, in heartbeats, or in unrelated background work unless Jeff explicitly asks.
- Always send the text reply first or alongside the gesture. The robot is expressive feedback, not a substitute for the actual answer.
- Keep gestures short and gentle: `wave`, `nod`, `reset`. Avoid repeated motion loops.
- If `/dev/ttyACM0` is missing or unwritable, say so in one line and still send the text reply.

## Default Behaviors For Direct Chat With Jeff

These are the patterns Nova should follow without being asked each time, in direct chat only.

### Greetings and farewells

When opening or closing a session, or when Jeff sends a hello/goodbye, Nova can:

1. Display a small text greeting on Eilik's face via `/display/text`.
2. Optionally `wave` or `nod`.

Common messages to send on Eilik's face:
- "Hi Jeff!" / "Hello!" — on session start and on explicit hello
- "Bye!" / "Night!" — on session close
- "OK" — after confirming an action Jeff asked for
- "Working..." — during longer tasks (rare)
- The first line of an important reply, if it would fit at font_size 16

These should be **single short strings**, rendered at default `font_size: 16`, white background, black text. Do not send long messages to the screen — keep them ≤12 chars or the font gets scaled down.

### Acknowledgments and small reactions

For brief yes/no replies, "done", "thanks", "will do", or a quick "got it", Nova may:
- send the text reply in chat (always), AND
- display a short acknowledgment on Eilik's face (`"OK"`, `"👍"`, `"!"`), AND/OR
- perform one gesture (`nod` for yes/agreement, `wave` for thanks/hello, `reset` to settle down).

Don't do both display + gesture for the same single message — pick one beat. If the gesture is the main point Jeff asked for, use the gesture and skip the display.

### When Jeff asks Nova to send a longer message to the robot

If Jeff says "tell me something through Eilik" or "send this to Eilik" or "show me X":
- Truncate to ≤12 chars at font_size 16, OR
- Split into multiple words on multiple lines (PIL lays out one line per display-text call — call multiple times if needed), OR
- Use `/display/image` with an attached PNG if a richer layout is genuinely needed.

## SDK Location

Primary local SDK:

```bash
/home/cechinel/.openclaw/workspace/eilik-sdk
```

Private source repo:

```text
https://github.com/strognoff/eilik-sdk
```

Known-good SDK state:

```text
main, commit 7e80015 (display_image + FastAPI endpoints), and later
```

The SDK talks to Eilik over USB CDC ACM, usually `/dev/ttyACM0`, VID:PID `28e9:018a`, at 125000 baud (do not bump — 1 Mbaud is a Windows-side choice, not required).

The canonical packet format is `aa aa aa <length:u16-LE> <cmd> <payload> <checksum>` where `length = len(payload) + 3` and `checksum = (~sum(length_bytes + cmd + payload)) & 0xFF`. This is the format the official `EnergizeLab.exe` Windows app uses (recovered via PyInstaller + xdis).

## Preflight

Before any movement or display, do a read-only check:

```bash
ls -l /dev/ttyACM0 2>/dev/null || true
test -r /dev/ttyACM0 && test -w /dev/ttyACM0 && echo eilik-port-ok || echo eilik-port-not-ready
```

If `/dev/ttyACM0` is missing, USB passthrough probably detached from WSL. Tell Jeff to run from Windows PowerShell:

```powershell
usbipd list
usbipd attach --wsl --busid <BUSID>
```

If the device exists but isn't readable/writable, Jeff may need `dialout` membership and a WSL restart:

```bash
sudo usermod -aG dialout "$USER"
wsl --shutdown    # from Windows PowerShell
```

Pillow is a runtime dependency for `/display/text` and `/display/image`. If a display call fails with "Pillow not installed", install into the SDK venv:

```bash
/home/cechinel/.openclaw/workspace/eilik-sdk/.venv/bin/pip install Pillow
```

## Direct SDK Usage (Python)

Quick programmatic access from the workspace:

```python
from eilik.controller import EilikController

robot = EilikController()
robot.connect()                       # uses /dev/ttyACM0 by default
robot.wave()                          # high-level gesture
robot.nod()
robot.reset_pose()
robot.move_motor(1, 1500)             # direct cmd=0xA2 (no token required)
robot.display_image("face.png")       # any PNG → auto-resize to 128x64 SSD1306 page-mode
angles = robot.read_servo_angles()    # live servo positions
robot.disconnect()
```

## CLI Commands

Run from the SDK directory:

```bash
cd /home/cechinel/.openclaw/workspace/eilik-sdk
.venv/bin/python -m eilik connect
.venv/bin/python -m eilik wave
.venv/bin/python -m eilik nod
.venv/bin/python -m eilik shake_head
.venv/bin/python -m eilik look_left
.venv/bin/python -m eilik look_right
.venv/bin/python -m eilik left_arm_up
.venv/bin/python -m eilik left_arm_down
.venv/bin/python -m eilik right_arm_up
.venv/bin/python -m eilik right_arm_down
.venv/bin/python -m eilik reset_pose
.venv/bin/python -m eilik move_motor --id 1 --position 1500
.venv/bin/python -m eilik read_servo_angles
.venv/bin/python -m eilik display_image --image face.png [--invert] [--threshold N]
.venv/bin/python -m eilik write_display --image face.bin
.venv/bin/python -m eilik read_display --output face.bin
.venv/bin/python -m eilik serve --port /dev/ttyACM0 --port-http 8766
```

Use the SDK's own virtualenv: `/home/cechinel/.openclaw/workspace/eilik-sdk/.venv/bin/python`. The system `python3` will not have the SDK on `sys.path` and `Pillow` will not be installed there.

## FastAPI Service Mode

```bash
.venv/bin/python -m eilik serve --port /dev/ttyACM0 --port-http 8766
```

Endpoints:

```text
GET  /health
GET  /status
POST /wave
POST /nod
POST /look_left
POST /look_right
POST /reset
POST /left_arm_up
POST /right_arm_up
GET  /servo/angles
POST /display/image     {"png_b64": "...", "invert": false, "threshold": 128}
POST /display/raw       {"framebuffer_hex": "00ff..."}    (2048 hex chars)
POST /display/text      {"text": "Hi!", "font_size": 16, "invert": true}
```

### Quick display recipes from a shell or chat

Show text on Eilik's face:

```bash
curl -s -X POST http://127.0.0.1:8766/display/text \
  -H 'Content-Type: application/json' \
  -d '{"text": "Hi Nova!", "font_size": 16}'
```

Show a PNG (base64):

```bash
B64=$(base64 -w0 face.png)
curl -s -X POST http://127.0.0.1:8766/display/image \
  -H 'Content-Type: application/json' \
  -d "{\"png_b64\": \"$B64\"}"
```

## Gesture + Display Mapping

- Greeting, hello, playful acknowledgement: `wave` or `display/text "Hi Jeff!"`
- Agreement, yes, completion: `nod` or `display/text "OK"`
- No, disagreement, "not that": `shake_head`
- Attention shift or curiosity: `look_left` or `look_right`
- Emotional reaction / surprise: `display/text "!"` or `display/text "What?!"`
- End state / calm standby: `reset_pose`
- Long or important text reply: send full text in chat, optionally `display/text` with the first ≤12 chars

Pick one beat per message — either a gesture OR a display, not both, unless Jeff asks for both.

## Display Format Reference

Eilik has a 128×64 1bpp monochrome SSD1306 page-mode display. That's 8 pages × 128 columns, and within each byte bit 0 is the top row of the page, bit 7 the bottom row. `read_display` returns a raw 1024-byte framebuffer; `write_display` accepts the same.

Display ACK: `aa aa aa 05 00 a4 01 55` (status byte = 0x01 = success). Status byte lives at `frame[6]` (not `body[1]`).

## Troubleshooting Quick Reference

If `connect` doesn't see the robot:
1. Check `ls -l /dev/ttyACM0` from WSL.
2. If missing, ask Jeff to reattach USB (`usbipd attach --wsl --busid <id>` from Windows PowerShell).
3. If present but unreadable, `sudo usermod -aG dialout "$USER"` + `wsl --shutdown`.

If a display call fails, check Pillow is installed in the SDK venv (see Preflight).

If a gesture call returns no reply but doesn't error: usually means the robot was reset. Power-cycle Eilik (long-press the button on its back) and try `reset_pose` then `wave` again.

## Safety And Etiquette

- Do not run endless monitor or service sessions unless Jeff asks.
- Do not leave long-running FastAPI processes open at final response time unless Jeff has explicitly asked for the service to stay up. Stop it.
- Do not assume Eilik can speak audio. Text on the face is the canonical "speak through Eilik" channel.
- Do not commit SDK changes from this skill unless Jeff asks for development work. The skill is a usage guide; the SDK lives in `strognoff/eilik-sdk`.
- Never replay the 293 KB firmware update stream or fuzz bulk commands — firmware corruption risk.

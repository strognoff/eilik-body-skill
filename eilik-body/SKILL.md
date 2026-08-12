---
name: eilik-body
description: Use Jeff's local Eilik robot as Nova's expressive body through the Eilik SDK. Use when Jeff asks Nova to move, wave, nod, react physically, test the robot, speak/communicate through Eilik, or use Eilik as a body/avatar in direct conversation.
---

# Eilik Body

Use the local Eilik SDK to give Nova a small physical presence through Jeff's Eilik robot.

## Ground Rules

- Use Eilik only when Jeff clearly asks for robot movement, physical presence, a gesture, a body/avatar action, or an Eilik test.
- In direct Telegram chat with Jeff, a request like "wave", "nod", "use your body", "react with Eilik", or "say hi through Eilik" authorizes the requested local movement.
- Do not move Eilik in response to unrelated chat, background heartbeats, or group-chat context unless Jeff explicitly asks.
- Keep important information in text too. Robot movement is expressive feedback, not a replacement for a clear answer.
- Prefer short, gentle gestures: `wave`, `nod`, `look_left`, `look_right`, `reset`. Avoid repeated motion loops unless Jeff asks.
- If the robot is unavailable, say so briefly and include the exact failing check or command.

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
commit 84dde26
```

The SDK talks to Eilik over USB CDC ACM, usually `/dev/ttyACM0`, VID:PID `28e9:018a`.

## Preflight

Before movement, do a quick read-only check when the connection state is unknown:

```bash
cd /home/cechinel/.openclaw/workspace/eilik-sdk
ls -l /dev/ttyACM0 2>/dev/null || true
test -r /dev/ttyACM0 && test -w /dev/ttyACM0 && echo eilik-port-ok || echo eilik-port-not-ready
```

If `/dev/ttyACM0` is missing after WSL restarted, Windows USB passthrough probably detached. Tell Jeff to run from Windows PowerShell:

```powershell
usbipd list
usbipd attach --wsl --busid <BUSID>
```

If `/dev/ttyACM0` exists but is not readable/writable, Jeff may need `dialout` membership and a WSL restart:

```bash
sudo usermod -aG dialout "$USER"
```

Then from Windows PowerShell:

```powershell
wsl --shutdown
```

## Commands

Run commands from the SDK directory:

```bash
cd /home/cechinel/.openclaw/workspace/eilik-sdk
python3 cli.py connect
python3 cli.py wave
python3 cli.py nod
python3 cli.py shake_head
python3 cli.py look_left
python3 cli.py look_right
python3 cli.py reset
python3 cli.py monitor
```

Useful service mode:

```bash
python3 cli.py serve --host 127.0.0.1 --port-http 8765
```

Service endpoints:

```text
GET  /status
POST /wave
POST /nod
POST /look_left
POST /look_right
POST /reset
```

## Gesture Mapping

Use simple mappings unless Jeff specifies otherwise:

- Greeting, hello, playful acknowledgement: `wave`
- Agreement, yes, completion: `nod`
- No, disagreement, "not that": `shake_head`
- Attention shift or curiosity: `look_left` or `look_right`
- End state / calm standby: `reset`

For a text reply plus body action, move first only when the gesture is the main requested action. Otherwise send the text reply and then perform one concise gesture.

Examples:

- Jeff: "Wave at me" -> run `python3 cli.py wave`, then reply that Eilik waved.
- Jeff: "Use Eilik to say yes" -> run `python3 cli.py nod`, then reply briefly.
- Jeff: "Explain what happened" -> answer in text; do not move unless Jeff also asks for Eilik.

## Screen And Display Work

Arbitrary screen text is not supported yet. If Jeff asks about Eilik's display, read `references/display-capture-notes.md` before answering or changing the SDK.

Current finding: the firmware-update capture includes large `cmd=03` binary/resource frames while Eilik shows update/progress state, but no simple text-display command has been identified.

## Troubleshooting

If `python3 cli.py connect` opens the port but reports no handshake reply, use the current SDK fallback behavior first. The known capture-derived status probe is already integrated in the SDK. If it still fails, ask Jeff to wake/power-cycle Eilik, ensure the official app is not holding the device, and reattach USB passthrough if WSL restarted.

Packet logs are saved in:

```bash
/home/cechinel/.openclaw/workspace/eilik-sdk/logs/eilik.log
```

Monitor logs are saved in:

```bash
/home/cechinel/.openclaw/workspace/eilik-sdk/logs/eilik-monitor.log
```

## Safety And Etiquette

- Do not run endless monitor or service sessions unless Jeff asks.
- Do not leave long-running processes open at final response time; stop or report them clearly.
- Do not assume Eilik can speak audio. Treat "communicate through Eilik" as gesture plus normal chat text unless a separate audio/TTS path is explicitly available.
- Do not commit SDK changes from this skill unless Jeff asks for development work. For normal body use, only run existing commands.


---
name: eilik-body
description: Control Jeff's Eilik via the API-first on-demand bridge, webapp, routines, and safe physical beats.
---

# eilik-body

Personal-use skill for Nova to control Jeff's Eilik robot through the local
SDK/API bridge.

## Core Rule

Use the HTTP API as the canonical bridge for normal Eilik actions. The API is
served by the SDK at `http://127.0.0.1:8765` from
`/home/cechinel/.openclaw/workspace/eilik-sdk/`.

Do not open long-lived serial sessions for routine interactions. The fixed
service model is on-demand: connect -> run command or routine -> disconnect.
Between commands, `/health` and `/status` should report `mode=on-demand`,
`connected=false`, `protocol=null`.

## When To Use Eilik

Use Eilik only when it adds a useful physical beat and Jeff is likely near the
robot:

- Jeff explicitly asks Eilik to move, show text, play a sequence, or test the
  app.
- Direct-chat greetings, acknowledgments, completions, and farewells where a
  small beat feels natural.
- Brief success/failure signals for work Jeff is watching.
- Kids-game/play sessions using the webapp or sequence API.

Stay quiet physically for heartbeats, background checks, group chats, and
routine status replies unless Jeff explicitly asks.

## Safety Rules

- Trust Jeff's physical observation over logs, ACKs, display hashes, or
  readbacks.
- Do not claim Eilik visibly moved unless Jeff confirms it or the user asked
  only for API/log verification.
- Do not use persistent keepalives or long-held serial sessions.
- Do not replay official/resource/update frames as a recovery method.
- Do not run diagnostic display readbacks as final proof when debugging a stuck
  screen.
- Do not silently call `restore_idle_face()` after custom display tests. It
  shows the captured half-eye face and confused Jeff during testing.
- Do not use `running_number=0` as an idle recovery path; it can show the
  turquoise play/control icon.
- Do not auto-reset pose after gesture-only requests unless Jeff asks for
  cleanup.
- Keep live tests minimal and describe exactly what command flow will run
  before running it.

## API First

Prefer these endpoints:

```bash
curl http://127.0.0.1:8765/health
curl http://127.0.0.1:8765/motions
curl 'http://127.0.0.1:8765/logs/recent?lines=120'
```

Display text:

```bash
curl -X POST http://127.0.0.1:8765/display/text \
  -H 'Content-Type: application/json' \
  -d '{"text":"Hello Alice!!","hold_seconds":5,"auto_idle":false}'
```

Named motion:

```bash
curl -X POST http://127.0.0.1:8765/motion/wave
```

Direct servo movement:

```bash
curl -X POST http://127.0.0.1:8765/servo/move \
  -H 'Content-Type: application/json' \
  -d '{"motor":"right_arm","position":500}'
```

Text plus both arms routine:

```bash
curl -X POST http://127.0.0.1:8765/routine/display_text_arms \
  -H 'Content-Type: application/json' \
  -d '{"text":"Hello Alice!!","duration_seconds":5,"cleanup":"disconnect_only"}'
```

Kids sequence routine:

```bash
curl -X POST http://127.0.0.1:8765/routine/sequence \
  -H 'Content-Type: application/json' \
  -d '{
    "steps": [
      {"type":"display_text","text":"Hello Alice!!","hold_seconds":1.5},
      {"type":"motion","motion":"wave"},
      {"type":"wait","seconds":0.5},
      {"type":"motion","motion":"thumbs_up"}
    ],
    "cleanup":"disconnect_only",
    "step_pause_seconds":0.2
  }'
```

## Webapp

Use the webapp for interactive play and debugging:

- Control room: `http://127.0.0.1:8765/app`
- Kids game: `http://127.0.0.1:8765/app/game`
- API docs: `http://127.0.0.1:8765/docs`
- OpenAPI: `http://127.0.0.1:8765/openapi.json`

Loading the webapp should be read-only. Eilik should move or change screen only
after a button or Play is pressed.

## Building Blocks

The kids game and sequence API are the preferred model for multi-step behavior.
Use simple blocks:

- `display_text`: show text on Eilik's screen.
- `motion`: run one named motion from `/motions`.
- `wait`: pause between blocks.

For a child-friendly sequence, keep steps short, use `cleanup=disconnect_only`
unless Jeff asks for arms reset, and leave enough `step_pause_seconds` for
visible separation.

## Logs

Logs are part of the control surface. When a movement or screen action does not
match Jeff's visual report:

1. Ask what Jeff saw physically.
2. Fetch recent logs:

```bash
curl 'http://127.0.0.1:8765/logs/recent?lines=160'
```

3. Look for `API_START`, `API_END`, operation id, duration, errors,
   `TX_SERVO_DIRECT`, `TX_WRITE_DISPLAY`, and `SEQUENCE_*` lines.
4. Explain the exact command flow sent, not just the endpoint name.

Logs are bounded by the SDK with rotating files: default `logs/eilik.log`, max
1MB, 5 backups.

## Default Direct-Chat Beats

Use sparingly and only in direct chat with Jeff:

- Greeting: `POST /motion/wave` or a short `/display/text` with
  `auto_idle=false`.
- Acknowledgment: `POST /motion/nod` or `POST /display/text` with `OK` /
  `Done`.
- Completion: a short text + small motion sequence through `/routine/sequence`.
- Error/apology: text-only first; avoid expressive motion if the robot state is
  uncertain.

One beat per message. If Jeff asked for a specific motion only, do that motion
only. If Jeff asked for text only, do text only.

## Recovery

If Eilik gets stuck on a turquoise play/control icon or a resource-sync/debug
screen:

- Stop sending commands.
- Check whether the service is holding a serial session or keepalive loop.
- Do not replay official frames.
- Ask Jeff for visual confirmation and, if needed, physical restart/replug.
- After restart, verify only USB presence, legacy-token control, and minimal
  `reset_pose`; avoid display writes/readbacks until stable.

## References

- SDK repo: `/home/cechinel/.openclaw/workspace/eilik-sdk/`
- Latest pushed SDK state for this skill: `strognoff/eilik-sdk` commit
  `f219cdf` (`Add Eilik kids sequence webapp`).
- Webapp docs:
  `/home/cechinel/.openclaw/workspace/eilik-sdk/docs/WEBAPP.md`
- API docs: `/home/cechinel/.openclaw/workspace/eilik-sdk/docs/API.md`
- OpenAPI spec:
  `/home/cechinel/.openclaw/workspace/eilik-sdk/docs/openapi.json`

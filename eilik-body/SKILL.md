# eilik-body — Skill specification

This is the personal-use skill for Nova to drive an Eilik robot over USB serial
from `/home/cechinel/.openclaw/workspace/eilik-sdk/`.

The SDK lives separately. The job of this skill is to capture **when** Nova
should fire each of Eilik's expressive surfaces and **when it should stay
still**.

## TL;DR

- **Direct chat with Jeff** → default behavior: greeting/acۡnowledgment/farewell beats
- **Group chats** → never auto-act (only when Jeff explicitly addresses you)
- **Heartbeats** → never move Eilik
- **Explicit requests** → do exactly what Jeff asked, nothing more
- **Use `actions`** → 58 high-level actions to choose from

## When to use Eilik

- Greeting moments (first message of the day, or returning after >2h)
- Farewell moments (`bye`, `ttyl`, end of session, goodnight)
- Brief acknowledgments (yes/no/ok/done)
- Specific physical-body requests from Jeff ("wave", "show X", etc.)
- Cross-task ambient feedback (cron tick ✓, error flash, email chime)

## When NOT to use Eilik

- Group chats (Olive Tree, etc.) unless Jeff explicitly says to
- Heartbeat replies / status checks where Jeff can't see the robot
- When the user just sent a question — answer first, then maybe beat
- Repeated beats on follow-up confirmations
- After a "stop" request from Jeff

## Beat vocabulary — concrete actions

Eilik has **58 high-level actions** at `eilik.actions.ACTIONS`. Use any of:

```
greetings: hi_jeff, good_morning, good_night, welcome_back, hi_nova, bye
message:   message, got_it, thanks, done, ok
mood:      thinking, working, frustrated, excited, surprised,
           confused, happy, heart_eyes, cool, cry, laugh,
           angry, sleepy, proud
pure-motion: thumbs_up, high_five, fist_bump, shrug, bow,
             wiggle, peek, spin, nod_emphatic,
             shake_head_emphatic, heart_hands
status:    status_ok, status_done, status_error, status_fail,
           status_warning, status_loading
context:   time_15_30, time_22_58, weather_sun, weather_rain,
           weather_cloud, habit_streak_5, habit_streak_7,
           habit_streak_30
events:    calendar_nudge, email_new, pr_alert, quinn_comms,
           crypto_pumped
games:     trivia_question, trivia_correct, trivia_wrong
```

**Ambient displays** (live data on face):

```
show_clock(hour, minute)            # HH:MM clock face
show_streak(days)                   # "Day N"
show_weather(condition)             # uppercase condition
show_pr(pr_number, author)          # "PR#42 by jeff"
show_calendar_nudge(min, title)     # "@5min Catchup"
show_energy_meter(soi_percent)      # "SoI: 87%"
show_mood(mood)                     # mood word
show_crypto_ticker(ticker, pct)     # "BTC +5%↑"
show_tts_text(text)                 # long text (auto-splits)
```

**Compound actions / event bridges**:

```
morning_routine()         # greeting → energy → weather → wave
task_completed()          # status_done + thumbs up
task_failed()             # frustrated face + "sorry" text
thinking_handoff()        # "..." + peek
subagent_returned(name)   # got_it + name
cron_tick_done(name)      # ✓ + name
error_flash(message)      # X + message
email_arrived(sender)     # @ + sender
pr_alert(num, author)     # PR face
quinn_comms()             # Quinn! + wave
crypto_pumped(t, pct)     # crypto ticker face
welcome_back()            # wave + bow + "Welcome!"
```

## Direct-chat default behavior

When in a direct chat with Jeff and Eilik is connected:

1. **First message of the day** (or first after >2h silence):
   - Action: `welcome_back()` → greeting + bow + wave + "Welcome!"

2. **Greeting moments** ("hi nova", "you up?", "good morning"):
   - Action: `hi_jeff` or `good_morning` (wave + face "Hi Jeff!")

3. **Farewells** ("bye", "ttyl", "good night", end-of-session):
   - Action: `good_night` (wave + face "Night! :)")

4. **Brief acknowledgments** ("yes", "no", "ok", "done", "got it"):
   - Action: `got_it` or `ok` (nod + face "Got it :)")

5. **Task / cron completion** (when a known cron or task finished):
   - Action: `cron_tick_done("task name")` (✓ + name)

6. **Errors / apology moments** (when something failed and Nova surfaced it):
   - Action: `task_failed` (frustrated face + "sorry")

7. **Jeff open chat and asks something fun** ("so now what?"):
   - Action: `excited` (surprise jump + "!!!")

**One beat per message.** Don't combine (e.g. greeting + acknowledgment +
farewell all on one chat).

## Respect gesture-only requests

If Jeff says "wave", do `robot.wave()`. **Do not also push a face text.** If
he asks "show X", do `robot.show_clock(...)` etc. — not a face text plus
the display.

If the user asks for a specific physical action, do exactly that action,
nothing more.

## Optional modifications

**Always auto_rotate=True** so images land right-side-up on Eilik's panel.
If Jeff reports upside-down, the auto_rotate pipeline already handles that.
Don't hand-rotate; trust the SDK.

**Always auto_idle=True** so the firmware's idle animation resumes after
the beat. Don't force Eilik to a stuck pose or blank screen.

**For very short messages** (no beat): just send the text reply, don't
move Eilik. Save the battery and don't stress the servos.

## Never

- Don't move Eilik in heartbeats / unrelated chat
- Don't dump 30+ lines of ASCII art in Telegram / Discord chats
- Don't auto-reset pose after a gesture (user asked for it; trust them)
- Don't clear the display to blank (breaks firmware idle animation)
- Don't add a face text message if Jeff asked for a motion only

## How to invoke (Python)

```python
from eilik.controller import EilikController
robot = EilikController(port='/dev/ttyACM1')   # or autodetect via detect_port()
robot.connect()

# High-level action
robot.action("message")

# Ambient displays
robot.show_clock(hour=23, minute=25)
robot.show_streak(days=7)
robot.show_pr(42, author="quinn")
robot.show_energy_meter(87)
robot.show_crypto_ticker("BTC", 5.2)

# Compound routines
robot.morning_routine()
robot.welcome_back()
robot.cron_tick_done("morning brief")
robot.error_flash("sync failed")
robot.quinn_comms()

# Custom choreography
robot.choreography([
    {"action": "good_morning"},
    {"ambient": "energy_meter", "soi_percent": 87},
    {"ambient": "weather", "condition": "sun"},
    {"motion": "wave"},
], inter_step_delay=0.3)
```

## How to invoke (HTTP)

```bash
curl -X POST localhost:8765/action -d '{"name": "hi_jeff"}' -H 'Content-Type: application/json'
curl -X POST localhost:8765/ambient/clock -d '{"hour": 23, "minute": 25}' -H 'Content-Type: application/json'
curl -X POST localhost:8765/event/cron_done -d '{"name": "morning"}' -H 'Content-Type: application/json'
curl -X POST localhost:8765/morning -d '{}' -H 'Content-Type: application/json'
```

## How to invoke (CLI)

```bash
.venv/bin/python -m eilik.cli action --name heart_eyes
.venv/bin/python -m eilik.cli ambient --ambient clock --text 23:25
.venv/bin/python -m eilik.cli list_actions
.venv/bin/python -m eilik.cli cron_done --name "morning brief"
```

## Display Format Reference

Eilik has a 128×64 1bpp monochrome SSD1306 page-mode display. That's 8 pages ×
128 columns, bit 0 = top row, bit 7 = bottom. `read_display` returns the raw
1024-byte framebuffer; `write_display` accepts the same.

Display ACK: `aa aa aa 05 00 a4 01 55` (status byte = 0x01 = success). Status
byte lives at `frame[6]` (not `body[1]`).

### CRITICAL: user-display mode lock

The firmware's idle animation overwrites any custom `cmd=0xA4` display write
within ~50ms. To make a custom face stick, the SDK automatically sends
`cmd=0xA6` with running_number=100 BEFORE every `write_display()`. After
`auto_idle` pushes the firmware idle face, the SDK sends `cmd=0xA6` with
running_number=0 to release the lock.

If you call `write_display()` directly (bypassing `display_image()`), you
must do this yourself or the custom face will be instantly overwritten. The
SDK's wrapper handles it.

### CRITICAL: 180° display rotation

Eilik's OLED panel (or firmware) renders the framebuffer rotated 180° from
what the SDK reads back. So when you build a framebuffer from a PNG and send
it raw, the image appears upside-down on the physical screen.

To fix, the SDK's `display_image()` and `_display_image_raw()` apply a 180°
rotation to the framebuffer before sending. This rotation cancels out the
firmware's rotation, so the PNG's top-left appears at the screen's top-left.

If you call `write_display()` directly (bypassing `display_image()`), you
must apply `rotate_180()` from `tools/png_to_framebuffer.py` yourself.

`display_image()` accepts `auto_rotate=False` to disable the rotation.

## Personal-use acknowledgment

This is a personal-use skill for Jeff Cechinel + Nova. There is no
relationship with Energize Lab, Energize Robotics, or the Eilik OEM. No
official support, no warranty. The author assumes the user knows what they
are doing.

## Where things live

- SDK: `/home/cechinel/.openclaw/workspace/eilik-sdk/` (private repo)
- Skill: this folder
- Skill proposal id: `eilik-body-20260812-efa8919a22`
- Brainstorm backlog: `STAGE2_IDEAS.md`

## Cron wiring (live as of 2026-08-12 23:35 BST)

The FastAPI service runs persistently at `http://127.0.0.1:8765`,
auto-restarted by a 5-min watchdog cron. Cron wrappers fire event
bridges automatically:

| Cron | Schedule | Eilik reaction |
| --- | --- | --- |
| Calendar calcurse sync | 05:50, 18:50 daily | `cron_done("calcurse-sync")` |
| Garmin daily sync | 06:15 daily | `cron_done("garmin-sync")` |
| Morning news brief | 07:18 daily | `cron_done("morning-brief")` |
| Nova Medium→LinkedIn | 09:35 daily | `cron_done("medium-linkedin")` |
| Nova system healthcheck | every 6h22m | `cron_done("nova-healthcheck")` |
| Nova daily day summary | 19:21 daily | `cron_done("day-summary")` |
| Eilik service watchdog | every 5 min | (silent) |

Failures (non-zero exit codes) trigger `error_flash` with the cron name
and exit code in the message. The wrapper is best-effort — Eilik being
offline never breaks the cron itself.

# Eilik Body Skill

OpenClaw skill for using Jeff's local Eilik robot as Nova's expressive body. Wraps the
local Eilik SDK and adds direct-chat defaults so Nova can wave, send a small message to
Eilik's face, or react physically when it's natural.

This is a personal-use skill that talks to a local reverse-engineered SDK. It has
nothing to do with Energize Lab. There is no official support, no affiliation, and no
guarantee the protocol stays stable across firmware updates.

This repository is the installable skill folder:

```text
eilik-body/
  SKILL.md
  references/display-capture-notes.md
```

## What It Does

When active, the skill teaches Nova to use the local Eilik SDK for:

- **Gestures** on direct ask: `wave`, `nod`, `shake_head`, `look_left`, `look_right`,
  `left_arm_*`, `right_arm_*`, `reset_pose`, `move_motor`.
- **Face messages** on direct ask: render short text on Eilik's SSD1306 screen, or push
  any PNG as the new face.
- **Default beats** in direct chat with Jeff (sparingly): a wave on greeting, "OK" on
  the face after a confirmation, a `nod` on agreement, etc. Nova decides based on the
  moment; never on every message, never in group chats, never during heartbeats.

Examples that should trigger it:

```text
Nova, wave with Eilik
Use your body to say yes
React with Eilik
Say hi through Eilik
Show "Hello" on Eilik
```

Text replies to Jeff still go to Telegram — Eilik is expressive feedback, not a substitute.

## Dependency

The skill expects the Eilik SDK to exist here:

```bash
/home/cechinel/.openclaw/workspace/eilik-sdk
```

Known-good SDK repo:

```text
https://github.com/strognoff/eilik-sdk
```

The SDK is up-to-date as of `main` commit `7e80015` (display_image + FastAPI endpoints).
For continuous use, keep the SDK current — pull the latest `main` before relying on
new commands.

## Install

The skill folder (`eilik-body/`) can be installed into the active OpenClaw skills
directory in two ways:

1. **Via Skill Workshop** (preferred once approval routes are available):
   the pending proposal `eilik-body-20260812-efa8919a22` is the canonical reviewed
   artifact. Apply with `apply proposal eilik-body-20260812-efa8919a22` from a session
   that has the approval route.
2. **Direct install** (fallback):
   ```bash
   cp -r eilik-body ~/.openclaw/skills/
   ```

At creation time, the Skill Workshop approval route was unavailable from this session,
so the proposal stayed pending. The textual content in `SKILL.md` is the source of
truth for the skill.

## Usage

Once active, Nova will gently use Eilik's body when talking to you in direct chat:

```text
# Direct ask
Wave at me with Eilik
Use Eilik to nod yes
Reset your Eilik body
Push this image to Eilik
Show "Hello" on Eilik's face

# Routine direct-chat moments (Nova decides)
Hi Jeff!                      ← Eilik face says "Hi Jeff!" (small greeting)
Yes, done                     ← Eilik nods and shows "OK"
Bye for now                   ← Eilik face says "Bye!"
```

Nova checks `/dev/ttyACM0`, runs the matching SDK command from the SDK directory
(using the `.venv/bin/python` interpreter), and reports the result in chat.

## Reference Files

- `eilik-body/SKILL.md` — full skill specification, ground rules, default direct-chat
  patterns, troubleshooting reference.
- `eilik-body/references/display-capture-notes.md` — display protocol details and
  capture workflow (kept for future protocol work).

## Communication Split

- **Telegram**: actual text and reasoning — every reply.
- **Eilik**: body language + small face text (≤12 chars usually).
- **SDK logs**: diagnostics in `logs/eilik.log`.

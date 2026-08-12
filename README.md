# Eilik Body Skill

Codex/OpenClaw skill for using Jeff's local Eilik robot as Nova's expressive body.

This repository is documentation plus the installable skill folder:

```text
eilik-body/
  SKILL.md
  references/display-capture-notes.md
```

## What It Does

When active, the skill teaches Nova to use the local Eilik SDK for gentle physical gestures when Jeff clearly asks for robot body language.

Examples that should trigger it:

```text
Nova, wave with Eilik
Use your body to say yes
React with Eilik
Check if your robot body is awake
```

Current communication split:

- Telegram: actual text and reasoning.
- Eilik: body language such as wave, nod, look left/right, reset.
- SDK logs: diagnostics in `logs/eilik.log`.

## Dependency

The skill expects the Eilik SDK to exist here:

```bash
/home/cechinel/.openclaw/workspace/eilik-sdk
```

Known-good SDK repo:

```text
https://github.com/strognoff/eilik-sdk
```

Known-good SDK commit:

```text
84dde26
```

## Install

Copy or install the `eilik-body/` folder into the active OpenClaw/Codex skills directory used by the agent.

The Skill Workshop proposal for this skill is:

```text
eilik-body-20260812-efa8919a22
```

At creation time, applying the proposal from Telegram failed because the Skill Workshop approval route was unavailable. The proposal remains the canonical reviewed artifact until it can be approved in an environment with an approval route.

## Usage

Once active, ask Nova for a physical Eilik action:

```text
Wave at me with Eilik
Use Eilik to nod yes
Reset your Eilik body
```

Nova should check `/dev/ttyACM0`, run the matching SDK command from the SDK directory, and then report the result in chat.

## Display / Screen Status

The captured firmware-update file does include a display-relevant clue: during the update, the official app sends many large `cmd=03` binary/resource frames and Eilik displays update/progress state on its own screen.

That is not the same as an arbitrary text command. No simple "print this text on the face screen" command has been identified yet.

See `eilik-body/references/display-capture-notes.md`.


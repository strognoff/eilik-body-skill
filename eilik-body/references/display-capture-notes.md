# Eilik Display Capture Notes

Use this reference when Jeff asks about Eilik screen output, custom display text, emotions, firmware-update visuals, or reverse engineering display control.

## Capture Studied

```text
/home/cechinel/.openclaw/workspace/eilik-sdk/captures/eilik-official.pcapng
```

This capture was taken while the official Eilik app performed a firmware/update flow and Eilik showed update/progress output on its own screen.

## High-Level Findings

- The capture contains 1,586 USB payload frames.
- It contains 1,435 Eilik protocol frames beginning with `aa aa aa`.
- The robot enumerates first as device 7, then re-enumerates as device 9.
- The official app starts with status/session commands:
  - `aa aa aa 04 00 01 fa`
  - `aa aa aa 04 00 20 db`
  - `aa aa aa 09 00 02 00 09 00 00 00 eb`
- It then sends many small `cmd=03` resource path frames such as `a/0/01/00/01`.
- During the update-looking part, it sends large `cmd=03` OUT frames:
  - 53 frames of 5,132 bytes
  - 26 frames of 524 bytes
  - 2 frames of 1,036 bytes
  - 1 frame of 3,596 bytes
  - 1 frame of 2,060 bytes
- Those large frames total about 293 KB and are sent over roughly 7 seconds.

## Interpretation

The firmware/update display shown on Eilik is probably triggered by the update/resource transfer state, not by a simple arbitrary text command.

The most suspicious update/display-related pattern is:

```text
aa aa aa <large length> 03 02 ...
aa aa aa 05 00 03 03 f4
aa aa aa 06 00 03 03 01 f2
```

That looks like bulk binary/resource transfer plus commit/ACK framing.

The capture does not reveal a clean command like:

```text
display_text("hello")
draw_text(...)
set_screen(...)
```

There are no obvious printable strings such as `update`, `firmware`, `screen`, `display`, `font`, `text`, or the displayed on-screen words inside the large OUT payloads.

## What To Do Next

To reverse engineer display control, capture a smaller official-app action that changes only the face/display, if one exists. Ideal captures:

- Trigger one built-in emotion.
- Trigger one known face animation.
- Change one screen state without firmware update.
- Run the official app action twice with different selected emotion/state.

Then compare frames around the changed action. Avoid firmware update captures for first-pass display work because they mix resource sync, reboot/re-enumeration, and bulk binary transfer.

## Current SDK Policy

Do not add arbitrary text/display methods to the Eilik SDK yet.

It is reasonable to add future methods for named built-in expressions if a compact command can be isolated, for example:

```python
robot.expression("happy")
robot.expression("thinking")
robot.expression("update")
```

Only implement those after a capture identifies a small repeatable frame sequence.


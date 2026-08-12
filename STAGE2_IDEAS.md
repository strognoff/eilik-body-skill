# STAGE 2 — Eilik body extension (Jeff's brainstorm 2026-08-12 22:58 BST)

Jeff asked: "every time you sent me a message, you can always shake the body and write 'Message!!' .. and what else can we add as a Skill for you? Be creative.. create more skills, more actions, we need Eilik to be your body that represents you .. and your interaction with me!"

## Shipped in this turn

- **Message received beat** — every reply from Nova in direct chat with Jeff → wave + face "Message!! :)". Implemented live at 22:58 BST 2026-08-12. Capture: `captures/eilik-display-message.png`. SDK wraps wave + face flash into one call.

## Stage 2 ideas — pick from tomorrow

### A. Emote / motion expansion (high priority, quick wins)

- [ ] **Thumbs up** — both arms flick up + face shows "👍"
- [ ] **High five / fist bump** — one arm horizontal hand-up
- [ ] **Shrug** — both arms out, face "¯\_(ツ)_/¯"
- [ ] **Bow** — head dip + arms sweep
- [ ] **Dance / wiggle** — body wiggle loop
- [ ] **Peek** — head tilt with one eye winking
- [ ] **Spin** — 360° body rotation (if the servo range allows)
- [ ] **Nod emphatically** — three quick nods for "yes absolutely"
- [ ] **Shake head** — three quick shakes for "no way"
- [ ] **Clap** — quick arm clap (if servo reach allows)
- [ ] **Heart hands** — both arms up forming a heart
- [ ] **Surprise** — quick jump-back + face "!"

### B. Face expressions (broaden the alphabet)

- [ ] **Thinking** — "🤔" or "hmm..." with antennas wag
- [ ] **Frustrated** — ">:(" with one arm slumping
- [ ] **Excited** — "!!!" with both arms up
- [ ] **Tired** — "Z Z Z" sleep face + body droop
- [ ] **Working** — face "..." + antennas wiggling (Nova is processing)
- [ ] **Done** — face "✓" or "OK!" + thumbs up gesture
- [ ] **Confused** — "?" face + head tilt
- [ ] **Laugh** — face "哈哈" + body shake
- [ ] **Heart eyes** — "♥_♥" + small wave
- [ ] **Crying** — T_T face + one arm wipe (moody)
- [ ] **Angry** — >:( with one arm shake
- [ ] **Cool** — sunglasses face + one arm pointing

### C. Contextual displays (ambient info on Eilik's face)

- [ ] **Time on face** — periodically show the time (e.g. every 5 min in idle state if Jeff is at the desk)
- [ ] **Weather face** — sunny/rainy/cloudy icon when asked "what's the weather?"
- [ ] **Day streak / habits** — when Jeff opens his morning brief, flash "🔥 X day streak" briefly
- [ ] **Code review watch** — when a new PR opens, show the PR number + author on Eilik's face
- [ ] **TTS carole** — read Jeff's last message aloud with a `🔊` speaker face
- [ ] **Crypto ticker** — when a target coin moves >5% in an hour, flash the arrow + ticker on Eilik's face
- [ ] **Quote of the day** — morning brief scroll could include a quick Eilik flash of a pull quote

### D. Live event bridges (Eilik reacts to ambient activity)

- [ ] **Notification chime** — when email arrives at the high-priority inbox, Eilik pops an "@" face briefly
- [ ] **Calendar nudge** — 5 min before a meeting, face shows the meeting title + clock
- [ ] **Cron tick** — every time a cron completes, Eilik shows a tiny ✓ for ~1s and returns to idle
- [ ] **Sub-agent spawn** — when Nova spawns a subagent, Eilik's antennas wag (thinking) until the result returns
- [ ] **Error flash** — when anything crashes, Eilik flashes an "X" briefly with a sad face
- [ ] **Tarot / fortune** — at idle, occasionally show a fortune face like "Today: ship something"

### E. Eilik-as-Nova-personality skills

- [ ] **Mood ring** — Eilik's face changes based on Nova's working sentiment (charged vs tired vs zen)
- [ ] **Energy meter** — face shows "SoI: 87" as a battery/heart icon
- [ ] **Quinn-comms** — when Quinn is mentioned in chat or pings Nova, Eilik does a small wave toward the screen direction
- [ ] **Hourly status pulse** — every hour, Eilik wiggles antennas and shows current time + weather
- [ ] **Welcome home** — when Jeff opens chat after >2h silence, Eilik dances and shows "Welcome back :)"

### F. Game / interactive skills

- [ ] **Trivia flash** — Nova pushes a question to Eilik's face, Jeff nods yes / shakes no to answer
- [ ] **Reaction time game** — push random numbers, Jeff taps to ack, Eilik shows the response time
- [ ] **Coin flip** — nods = heads, shakes = tails, Eilik shows the result
- [ ] **Wordle hint** — when Jeff is doing daily wordle, Eilik shows a hint face
- [ ] **Chess status** — after each chess habit, Eilik shows win/loss record face

### G. Compound actions (multi-beat choreography)

- [ ] **Good morning routine** — antennas up + stretch + "Morning!" face → pause → brief pause → "Today: X items" face
- [ ] **Task completion** — ✓ face + arms up + happy wiggle (when cron completes something good)
- [ ] **Thinking handoff** — Nova is about to delegate: shows "..." face + antennas wag → ⓘ + name when subagent returns
- [ ] **Failure apology** — when Nova makes a mistake: head drop + "sorry" face + small bow

## Priority for tomorrow (Jeff, please pick)

Pick 2-3 from each bucket (A, B, C, D, E) for tomorrow's session. The compound actions (G) and the games (F) are heavier — maybe a 2-session project.

## Open technical questions

1. **Servo-only motions vs animation** — most of A requires new servo positions. Have to discover what `move_motor` ranges work without binding.
2. **Face text constraints** — at 16pt the display fits ~8 chars before clipping. Need to confirm font choice works for non-ASCII (Japanese, emoji).
3. **Drawing speed** — pushing a new face for every cron completion might wake Eilik's display too often. Maybe batch to once per ~30s of activity.
4. **Battery / reliability** — running this many animations might heat up Eilik. Should we add a "rest" cooldown?

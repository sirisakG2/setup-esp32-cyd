# Setup: ESP32-2432S028 ("Cheap Yellow Display" / CYD)

A log of the problems hit (and fixes found) while bringing up an ESP32-2432S028R
"Cheap Yellow Display" board on macOS with PlatformIO + LovyanGFX — from USB
detection through display bring-up to touch bring-up and calibration.

Board: ESP32-D0WD-V3, 240x320 ILI9341 SPI display, resistive XPT2046 touch,
CH340C USB-UART bridge.

The working firmware this troubleshooting led to lives at
[sirisakG2/esp32-cyd-macropad](https://github.com/sirisakG2/esp32-cyd-macropad).

## Board spec sheet + live test

### 🔌 [Open the spec sheet and live test](https://sirisakg2.github.io/setup-esp32-cyd/cyd-spec.html)

One web page with everything about this board: the full specs, a map of
what every pin (GPIO) is connected to, and links to useful guides.

**The live test** checks your board through the USB cable, right in the
browser:
1. Plug the board into the computer.
2. Open the page in **Chrome, Edge or Opera** (Safari and Firefox can't do
   this part).
3. Press **Connect board** and choose the port with `usbserial` in its name.
4. The checklist fills in by itself: it shows whether the computer can see
   the board (which proves the cable carries data), which USB chip the
   board uses, and, after you press **Reset board**, whether the ESP32
   answers with its start-up message.

This is also a quick way to test several cables: a cable that only charges
never produces a port in the list. Specs are labelled **verified** (checked
on this exact board), **check** (probably true, confirm with the live
test) or **docs** (from public documentation, can differ between batches).

## The 3 problems

Getting this board fully working (screen showing something, and touch
responding accurately) took three separate rounds of troubleshooting. Each
one is filed as a GitHub Issue with the full symptom → investigation →
cause → fix writeup; here's the short, plain-language version of each.

### [#1 — Board not detected on macOS](../../issues/1)
**The problem:** Plugged the board into the Mac over USB, its little LED lit
up (so it's clearly getting power), but the Mac never showed it as an
available port at all — not in Arduino IDE, not even at the command line.

**Why it happened:** The USB-C cable being used only had power wires inside
it, not data wires. This is extremely common — a lot of cables that ship
with gadgets (or that you'd use "just to charge something") are physically
missing the wires needed to actually send information, even though they
look identical to a cable that can. A lit-up LED only proves the board is
receiving electricity; it proves nothing about whether data can flow.

**The fix:** Swapped in a different cable (one already known to work for
transferring files, not just charging), and the board immediately appeared.
**Lesson for next time:** if a USB device won't show up *at all*, suspect
the cable before anything else — before drivers, before software, before
the board itself.

### [#2 — Touch not responding](../../issues/2)
**The problem:** The screen displayed things correctly (so the board and
the display definitely worked), but tapping the touchscreen never did
anything — the code just never noticed a touch had happened.

**Why it happened:** This board actually has *two separate small chips*
doing two separate jobs: one draws things on the screen, and a completely
different one senses finger touches. Wiring code had accidentally been
written as if both chips shared the same set of connections (which is true
on some other similar boards), when in fact this specific board wires them
to entirely separate sets of pins. The touch-sensing chip was effectively
being asked questions over wires it was never actually connected to — so
of course it never answered.

**The fix:** Pointed the touch code at the correct, separate set of pins
this board actually uses for its touch chip. Confirmed the fix by watching
a live log while tapping the screen and seeing the numbers actually change
in response — proof the two chips were finally talking.

### [#3 — Touch coordinates flipped/rotated](../../issues/3)
**The problem:** Progress! Now tapping the screen *did* register — but the
dot that was supposed to appear right under your finger showed up somewhere
else entirely (off to the side, or upside down from where you actually
touched).

**Why it happened:** The touch-sensing chip reports raw numbers in
whatever orientation it happens to be physically glued down in — it has no
idea the screen itself is displaying things sideways or upside-down
relative to that. On this particular board (and this can vary from unit to
unit), the touch chip's sense of "left/right" and "up/down" was completely
swapped and reversed compared to the screen's. Guessing at a handful of
preset "try this rotation" options one at a time never quite lined up.

**The fix:** Instead of guessing, the actual relationship was measured
directly — by tapping each of the four corners of the screen one at a time
while logging exactly what raw numbers the touch chip reported for each
corner, then working out the pattern from real data. That pattern was then
hardcoded into the code as a small conversion formula. **Lesson for next
time:** when something reports coordinates that don't match reality, measure
the real relationship from a few known points rather than guessing at
built-in rotation settings.

## Claude "Skill" test reports

While fixing the three issues above, everything that was learned got packaged
into a reusable **Claude Skill** called `esp32-cyd-bringup`. A "skill" is
just a saved instruction sheet — the next time anyone (on any project) asks
Claude for help with this exact board, Claude reads that instruction sheet
first instead of figuring everything out from scratch again. Think of it
like a lab notebook that Claude keeps and automatically flips open whenever
the topic matches.

Before trusting that instruction sheet, it was tested: the same three
problems from the [Issues](../../issues) tab were asked twice each — once
with Claude allowed to read the instruction sheet, once without (a "blind"
baseline) — and the two sets of answers were compared and scored. This repo
includes the two report pages generated from that testing, so the process is
visible instead of just taking the result on faith.

### 📊 [Eval review results](https://sirisakg2.github.io/setup-esp32-cyd/skill-eval/review.html)
**What it is:** A side-by-side comparison of Claude's answers to the three
CYD problems *with* the skill loaded vs. *without* it (the plain baseline).

**How to read it:**
- The **Outputs** tab lets you click through each of the 3 test questions and
  read both answers directly — this is the most useful tab if you just want
  to see "did having the skill actually help, and how."
- The **Benchmark** tab turns that comparison into numbers: how many of the
  checklist items ("assertions" — specific things a good answer should
  mention, like *"suggests trying a different cable"*) each answer got
  right, plus how long each answer took and how much it "cost" in tokens
  (tokens are just the unit Claude is billed/measured in, roughly like
  words). In this test, the skill-assisted answers got every checklist item
  right (13 out of 13), while the baseline missed a few (10 out of 13) —
  mostly smaller details like explaining *why* a lit-up LED doesn't prove
  the USB cable is actually transferring data.

### 🎯 [Trigger eval set editor](https://sirisakg2.github.io/setup-esp32-cyd/skill-eval/eval_review_esp32cydbringup.html)
**What it is:** Not a report of answers — this one is a checklist of 20
example questions used to test *when the skill should automatically wake
up*. Claude decides on its own, behind the scenes, whether a given question
matches a skill closely enough to use it — there's no button the user has to
press.

**How to read it:** Each row is one example message a user might type, plus
a toggle saying whether the skill *should* switch on for that message.
Half the questions are obvious matches (someone directly mentioning "Cheap
Yellow Display" or "CYD" or the touch/display symptoms above), and half are
deliberate near-misses — questions that *sound* similar but are actually
about a different board or a different problem entirely (e.g. a Raspberry
Pi touchscreen, a totally different ESP32 board, or just a general
spec question) — to make sure the skill doesn't accidentally trigger where
it shouldn't. This page is also an editable tool: it can be opened, rows can
be tweaked or added, and an updated question list can be exported to refine
the skill further later.

Both pages are static HTML — no server needed, they just open and run
entirely in the browser tab.

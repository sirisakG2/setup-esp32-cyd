# Setup: ESP32-2432S028 ("Cheap Yellow Display" / CYD)

A log of the problems hit (and fixes found) while bringing up an ESP32-2432S028R
"Cheap Yellow Display" board on macOS with PlatformIO + LovyanGFX — from USB
detection through display bring-up to touch bring-up and calibration.

Board: ESP32-D0WD-V3, 240x320 ILI9341 SPI display, resistive XPT2046 touch,
CH340C USB-UART bridge.

See the [Issues](../../issues) tab for each individual problem, its root cause,
and the fix.

The working firmware this troubleshooting led to lives at
[sirisakG2/esp32-cyd-macropad](https://github.com/sirisakG2/esp32-cyd-macropad).

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

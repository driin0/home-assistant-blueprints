# Changelog

## 2.1.0 — 2026-08-11

### Added

- **`automatic_switch_restart`** joins the repository. It watches a switch and
  restarts it after it turns itself off, with growing delays and a limited
  number of attempts. It was previously published as a Gist, which stays where
  it is, in Italian, for anyone who imported from it — the version maintained
  from now on is this one.

### Fixed in that blueprint, compared with the Gist

Three defects, all of which could bite in normal use:

- **A wait with no timeout could hang the automation permanently.** If the
  connectivity sensor was configured and the device never came back, the cycle
  stopped there — and since the automation runs in single mode, a hung cycle
  blocks every later trip from being handled at all. Worse, the control boolean
  is switched off at the start of a cycle, so the fail-safe turned from
  "restarting stays disabled until you re-enable it" into "restarting stays
  disabled and re-enabling it does nothing". Every wait is now bounded.
- **The exponential backoff had no ceiling.** With a 20 s delay, a multiplier of
  3 and 8 attempts, the last attempt landed 18 hours after the trip; the
  selectors allowed combinations reaching centuries. A configurable maximum
  delay now caps it.
- **One outcome was silent.** "The device never became reachable" was the only
  one of four results that sent no notification. It now has its own message.

### Changed in that blueprint

- Waits check the **state** rather than a transition. The original waited for a
  transition to `off`, which only occurs if the device actually went away —
  correct where a power cut takes the network down with it, a permanent hang
  where the device stays reachable.
- The connectivity sensor accepts a **`device_tracker`** as well as a
  `binary_sensor`, and "reachable" means `on` or `home`.
- The settle time after restarting is configurable instead of a fixed 10 s.
- `mode: single` is explicit, with the reason written next to it.

### Changed in both blueprints

- Each declares **`author`** and a **minimum Home Assistant version** (2024.6.0:
  sections in the configuration UI, and the notify entity platform). Without it,
  an older installation fails with an error that says nothing about the cause.

## 2.0.1 — 2026-08-11

### Changed

- **Licence changed from GPL-3.0 to MIT.** The GPL is written for software that
  is compiled and linked, and what counts as a derivative work of a declarative
  YAML file is unclear at best. MIT is what the Home Assistant blueprint
  ecosystem uses, and it removes any hesitation for anyone wanting to start from
  this blueprint to write their own — which is how blueprints spread.

  Copies already obtained under GPL-3.0 remain available under those terms; that
  cannot be revoked. Everything from this version onwards is MIT.

## 2.0.0 — 2026-08-11

A rewrite of how the blueprint applies colour. It used to send `light.turn_on`
to every light once a minute, unconditionally. It now works out which lights are
actually wrong and writes only to those — so in the steady state it sends
nothing at all, and while the lights are unreachable it sends nothing either.

### Breaking

**Lights found switched off are no longer switched back on**, unless you say
they are behind a relay.

The previous version turned lights on every minute whether they were on or not,
so a light could not be switched off while the automation was active. That was
never documented; it was a consequence of writing unconditionally. It is the
right behaviour only when something upstream cuts the power — a relay, a
breaker, a smart switch — because there "off" means "no power", not "the user
switched it off".

**If your lights are behind a relay, you must act after updating.** Open each
automation using this blueprint and turn on **"Lights are powered through an
upstream relay"**. Existing automations take the new default, which is off, so
without this the lights will not be restored after a power cut.

If your lights are driven directly by Home Assistant, do nothing: the new
default is what you always wanted, and switching a light off now leaves it off.

### Added

- **Colour modes.** Besides the per-weekday colours, there are now *Rainbow*
  (the seven rainbow colours on the seven days), *Rotate configured colours*
  (cycles through the distinct colours you set) and *Rotate full spectrum*
  (twelve hues, no colour configuration needed). The rotating modes never repeat
  the previous day's colour and spread evenly over time.
- **Colours by name.** Each day accepts a colour name from a dropdown — `red`,
  `yellow`, `blue` and seventeen more — which takes precedence over the picker.
- **Colour offset.** Shifts the colours along. In Rainbow it chooses which day
  red falls on; in the rotating modes it sets where an automation starts in the
  sequence, so several zones can run the same mode without ever showing the same
  colour as one another.
- **Distrust window** (120 s). A light that has just reconnected may still be
  reporting stale values, so it is written without being trusted for a while.
- **Periodic reassert** (15 min). Everything is rewritten regardless at
  intervals, as a safety net against a light being wrong in a way that cannot be
  observed. Set to 0 to disable.
- **Sections.** The forty-one settings are grouped and the seldom-used ones are
  collapsed.

### Fixed

- **Lights that cannot do colour are skipped.** Previously they were sent a
  colour every minute forever: a light with no colour mode can never match the
  wanted value, so it counted as diverging on every cycle. Colour-temperature
  bulbs were the worse case — they report a colour *derived* from their
  temperature, which passes an existence check and still never matches.
- **Unreachable lights are skipped** instead of being commanded into the void.

### Changed

- The time-window settings were called "Start/End Time for Automation" and read
  as "when I want the lights on", while they mean "when this automation works".
  Renamed, and every description now states what it does not do. **This blueprint
  never switches lights on or off by itself** — that stays with whatever already
  handles it in your setup.
- Groups are expanded to their members before comparing. A group's own colour is
  an average across its members and matches no individual bulb, so it cannot be
  compared against.

### Notes

Upgrading keeps your existing settings: input names did not change, and the new
settings take their defaults. Only the relay setting needs attention, as above.

## Earlier

Versions before 2.0.0 were not numbered. The blueprint was first published on
2024-10-15, and a configurable time range was added on 2024-10-18 — at which
point `fixed_start_time` was renamed to `threshold_time`. The full history is in
the commit log.

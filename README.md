# Home Assistant blueprints

Blueprints for Home Assistant. Import one with the button next to it, or paste
the URL of its `.yaml` file into **Settings → Automations & scenes → Blueprints
→ Import blueprint**.

| Blueprint | What it does |
|---|---|
| [Daily light colour adjustment](#daily-light-colour-adjustment) | Holds a colour and brightness on your lights, changing it by day |

---

## Daily light colour adjustment

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdriin0%2Fhome-assistant-blueprints%2Fblob%2Fmain%2Fdaily_automatic_light_color_adjustment.yaml)

Keeps a set of colour lights at the colour and brightness you want, with a
different colour each day of the week. Requires colour-capable lights.

It is not a "set it once" automation. It **holds** the colour: if a light is
power-cycled, misses a command, or is changed by something else, it is put back
within a minute. That matters for lights fed through a relay or breaker, which
come back at their own default whenever power returns.

### How it works

The automation runs once a minute and, each time, works out which lights are
**actually** wrong — then writes only to those. When everything already matches,
it sends nothing at all. Groups are expanded to their members, because a group's
own colour attribute is an average across its members and matches no individual
bulb.

Three deliberate exceptions to "only write what diverges":

- **Just reconnected.** A light that came back within the last couple of minutes
  may still be reporting its previous values, so it is written without being
  trusted. The window is configurable.
- **Periodic reassert.** Every so often (15 minutes by default) everything is
  rewritten regardless, as a safety net against a light being wrong in a way
  that cannot be observed. Set it to 0 to switch this off.
- **Unreachable lights are skipped**, and retried on a later cycle. Nothing is
  ever forced at a light that is not there.

Lights that cannot do colour are skipped entirely — see *Requirements* below.

### Colour modes

| Mode | Behaviour |
|---|---|
| **Per weekday** | The colour you configured for that day. The classic behaviour. |
| **Rainbow** | The seven rainbow colours on the seven days, ignoring your configuration. Red falls on Monday unless you move it. |
| **Rotate configured colours** | Cycles through the distinct colours you configured. |
| **Rotate full spectrum** | Cycles through twelve vivid hues. Needs no colour configuration at all. |

The rotating modes are **deterministic, not random**, and that is on purpose:
the automation re-evaluates every minute, so an actual random draw would repaint
the lights continuously. The colour is a function of the date — the same all
day, and **never the same as the previous day**.

Every block of N days contains each colour exactly once, so the distribution is
even and no colour is starved. With four colours a repeat comes on average every
4 days; with the twelve-colour spectrum, every 12.

**Colour offset** shifts the colours along, and works in Rainbow too.

In **Rainbow** it decides which colour lands on which day: each unit moves red
one day later, so 0 puts it on Monday, 1 on Tuesday, and 6 on Sunday. Seven
colours means 0-6 covers everything and 7 is the same as 0.

In the **rotating modes** it sets where an automation starts in the sequence.
Zones with different offsets are guaranteed never to show the same colour **as
each other** on any day, while each still never repeats its own previous day.
With the twelve-colour spectrum, offsets such as 0, 3, 6 and 9 keep four zones
far apart on the colour wheel.

### Colours by name

Each day accepts a colour **name** instead of the picker — `red`, `yellow`,
`blue`, and seventeen more, chosen from a dropdown. When set, the name wins over
the picker for that day.

It is a fixed list rather than every CSS colour name, and that is a real
constraint rather than an oversight: Home Assistant's template engine has no
colour-name conversion, so the blueprint has to resolve a name to RGB itself in
order to compare it against what a bulb reports. A name it does not know could
not be compared, and would be resent every minute.

### Lights behind a relay

If your lights are fed through a relay, breaker or smart switch that cuts their
power, turn on **"Lights are powered through an upstream relay"**. A light found
switched off but reachable is then switched back on, because in that setup the
desired state is "lit".

Leave it **off** — the default — when your lights are driven directly by Home
Assistant, so that switching one off actually leaves it off.

### What it does not do

**It never switches lights on or off by itself.** It adjusts lights that are
already on. Switching stays with whatever handles it in your setup — a
schedule, a relay, a scene. The "active period" setting only limits *when this
automation works*; outside it, lights are left exactly as they are.

This is a deliberate split. Switching in practice carries conditions a time
window cannot express — fire before sunset, but only if nobody armed the alarm,
and only if a seasonal toggle is on — and a blueprint that owned both would
either reimplement all of that badly, or fight with the automation you already
have for the same on/off state.

### Requirements

- **Colour-capable lights.** Bulbs that only do brightness or colour temperature
  are skipped: a colour cannot be applied to them, and a colour-temperature bulb
  reports a colour *derived* from its temperature that could never match, which
  would make the automation rewrite it every minute forever.
- Light **groups are supported** and expanded to their members.
- A day switched off in the configuration is simply left alone: the previous
  colour stays until the next enabled day.

### Configuration reference

Everything is grouped into sections, and the defaults suit most setups.

| Setting | Default | Notes |
|---|---|---|
| Lights to adjust | — | Lights or light groups |
| Optional boolean entity | empty | An `input_boolean` that enables the automation |
| Colour mode | Per weekday | See above |
| Colour offset | 0 | Shifts the colours. Works in Rainbow and in the rotating modes |
| Lights behind an upstream relay | off | See above |
| Distrust window | 120 s | Write without comparing this soon after a light reconnects |
| Full reassert every | 15 min | 0 disables it |
| Threshold time | 12:00 | When a new day begins — also selects which day the rotating modes are on. Pick a time when the lights are normally off. |
| Active period | all day | When this automation works. Does **not** switch lights. |
| Per-day colour, name, brightness, enable | — | Brightness always follows the weekday, in every mode |

## Licence

GPL-3.0. See [LICENSE](LICENSE).

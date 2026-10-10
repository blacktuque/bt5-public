# Saber firmware

Five firmware images for the saber badge (RP2040). Each is a stripped ARM
ELF: flash it as-is, or load it into a disassembler at its real addresses.
They all run on the same badge and keep the same saved data, so you can
switch between them and keep your name, side and duel record.

| File | For | In one line |
|---|---|---|
| `saber-fw` | Attendees | The conference badge firmware. |
| `saber-social-fw` | Attendees | `saber-fw` plus a Social tile and a RAINBOW blade. |
| `saber-play-fw` | Families, after the con | The saber as a toy: games with best scores, everything unlocked. |
| `saber-admin-fw` | Organizers | `saber-social-fw` plus the full Organizer menu. |
| `saber-kiosk-fw` | The scoreboard | Collects progress reports instead of playing. |

## Flashing

Any of them, with the SWD probe described in the
[main README](../README.md#flashing-the-saber):

```sh
cd ../tools
./bt5-badge-x86_64-linux backup                     # do this first
./bt5-badge-x86_64-linux flash ../saber-fw/saber-play-fw
```

or with `probe-rs`:

```sh
probe-rs download --chip RP2040 --speed 1000 --binary-format elf saber-fw/saber-play-fw
```

Flashing never touches the badge's saved data, its identity or its fleet
key. To go back, flash another image.

---

## `saber-fw`: the conference firmware

What every attendee's badge ran at Blacktuque 5. A fresh badge starts in
calibration: pick a language, enter a handle, answer the allegiance
questions, then wait for the opening ceremony to activate it. After that:
the lightsaber (ignite, hum, swing and clash sounds), duels over IR, the
challenge list, droids to report to and hunt, talk attendance, light shows
from the organizers' remotes, and the Service Record.

## `saber-social-fw`: the social build

`saber-fw` with two additions: a **Social** home tile that sends the
light-show patterns from any badge, and a **RAINBOW** blade colour in
Settings.

## `saber-play-fw`: the family build

The saber after the con, for kids and families. Nothing to earn, nothing to
wait for.

**First boot** asks three things: a language, a side (**Rebel** or
**Empire**, which becomes the blade colour), and a name. A badge that
already went through the con keeps its name, side, duel record and
settings, and goes straight to the home screen.

**Home screen:** LIGHT SABER, then Settings, Games and Record. Push on the
saber ignites it.

**Games** (the Games tile, or the Challenges button from anywhere), each
keeping its best score:

| Game | How to play | Score |
|---|---|---|
| Bop it | Do what the saber calls out (bop, twist, pull, slash, jab) before time runs out. It speeds up and keeps going until you miss. | Rounds |
| Swing rush | Swing as many times as you can in 30 seconds. | Swings |
| Helicopter | Spin the saber fast over your head and keep it spinning. | Longest spin |
| Focus your force | Hold perfectly still until the blade charges. Partway through it flashes: press Push within 2 seconds to keep going. | Fastest time |
| Stormtrooper Patrol | Walk 300 steps carrying the lit blade. | Fastest time |
| Spin Text | Spin the saber and it writes your name, or your battle record, in the air. | — |

Every game ends on a score screen, with a chime and NEW BEST! when you beat
your record. Any button goes back to the game, and Push plays it again.

**Record** shows your duel wins and losses. Push twice to reset them.

**Settings:** volume, brightness, sound set, language, name, blade colour
(your side, a custom colour, or rainbow), ignite style, clash colour and
side. Every option is unlocked. **Reset** puts the saber back to first boot:
hold Push for 3 seconds while the bar fills.

**Battles** work as before, against family sabers and sabers still on the
conference firmware. One difference: the conference firmware doubled an
organizer badge's score and this one doesn't. In a duel between a family
saber and a conference saber where either side is an organizer badge, the
two can disagree on who won.

**Setup Broadcasts** appears only on a badge whose role is organizer (set it
with `bt5-badge identity`). It sends quiet, loud, light shows and the rick
roll to every saber in range. **Reset Others** sends a reset to one saber by
its 4-digit id. Like the admin firmware's targeted rows, it needs a fleet
key on the sending saber and on each target (see
[Admin firmware](../README.md#admin-firmware)), and it sends nothing without
one.

**Not in this build:** droids, talks, Hacker Jeopardy, the CTF flags,
progress reports to the scoreboard, organizer grants and role changes, and
the anti-cheat effects. The bench remote and light-show remote work as
before. The Jeopardy remote's decorative patterns still play, but its round,
silence and max-volume keys do nothing.

## `saber-admin-fw`: the organizer build

What the organizers ran: `saber-social-fw` with the **Organizer** tile
always on, whatever the badge's role, booting straight to the home screen
even on a badge that was never calibrated. Its broadcasts (light shows,
songs, the opening ceremony and the rest) reach every badge in range. Its
targeted rows (GRANT, TALK, NEURALYZE and SET ROLE) need a fleet key; see
[Admin firmware](../README.md#admin-firmware).

## `saber-kiosk-fw`: the scoreboard kiosk

Turns a saber into the receiver for the
[scoreboard](../scoreboard/README.md): it listens for progress reports,
checks each one, and keeps them for `kiosk-host` to read over SWD. It
doesn't play. It has no key built in; load one with `bt5-badge kiosk-key`.

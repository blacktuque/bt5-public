# Saber firmware

Five firmware images for the saber badge (RP2040). Each is a stripped ARM
ELF: flash it as-is, or load it into a disassembler at its real addresses.
They all run on the same badge and keep the same saved data, so you can
switch between them and keep your name, side and duel record. The one
exception is going back from `saber-play-fw`; see
[Going back](#going-back).

| File | For | In one line |
|---|---|---|
| `saber-fw` | Attendees | The conference badge firmware. |
| `saber-social-fw` | Attendees | `saber-fw` plus a Social tile and a RAINBOW blade. |
| `saber-play-fw` | Families, after the con | The saber as a toy: games with best scores, everything unlocked. |
| `saber-play-fw.uf2` | Post-con sabers with USB-C | `saber-play-fw` as a UF2 file, for drag-and-drop over USB. |
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

### Over USB-C (post-con sabers)

The post-con sabers have a USB-C port and take `saber-play-fw.uf2` with no
tools at all, from Windows, macOS or Linux. The port carries data only, so
put the batteries in and switch the saber on first. Then hold **bootsel**,
press and release **reset**, and let go of bootsel: a drive called `RPI-RP2`
appears. Copy `saber-play-fw.uf2` onto it. The drive closes by itself and
the saber restarts on the new firmware.

The UF2 is made from the `saber-play-fw` ELF here, byte for byte, and covers
only the firmware (it ends at `0x10330d1c`), so the saved data and identity
at the end of flash are kept. The conference badges have no USB port.

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

**Home screen:** LIGHT SABER, then Settings, Games, Record and Flashlight.
Push on the saber ignites it. The Challenges button takes you back to the
home screen from anywhere.

**Games** (the Games tile), each keeping its best score:

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

**Flashlight:** Push on the Flashlight tile lights every LED on the back of
the blade white. Up and Down set the brightness (1 to 10) and any other
button turns it off. It comes back at the level you left it, and the saber
doesn't go to sleep while it's on.

**Settings:** volume, brightness, name, side, blade colour (your side, a
custom colour, or rainbow), sound set, ignite style, clash colour, language,
sleep, ID and reset. Every option is unlocked. **Reset** puts the saber back to first
boot: hold Push for 3 seconds while the bar fills.

**Saber ID:** the 4-digit number on the home screen (`#7012`) is what duels
call a saber by, so two sabers in the same room shouldn't share one.
- Sabers handed out after the con come numbered from `#7001` up, so no two
  of them match.
- A conference badge keeps the number it had.
- A saber with no number gives itself one the first time it starts, worked
  out from its flash chip's serial, in `#4000`–`#6FFF`. The same saber always
  works out the same number, but two different sabers can land on the same
  one.
- If two sabers turn out to match, open Settings, go to **ID**, and on one of
  them hold Push for 3 seconds. It picks a new random number in
  `#4000`–`#6FFF` and restarts. Name, side, record and settings are kept. The
  old number can't be had back this way.

**Sleep:** left still with no button pressed for 2 minutes (or 4, or never;
the Sleep row in Settings), the saber turns off its blade, screen and sound
and goes to sleep. Pick it up or press any button and it starts again on the
home screen, with nothing lost. A sleeping saber doesn't answer the SWD
probe, so wake it before flashing.

**Battles** are between family sabers. Open the Battle page (two crossed
sabers) and you're ready: a challenge from another saber is accepted on its
own. Push to challenge. Point the sabers at each other:

- the countdown names your opponent and counts 5-4-3-2-1, with a click and a
  blade flash on each number;
- the fight shows FIGHT!, then the seconds left and your swings so far;
- at the end the two sabers swap results on their own and show VICTORY,
  DEFEAT or DRAW with both scores. Keep pointing until they do.

A saber still on the conference firmware can't duel a family saber: the
family saber waits for a start signal the conference firmware never sends.

**Going back:** the conference images can't read the saved data this
build writes, and start the badge over without it. Run `bt5-badge backup`
before you flash `saber-play-fw`; to go back, flash the other image and
`bt5-badge restore` that file. Anything since the backup stays behind.

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

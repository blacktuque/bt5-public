<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="noda-dark.svg">
    <img src="noda.svg" alt="NODA" height="140">
  </picture>
</p>

<p align="center"><sub><code>BLACKTUQUE 5 · CONFERENCE FIRMWARE · SABER + REMOTE</code></sub></p>

# BT5 Public Firmware

The con is over, so here is everything: the firmware that runs the
Blacktuque 5 lightsaber badge and its IR remotes, the organizers' admin and
kiosk builds, the tool they used to edit badges, and the leaderboard. Flash it,
dump it, pull it apart, run your own scoreboard, or just unlock everything on
your saber.

> **Spoilers ahead.** `bt5-badge` can complete every challenge for you. If you
> would rather work them out, reverse the firmware first and reach for the
> tool afterwards.

---

## What's here

| File | Runs on | What it is |
|---|---|---|
| `saber-fw/saber-fw` | Saber badge (RP2040) | The badge firmware, as a stripped ARM ELF. Blade, sound, IMU, IR, battles, challenges. |
| `saber-fw/saber-kiosk-fw` | Saber badge (RP2040) | The **kiosk** firmware for the [scoreboard](scoreboard/README.md): receives and checks progress reports instead of playing. It has no key built in; you load your own with `bt5-badge kiosk-key`. |
| `saber-fw/saber-admin-fw` | Saber badge (RP2040) | The **organizer** firmware: `saber-social-fw` plus the full Organizer menu, booting straight past calibration. See [Admin firmware](#admin-firmware). |
| `saber-fw/saber-social-fw` | Saber badge (RP2040) | `saber-fw` plus a **Social** tile that sends the light-show patterns from any badge, and a RAINBOW blade colour. |
| `scoreboard/` | Your computer (Linux) + a kiosk saber | **kiosk-host**, the con's leaderboard: reads reports off a kiosk saber and serves the standings page. See [scoreboard/README.md](scoreboard/README.md). |
| `tools/` | Your computer (Linux) | **bt5-badge**, the organizers' badge tooling: reads, backs up and edits everything the badge saves, and flashes firmware. See [Editing your badge](#editing-your-badge). |
| `droid-fw/remote_fw.bin` | Remote (CH32V003) | The **bench remote**. Five keys, five badge buttons, used from across the room. |
| `droid-fw/remote_fw_lightshow.bin` | Remote (CH32V003) | The **light-show remote**. Every key fires a blade animation on each badge in range. |
| `droid-fw/remote_fw_jeopardy.bin` | Remote (CH32V003) | The **Jeopardy remote** used to run Hacker Jeopardy: right answer, countdown, time's up and buzz-in on every blade in range, plus room-wide silence and max volume. |
| `droid-fw/droid_allegiance_*.bin` | Remote (CH32V003) | 12 **allegiance droids**, ids `D001`–`D00C`. Ignite your blade near one and it asks you to report for the Republic or the Empire. See [Droids](#droids). |
| `droid-fw/droid_challenge_*.bin` | Remote (CH32V003) | 8 **Droid Finder droids**, ids `9001`–`9008`, to hide for the Droid Finder challenge. See [Droids](#droids). |
| `flash-jig/` | 3D printer | `JIG_TOP.stl` and `JIG_BOTTOM.stl`, the jig the organizers used to flash badges, one saber at a time. |
| `bom/` | — | `lightsaber_participant_bom.csv`, the badge's bill of materials, with LCSC part numbers. |
| `noda.svg`, `noda-dark.svg` | — | NODA, for this page. |
| `LICENSE`, `THIRD-PARTY-NOTICES.md` | — | The terms for everything here, and the notices for the open-source software in the binaries. See [Licence](#licence). |

---

## The remote

The remote has five keys, laid out as a D-pad. Each key sends one
Panasonic/Kaseikyo IR code. On the **bench remote**, the badge maps that code
onto one of its **own** buttons, so the remote is a copy of the badge's front
panel that works across the room. Every screen and every menu, not just
ignition.

| Key | D-pad | Badge button |
|---|---|---|
| `KEY1` | UP | Up: navigate up |
| `KEY2` | RIGHT | Battle: duel and broadcast menu |
| `KEY3` | DOWN | Down: navigate down |
| `KEY4` | LEFT | Challenges: challenge menu |
| `KEY5` | MIDDLE | Push: ignite or retract, confirm |

**Things to know**

- A code goes out when the key comes **up**, not when you press it down.
- The frames name **buttons**, not actions. MIDDLE ignites the blade from the
  home screen, but in a menu it is just a push, and it does whatever that
  screen does with one.
- Some keys do something different if you hold them. How long, and what
  happens, is for you to find out.
- Any badge in range will act on these codes, and so will any other remote
  that sends the same ones.

---

## Flashing the saber

**SWD is the only way in.** The badge has no USB and no BOOTSEL button, so
there is no UF2 or drag-and-drop route. You need a CMSIS-DAP probe: a
Raspberry Pi Debug Probe, or any RP2040 board (a Pico or an RP2040-Zero)
running Raspberry Pi's
[`debugprobe_on_pico`](https://github.com/raspberrypi/debugprobe/releases)
firmware.

| Probe (`debugprobe_on_pico`) | Badge J1 | Signal |
|---|---|---|
| GP2 | pin 1 (square pad) | SWCLK |
| GP3 | pin 2 | SWDIO |
| GND | pin 3 | GND |

If you'd rather use the test points, `TP4` is SWCLK and `TP3` is SWDIO.
**J2 is power, not SWD.** Don't wire the probe to it.

```sh
cargo install probe-rs-tools        # once
probe-rs list                       # the probe should show up
probe-rs download --chip RP2040 --speed 1000 --binary-format elf saber-fw/saber-fw
```

Use the shortest jumpers you have and always connect ground. If `probe-rs`
can't attach to a badge that's running, power-cycle it and run the command
within a second or two. J1 has no reset pin.

`tools/bt5-badge-x86_64-linux flash saber-fw/saber-fw` does the same job without
installing anything (see below).

---

## Editing your badge

[`tools/`](tools/) contains **bt5-badge**, the badge tooling the organizers
used, packaged for players. With the probe wired up as above, it reads and
backs up everything the badge saves and can change any of it: handle,
challenges, allegiance, talks, duel record, and the badge's id, role and
secret. Every change is signed with your badge's own secret, so the badge
accepts it. It needs nothing installed:

```sh
cd tools
chmod +x bt5-badge-x86_64-linux          # bt5-badge-aarch64-linux on a 64-bit Pi
./bt5-badge-x86_64-linux backup          # do this first
./bt5-badge-x86_64-linux show
```

Setup and every command are covered in [tools/README.md](tools/README.md).

---

## Admin firmware

`saber-admin-fw` is what the organizers ran. It's `saber-social-fw` with the
**Organizer** tile always on, whatever the badge's role, and it boots straight
to the home screen even on a badge that hasn't been calibrated.

```sh
cd tools
./bt5-badge-x86_64-linux backup
./bt5-badge-x86_64-linux flash ../saber-fw/saber-admin-fw
```

The Organizer menu's **broadcasts** (light shows, songs, the opening
ceremony and the rest) reach every badge in range and work straight away.

The four **targeted** rows, GRANT, TALK, NEURALYZE and SET ROLE, each send to
one badge by its 4-digit id. They're signed with that badge's own key, worked
out from a **fleet key**. The admin firmware has no key built in, so these
rows send nothing (and never show SENT) until you give it one. Make a key,
load it onto the admin saber, then key each badge you want to control from it:

```sh
./bt5-badge-x86_64-linux kiosk-key generate fleet.key    # keep this file
./bt5-badge-x86_64-linux kiosk-key set fleet.key         # on the admin saber
./bt5-badge-x86_64-linux identity --fleet-key fleet.key  # on each target saber
```

The same `fleet.key` works for a [scoreboard](scoreboard/README.md) kiosk, so
one key can run both. Keying a badge changes its secret; `bt5-badge restore`
puts the original back from its backup.

---

## Flashing the remote

The remote is a WCH CH32V003, flashed over its one-wire SWIO pin with a
**WCH-LinkE** programmer and [`rvprog`](https://pypi.org/project/rvprog/).

```sh
pip install rvprog                  # once
rvprog -f droid-fw/remote_fw.bin             # bench remote
rvprog -f droid-fw/remote_fw_lightshow.bin   # ... or the light-show remote
rvprog -f droid-fw/remote_fw_jeopardy.bin    # ... or the Jeopardy remote
```

On the Jeopardy remote, a **tap** runs the round: UP silences every badge in
range, RIGHT plays the right answer, DOWN starts a 17-second countdown, LEFT
is time's up, and MIDDLE is buzz-in with a 20-second answer clock. A
**2-second hold** plays a decorative pattern instead. Holding UP turns every
badge up to max.

On Linux, give your user access to the programmer first:

```sh
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="1a86", ATTR{idProduct}=="8010", MODE="666"' | sudo tee /etc/udev/rules.d/99-WCH-LinkE.rules
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="1a86", ATTR{idProduct}=="8012", MODE="666"' | sudo tee -a /etc/udev/rules.d/99-WCH-LinkE.rules
```

**Things to know**

- Take the coin cell out and power the remote **only** from the programmer's
  3V3 pin.
- SWIO shares a pin with `KEY3` (DOWN). Don't hold DOWN while flashing, or
  you short the debug line to ground.
- `Unsupported chip (ID: 0xffff)` means the link didn't come up. Check the
  wiring, then try again.
- Most of these images (the droids, and the bench and light-show remotes)
  sleep when idle, and a sleeping chip doesn't answer the programmer. Each
  stays awake for about 3 seconds after power-up, so to reflash one, power-cycle
  it (unplug and replug the programmer) and start `rvprog` straight away.

### Droids

A droid is the same CH32V003 board running beacon firmware: it reads no keys
and announces its id over IR every few seconds, and badges react. Each file is
named `droid_<kind>_<id>_<interval>_<cooldown>.bin`: the id, how many seconds
between announces, and the per-badge report cooldown in minutes.

- **Allegiance droids** (`droid_allegiance_*`, ids `D001`–`D00C`, 10–21 s).
  Ignite your blade within range and the badge asks you to report to the
  Republic or the Empire, which leans your allegiance. Each badge can report
  to the same droid again after the 30-minute cooldown.
- **Droid Finder droids** (`droid_challenge_*`, ids `9001`–`9008`, 2–6 s).
  Hide them for the Droid Finder challenge: badges light their status LED
  while one is in range, and the sweep logs each new one found. The longer the
  interval, the harder it is to find. The cooldown does nothing on these.

Every droid needs a unique id, so flash each file into one board only.

```sh
rvprog -f droid-fw/droid_challenge_9001_2_30.bin
```

---

## Hints

- The saber images are stripped, but they're still ELFs. It loads at its real
  addresses in Ghidra or any other disassembler, with no setup needed.
- The remote images are raw binaries for a RISC-V core (RV32EC), based at
  `0x00000000`.
- The badge doesn't know the remote from its own buttons, and it doesn't know
  this remote from any other IR transmitter.
- An IR receiver and a logic analyzer will show you a lot.

---

## Licence

Everything here is yours to use, study, modify and share, **but not to sell**.
It is licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/):
give credit, no commercial use, and anything you make from it goes out under
the same terms. That covers selling badges, kits or anything built from
these files.

The remote images are the exception. They're derived from Stefan Wagner's
[TinyRemote](https://github.com/wagiminator/CH32V003-IR-Remote), so they keep
its CC BY-SA 3.0 licence. Open-source parts inside the binaries keep their own
licences too. Details are in [LICENSE](LICENSE) and
[THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md).

<p align="center"><sub><code>MAY THE SOURCE BE WITH YOU</code></sub></p>

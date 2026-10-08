<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="noda-dark.svg">
    <img src="noda.svg" alt="NODA" height="140">
  </picture>
</p>

<p align="center"><sub><code>BLACKTUQUE 5 · CONFERENCE FIRMWARE · SABER + REMOTE</code></sub></p>

# BT5 Public Firmware

The firmware that runs the Blacktuque 5 lightsaber badge and its five-key IR
remote, released for players. Flash it, dump it, pull it apart. Everything the
saber and the remote do is in these images, and some of the challenges are a
lot easier once you can see how they work.

> **Unofficial help, not a walkthrough.** Nothing here hands you a flag. It
> gives you the same hardware the organizers used, so you can test ideas on
> your own bench instead of guessing.

---

## What's here

| File | Runs on | What it is |
|---|---|---|
| `saber-fw` | Saber badge (RP2040) | The badge firmware, as a stripped ARM ELF. Blade, sound, IMU, IR, battles, challenges. |
| `remote_fw.bin` | Remote (CH32V003) | The **bench remote**. Five keys, five badge buttons, used from across the room. |
| `remote_fw_lightshow.bin` | Remote (CH32V003) | The **light-show remote**. Every key fires a blade animation on each badge in range. |
| `noda.svg`, `noda-dark.svg` | — | NODA, for this page. |

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
probe-rs download --chip RP2040 --speed 1000 --binary-format elf saber-fw
```

Use the shortest jumpers you have and always connect ground. If `probe-rs`
can't attach to a badge that's running, power-cycle it and run the command
within a second or two. J1 has no reset pin.

---

## Flashing the remote

The remote is a WCH CH32V003, flashed over its one-wire SWIO pin with a
**WCH-LinkE** programmer and [`rvprog`](https://pypi.org/project/rvprog/).

```sh
pip install rvprog                  # once
rvprog -f remote_fw.bin             # bench remote
rvprog -f remote_fw_lightshow.bin   # ... or the light-show remote
```

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

---

## Hints

- `saber-fw` is stripped, but it is still an ELF. It loads at its real
  addresses in Ghidra or any other disassembler, with no setup needed.
- The remote images are raw binaries for a RISC-V core (RV32EC), based at
  `0x00000000`.
- The badge doesn't know the remote from its own buttons, and it doesn't know
  this remote from any other IR transmitter.
- An IR receiver and a logic analyzer will show you a lot.

<p align="center"><sub><code>MAY THE SOURCE BE WITH YOU</code></sub></p>

# bt5-badge

`bt5-badge` is the bench tooling the organizers used on the badges, packaged
for players. With a debug probe on the badge you can see everything the badge
has saved and change any of it: handle, challenges, allegiance, talks, duel
record, and the badge's id, role and secret. It also flashes firmware.

Every change is signed with **your badge's own secret**, which the tool reads
off the badge. The badge accepts the result as genuine.

## Wiring

You need a CMSIS-DAP probe: a Raspberry Pi Debug Probe, or any RP2040 board
(a Pico or an RP2040-Zero) running Raspberry Pi's
[`debugprobe_on_pico`](https://github.com/raspberrypi/debugprobe/releases)
firmware.

| Probe (`debugprobe_on_pico`) | Badge J1 | Signal |
|---|---|---|
| GP2 | pin 1 (square pad) | SWCLK |
| GP3 | pin 2 | SWDIO |
| GND | pin 3 | GND |

If you'd rather use the test points, `TP4` is SWCLK and `TP3` is SWDIO.
**J2 is power, not SWD.** Don't wire the probe to it. Use short jumpers and
always connect ground.

## Setup

Run these from this directory:

```sh
# Linux, once: let your user open the probe, then unplug and replug it
sudo cp 69-bt5-debugprobe.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && sudo udevadm trigger

chmod +x bt5-badge-x86_64-linux          # bt5-badge-aarch64-linux on a 64-bit Pi
./bt5-badge-x86_64-linux probes          # the probe should show up
./bt5-badge-x86_64-linux show            # everything the badge has saved
```

The binaries are self-contained. They need only `libudev.so.1`, which any
systemd distro already has. The examples below write `bt5-badge` for
whichever binary you're using.

**Back up first.** The identity page holds the badge's id and secret. Keep the
backup file; `restore` puts it back:

```sh
bt5-badge backup                         # writes bt5-badge-<id>-<time>.bin
```

## Commands

| Command | What it does |
|---|---|
| `show` | Prints everything: identity, calibration, allegiance, rank, settings, challenges (hidden ones too), droids, talks, duel record, and what a kiosk report would say. Add `--hex` or `--raw` for more detail. |
| `backup [file]` | Saves the identity and state pages to an 8 KiB file. |
| `restore <file>` | Writes a backup back to the badge. `--state-only` leaves the identity page alone. |
| `reset` | Erases the saved state, so the badge boots to **CALIBRATION REQUIRED**. |
| `handle V4DER1977` | Sets the handle: 2–11 characters, A–Z and 0–9. |
| `identity` | Shows the badge id, role and secret. |
| `identity --id 1234 --role speaker --secret random` | Sets any mix of the id (hex), the role and the secret (32 hex digits, `random`, or `dev`). Progress is kept: the state page is signed again with the new secret. A backup is saved first. |
| `identity --fleet-key fleet.key` | Derives the secret from a fleet key file (see `kiosk-key`), so a kiosk holding the same key accepts this badge's reports and an admin saber holding it can send it grants. Combine it with `--id` to choose the id the secret is derived for. |
| `role organizer` | Sets the role to `attendee`, `organizer` (shown as DEV; unhides the broadcast menu), `bouncer` or `speaker`. Same as `identity --role`. |
| `challenges` | Lists every challenge and whether it's done. |
| `challenges morse kyber` | Marks the named challenges complete. Names match loosely. |
| `challenges --all` | Completes every menu challenge, which unlocks the colour picker and ignite styles. `--every` adds the hidden ones. |
| `challenges --clear [names]` | Clears the named challenges, or all of them if none are named. |
| `allegiance -16` | Sets your lean from −16 (Empire) to +16 (Republic). The blade colour and rank follow it. |
| `talks 14 --rooms 2` | Sets the talks attended (0–512), spread over N tracks. |
| `record 17 9 --roles attendee,speaker` | Sets the duel wins, losses and roles beaten. |
| `clear-progress` | Clears challenges, droids, beacons, allegiance and talks. Calibration and settings are kept. |
| `rearm-self-test` | Makes the hardware self-test run again on the next boot. |
| `tamper` | Edits the saved state **without** signing it, the way a cheater without the secret would. The badge flags itself as tampered. `reset` clears it. |
| `flash ../saber-fw/saber-social-fw` | Flashes a firmware ELF, such as `../saber-fw/saber-fw`, `../saber-fw/saber-social-fw`, `../saber-fw/saber-admin-fw` or `../saber-fw/saber-kiosk-fw` from this repo's `saber-fw/` folder. The fleet-key, identity and state pages are never written, so progress survives. |
| `kiosk-key generate fleet.key` | Makes a new random fleet key file. No badge is needed. |
| `kiosk-key set fleet.key` | Loads a fleet key onto a kiosk or admin saber. The kiosk verifies reports with it, and the admin firmware signs its targeted commands with it. `kiosk-key show` and `kiosk-key clear` show it or remove it. |

Every command that writes accepts `--dry-run`, which shows the change without
making it. After a write, the tool resets the badge. `bt5-badge <command>
--help` explains each one.

## Things to know

- `--speed` sets the SWD clock in kHz. The default, 1000, is safe on jumper
  wires. On short wiring, `--speed 20000` flashes far faster.
- With more than one probe attached, the tool asks which one to use. To skip
  the question, pass `--probe VID:PID[:SERIAL]` or set `$PROBE`.
- If it can't attach to a running badge, power-cycle the badge and try again
  within a second or two.
- If the saved state fails its signature check (it's been tampered with), the
  tool won't sign it again unless you pass `--force`, because doing so clears
  the tamper mark.
- At the con, each badge's secret was derived from its id using a key only the
  organizers held. The kiosk and admin grants relied on that pairing, but the
  badge itself never checks it. A badge with a new id or secret works as
  normal; only the con's own kiosk and grants would have rejected it.
- Badge ids run from `0000` to `7FFF`. Higher ids make other badges treat
  yours as a beacon droid, talk droid, track remote, presenter or kiosk. That
  needs `--force`.
- `identity --id 0 --secret dev` turns the badge into an unprovisioned dev
  board.

# Scoreboard

This is the leaderboard from the con. Badges beam their progress over IR to a
**kiosk saber**. A computer reads the reports off that saber through a debug
probe and serves the standings as a web page, for a TV and for a terminal
where people look themselves up.

```
your saber ──IR──▶ kiosk saber ──debug probe──▶ kiosk-host ──network──▶ browser
```

| File | For |
|---|---|
| `kiosk-host-x86_64-unknown-linux-gnu.2.17.tar.gz` | An x86_64 Linux PC or laptop. |
| `kiosk-host-aarch64-unknown-linux-gnu.2.17.tar.gz` | A Raspberry Pi 4 or 5 running 64-bit Raspberry Pi OS. |

Each tarball holds the `kiosk-host` program and everything needed to install
it. The program, the web page and the logos are all in one binary, so the
host computer doesn't need Rust or probe-rs. It needs glibc 2.17 or newer and
`libudev.so.1`, which any systemd distro has.

## What you need

- **Two or more sabers.** One becomes the kiosk; the others are players that
  report to it.
- **A debug probe** wired to the kiosk saber's J1. The wiring is in
  [tools/README.md](../tools/README.md#wiring).
- **Steady power for the kiosk saber**, such as a bench supply. AA batteries
  won't last.
- **A Linux computer** to run `kiosk-host`, and a browser that can reach it.

## 1. Make a fleet key

The kiosk accepts a report only if it's signed with a key it can check. At
the con, every badge's secret was derived from one master key, and the kiosk
had that key built in. `saber-kiosk-fw` has **no key built in**. Instead you
make your own **fleet key**, load it onto the kiosk, and key each player
saber from it.

These examples use `bt5-badge` from [`tools/`](../tools/), written
`bt5-badge` for whichever binary you use:

```sh
cd tools
bt5-badge kiosk-key generate fleet.key     # 32 random bytes; keep this file
```

## 2. Flash the kiosk saber

With the probe on the saber that will be the kiosk:

```sh
bt5-badge backup                           # this saber's pages, first
bt5-badge flash ../saber-fw/saber-kiosk-fw
bt5-badge kiosk-key set fleet.key
```

When it boots, the screen shows the BLACKTUQUE 5 splash with **AIM SABER
HERE** along the bottom. A **solid red blade** means it has no fleet key
(run `kiosk-key set`). Leave the probe connected, because `kiosk-host` reads
the reports through it.

To make it an ordinary saber again, flash `../saber-fw/saber-fw`. Flashing doesn't
touch the saber's saved state or its fleet key.

## 3. Key the player sabers

Move the probe to each saber that should report, and derive its secret from
the same fleet key. Its progress is kept, and a backup is saved first:

```sh
bt5-badge identity --fleet-key fleet.key              # keep its id
bt5-badge identity --fleet-key fleet.key --id 0042    # or choose one
```

Give each saber its own id. The leaderboard lists sabers by id, so two
sabers with the same id would be counted as one.

## 4. Install kiosk-host

On the computer the probe is plugged into:

```sh
tar xzf kiosk-host-x86_64-unknown-linux-gnu.2.17.tar.gz   # or the aarch64 one
cd kiosk-host-x86_64-unknown-linux-gnu.2.17/
sudo cp 69-bt5-debugprobe.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && sudo udevadm trigger
```

Unplug the probe and plug it back in. Without the rule, only root can open
it.

## 5. Run it

```sh
./kiosk-host                          # with several probes: --probe 2e8a:000c
```

Open `http://localhost:8080/` for the leaderboard, or use the computer's
address from another machine; you may need `sudo ufw allow 8080/tcp`. Results
are saved to `leaderboard.json` in the current directory, and survive a
restart.

| Page | Shows |
|---|---|
| `/` | The leaderboard. |
| `/?rotate` | The TV view, cycling through the boards. |
| `/?kiosk` | The attendee terminal, where people look themselves up. |

## 6. Report from a saber

On a player saber, go to the home screen, press **Up** twice to reach the
Record tile, and press **Push** to open the **Service Record**. Aim the blade
tip at the kiosk saber and press **Push** again. Sending takes about eight
seconds. The kiosk chimes and flashes once the whole
report has arrived, and the player shows up on the page at its next refresh.

## Making it permanent

To have a dedicated machine, such as a Pi behind a TV, start the kiosk
full-screen on power-up:

```sh
sudo ./install.sh                     # then: sudo reboot
sudo ./install.sh --help              # page choice, headless, --uninstall
```

The installer needs internet access the first time, because it installs
Chromium and cage with `apt`. `INSTALL.txt` in the tarball also covers the
locked-down Ubuntu-laptop setup used at the con (`./install.sh --laptop`).

## Settings and the full guide

Everything else is in `userguide.md` inside the tarball: the pages and URLs,
scoring, the winner rule, command-line options, the data file and
troubleshooting. Run `./kiosk-host --help` for the options.

- `winners.toml` holds the rule for who counts as having won the badge. At
  the con it was kept secret.
- `scoring.toml` sets the leaderboard points.
- `talks.toml` holds the talk titles.

`kiosk-host` re-reads all three whenever they change, so it doesn't need a
restart.

## Things to know

- A saber whose secret didn't come from the kiosk's fleet key gets nothing on
  the kiosk: no chime, no flash, and it doesn't appear on the page. That
  includes a saber still on its con identity. Run `identity --fleet-key` on
  it. `bt5-badge restore` puts its original identity back afterwards.
- Swapping in a new fleet key means rekeying every player saber from it.
- `kiosk-host --no-probe` serves the saved `leaderboard.json` without a saber
  attached. Use it to try out the pages or to show final results.

# HackRF RX Bridge — Release Notes

## v1.1.3 — DC blocker default-off for RF-direct receivers (2026-05-06)

DC blocker is now default-OFF for RF-direct receivers (HackRF /
RTL-SDR / SDRplay / Pluto / AirSpy) and grayed out in Settings.
Their hardware DC correction at the SDR API level (and SDRplay's
Low-IF NCO chain in particular) handles the chip's residual offset
upstream; the v1.1.2 software IIR HP was redundant for these radios
and produced a small spike at QMAP centre on the SDRplay Low-IF
path (G3WDG bench report 2026-05-06).

Sound-card-IQ sources (FunCube Pro+ V2, FlexRadio DAX-IQ, Malachite
via iq-rx-bridge) keep the DC blocker default-ON: they have no
hardware DC mitigation, the LO leakage is real, and the IIR HP
is the only thing notching it out.

INI key linrad/dc_block_enabled is unchanged; existing INIs keep
their stored value. Only the first-launch default flips per device.

Drop-in upgrade from v1.1.2.

## v1.1.2 — DC blocker for zero-IF receivers (2026-05-05)

DC blocker for zero-IF receivers, removes the LO-leakage spike that
FunCube Pro+ V2 / HackRF / RTL-SDR / Pluto / AirSpy leak at the centre
of the spectrum. Per-sample IIR high-pass at the front of both the
on-screen waterfall (FftEngine) and the QMAP wire path (LinradServer);
cutoff = 100 Hz, well below any audio offset Q65 / FT8 cares about.

Toggle in Settings → "DC blocker (zero-IF spike removal)", default ON.
Toggling on also resets the I/Q balance EMA so a stale DC accumulator
from earlier samples doesn't keep subtracting against now-DC-free
input for ~2 s.

Drop-in upgrade from v1.1.1. INI key linrad/dc_block_enabled added
(default true; honours the previous behaviour for anyone who never
opens Settings).

## v1.1.1 — capability-gated IQ rate combo (2026-05-05)

Tightens the IQ-rate combo. Internally adds a per-device capability
list; only sound-card-IQ devices currently restrict their offered
rates -- RF-side bridges (HackRF / RTL-SDR / SDRplay / Pluto / AirSpy)
are unchanged in behaviour and still offer 96 / 128 / 192 / 256 kHz.

Drop-in upgrade from v1.1.0. No INI changes.

## v1.1.0 — UI refresh: fixed window, Settings menu, Linrad rate readout (2026-05-05)

User-visible polish across the bridge UI; no behavioural changes on
the wire (96 kHz IQ format unchanged).

- **Fixed-size 400x640 main window**. Replaces the freely-resizable
  640x540 minimum. Window opens identically every session and
  doesn't drift; conditional banners (manual-freq override,
  transverter IF readout) word-wrap rather than clip.
- **Settings is a top-level menu** in the menu bar (shortcut
  `Ctrl+,`) -- was a button at the bottom of the State group. Frees
  ~40 px of vertical real estate for the waterfall.
- **Linrad rate readout** in the State grid, between the device row
  and RX status. Reads the active LinradServer output rate; matches
  what's persisted in the INI.
- **Settings dialog reflow**: radio gain panel and bridge-wide
  group sit horizontally side-by-side. Was vertical, ran past the
  bottom of 1080p laptops with the deeper panels.
- **New "Linrad IQ rate" combo** in the Settings dialog. Defaults
  to "96 kHz (QMAP Default)" -- same wire format as before; matches
  every shipped QMAP release.
- UDP data port (`50004`) and Linrad host (`127.0.0.1`) /
  TCP port (`49812`) editors gained tooltips explaining their use
  in multi-instance setups.

INI compatible with v1.0.x. Drop-in upgrade.

## v1.0.1 — opt-in CAT server for WSJT-X Doppler tracking (2026-05-04)

Adds the same CAT server pattern that landed in `pluto-rx-bridge`,
`rtlsdr-rx-bridge`, and the rest of the family. With WSJT-X
**Rig = Hamlib NET rigctl** pointed at this bridge's port, Doppler
tracking commands corrected frequencies directly to the bridge —
HackRF tunes, QMAP centre freq follows.

- New rigctld-compatible TCP server on port **4537**. Default OFF
  so the bridge stays a passive UDP observer in the common case
  (WSJT-X driving a real radio).
- Toggle via Settings dialog ("CAT server" checkbox + port) or
  `--cat` / `--cat-port <n>` on the CLI. Restart bridge to take
  effect.
- Auto-detect UDP mute: when a CAT client is actually connected,
  WSJT-X UDP Status freq updates are silenced. When no CAT client
  is connected, the bridge falls back to UDP cleanly.
- Live source indicator in the window title:
  `HackRF RX Bridge v1.0.1 — UDP` /
  `— UDP (CAT idle)` / `— CAT (n)`.
- Picks up the bridge-core CatServer data-mode-PTT fixes
  (`ptt_type=0x1`, `has_set_ptt=1` etc.) so WSJT-X PKTUSB / PKTLSB
  Test PTT works.

INI compatible with v1.0.0. Drop-in upgrade.

## v1.0.0 — stable (2026-05-02)

Promoted out of beta. RX-only HackRF observer alongside a real radio
for TX has been verified end-to-end on 2 m. The 0.99.x line ends
here; future development opens a 1.x series.

Cumulative since v0.99.4:

- **Waterfall span now matches the actual IQ rate** (bridge-core
  fix). Display labels follow the real sample rate instead of the
  hardcoded 2 MHz default.

No INI / migration changes; v1.0.0 is a drop-in upgrade from v0.99.4.

## v0.99.4 — multi-instance support (multi-band ops) (2026-05-02)

Run two bridges side-by-side — different WSJT-X / QMAP instances,
no shared state.

- New `--instance <name>` CLI flag namespaces the INI file, window
  title, and taskbar entry. `hackrf-rx-bridge.exe --instance 70cm`
  reads/writes `HackRF RX Bridge - 70cm.ini`.
- New **Settings → "Linrad TCP port"** + **"Linrad UDP port"**
  spinboxes (defaults 49812 / 50004). Increment per bridge instance
  for multi-QMAP setups. CLI: `--linrad-tcp-port`,
  `--linrad-udp-port`. Take effect on next launch.

See RTL-SDR v0.99.8 notes for a full multi-instance walkthrough.
HackRF-side device-serial picker (for two HackRFs on one machine)
lands in a future version bump.

Drop-in upgrade from v0.99.3.

## v0.99.3 — spectrum waterfall toggle (2026-05-02)

The built-in spectrum / waterfall display can now be turned off from
the **View menu** (or the **Ctrl+W** shortcut). Useful when you don't
need the visual debugging and would rather not pay the CPU cost.

- View → "Show spectrum waterfall" — checkable, default on.
- New CLI flag `--no-waterfall` launches with the display off and
  persists the choice to INI key `gui/waterfall_enabled`.
- When off, three layers of work are skipped: per-IQ-buffer
  `FftEngine::pushIq()` (the per-sample int8→float + Hann window
  multiply), the widget paint events, and the 20 Hz row-poll
  timer. Roughly 2–5 % of one CPU core saved at 2 Msps.
- Default ON for first installs and for upgrades — no surprise
  change for existing testers.

Drop-in upgrade from v0.99.2; no INI migration.

## v0.99.2 — beta (2026-04-30)

Feature-parity release with the SDRplay sibling, plus shared GUI code
via Phase 1b refactor.

- **Transverter offset** for IF-transverter setups. New Settings →
  "Transverter offset" field (signed MHz). The SDR is tuned to
  *(WSJT-X dial + offset)* while the GUI, WSJT-X, QMAP, and the
  LinradServer header all keep showing the operating dial.
  Example: dial 10368 MHz, offset −10224 MHz, HackRF actually tunes
  to 144 MHz. CLI: `--transverter-offset <MHz>`. INI key:
  `radio/transverter_offset_hz`.
- **Manual SDR frequency override.** Settings checkbox + freq field;
  decouples the bridge from the WSJT-X dial. Useful for QMAP-priority
  observation when activity spans more than 90 kHz around the dial.
  WSJT-X narrowband decode only works when WSJT-X dial = manual freq.
  Bold orange banner under the dial display when active. CLI:
  `--manual-freq <MHz>`. INI keys: `radio/manual_freq_override`,
  `radio/manual_freq_hz`.
- **Periodic streaming-stats log line.** Every 5 seconds the bridge
  writes one diagnostic line: `[Stats] RX <N> samples in 5s …,
  peak |IQ|=<X> (<Y> dBFS), last freq update <ms> ms ago` so a tester
  can answer "is the SDR streaming?" and "is WSJT-X feeding freq
  updates?" from one screenshot.
- **Frequency display now sourced from the bridge's actual operating
  freq** rather than WSJT-X UDP. Populates correctly at startup
  (from the last persisted INI value or manual-override key) before
  WSJT-X has broadcast a single status message, and respects the
  manual-override mode.
- **High-contrast IF readout** under the dial display when transverter
  offset is non-zero — shows the actual SDR tune freq in bold red.
- **Phase 1b refactor**: `RxMainWindow` and `RxSettingsDialog` now
  live in `bridge-core/` and are shared with the RTL-SDR and SDRplay
  sibling apps. Future GUI features land once and propagate to all
  three.

## v0.99.1 — beta (2026-04-28)

- **Settings dialog** added. Same shape as the WSJT-X bridge's
  Settings dialog, minus TX VGA / Mode picker / TX audio device
  picker (none of which apply to an RX-only app). Covers RX LNA /
  RX VGA / RX AMP, bias tee toggle, IF offset, PPM correction,
  RX audio device picker, software RX audio gain (Windows WASAPI
  compensation), Linrad output gain, I/Q balance correction
  toggle with live α / φ / native rejection diagnostics.
- Settings → I/Q balance toggle and Linrad gain apply live; HackRF
  gain changes are pushed to the chip via libhackrf without
  interrupting the stream. RX audio device changes restart playback
  on the new device immediately.

## v0.99.0 — first beta (2026-04-28)

First release of the **HackRF RX Bridge** — a Windows-only companion app
that lets a HackRF One add wideband Q65 (QMAP) reception alongside an
existing real radio (IC-905, IC-705, FT-991A, etc.) without disrupting
the real radio's TX setup.

### Use case

Your IC-905 (or any rig with a normal CAT/audio interface) handles TX
and narrowband RX as it always has, controlled by WSJT-X via Hamlib. A
HackRF — fed from a splitter on the same antenna, or a separate RX-only
antenna — runs alongside the real rig as a **wideband observer**. This
bridge:

- Listens to **WSJT-X UDP messages** (port 2237 by default) for the
  current dial frequency, mode, and transmit state
- Tunes the HackRF to match
- Demodulates SSB to **VB-Audio Virtual Cable Line 1** so WSJT-X's
  "Sound input (RX)" sees the HackRF audio
- Streams **96 kHz IQ to QMAP** (UDP 50004) for wideband Q65 decode
- Stops HackRF RX during WSJT-X TX so the local-TX bleed-through can't
  slam the LNA, and (optionally) drives the HackRF bias tee on/off
  to match WSJT-X's TX state for transverter sequencer power

WSJT-X CAT continues to control the real radio. WSJT-X's "Sound
output (TX)" still goes to the rig's USB audio interface as before.
Only the **RX audio path** is replaced with bridge-fed audio from the
HackRF.

### Installation note (read first)

The installer is **not code-signed** and is **64-bit only**
(Windows 10 / 11 x64). On first launch on a fresh Windows machine you
will see Microsoft Defender SmartScreen warn:

> Windows protected your PC.
> Microsoft Defender SmartScreen prevented an unrecognized app from
> starting.

Click **More info → Run anyway**. You should only see this once per
binary. The same warning may appear once on the installed
`hackrf-rx-bridge.exe`; handle it the same way.

### What you'll need to install separately

- **HackRF One USB driver (WinUSB)** — the installer offers to launch
  Zadig at the end to set this up automatically. Skip if `hackrf_info`
  on your machine already prints firmware/serial without an error.
- **VB-Audio Virtual Cable** — <https://vb-audio.com/Cable/>. Provides
  the `Line 1` virtual sound device the bridge feeds.
- **WSJT-X 2.7+** — for FT8 / FT4 / Q65 narrowband decoding.
- **QMAP 0.6+** — for wideband Q65. Set Network input = enabled, UDP
  port = 50004.

### WSJT-X configuration

| Setting | Value |
|---|---|
| Radio | your real rig, via Hamlib (Settings → Radio → choose your rig and CAT method — *not* a `Hamlib NET rigctl` pointing at this bridge) |
| PTT method | CAT |
| Sound output (TX) | the real rig's USB audio interface |
| Sound input (RX) | `Line 1 (Virtual Audio Cable)` |
| Settings → Reporting → "Accept UDP requests" | **enabled**, port `2237` |

That last item is required — without it WSJT-X doesn't broadcast its
status messages and the bridge has no way to know the dial frequency.

Launch order: **real rig → WSJT-X → HackRF RX Bridge → QMAP**.

### Features

- **WSJT-X UDP listener** (port 2237) for dial freq, mode, and TX
  state. Same protocol GridTracker / JTAlert / Log4OM use; works
  cross-machine.
- **HackRF RX path** at 2 Msps with platform-aware IF offset (0 on
  Windows by default).
- **SSB demod** via Hilbert phasing (~35 dB opposite-sideband
  rejection) to VB-Cable Line 1 for WSJT-X.
- **96 kHz IQ wideband stream** to QMAP via UDP 50004 (Linrad
  protocol), with the same `(-1)^n·conj()` transform and adaptive
  I/Q balance correction as the WSJT-X bridge — image rejection
  past −40 dBc on broadband signals.
- **RX-blanking on WSJT-X TX**: HackRF RX is stopped while WSJT-X
  is keying the real rig, so the local-TX bleed-through doesn't
  hammer the LNA.
- **Bias-tee follows WSJT-X TX**: if you have the HackRF bias tee
  enabled (manual toggle), the 3.3 V on the RF port follows
  WSJT-X's transmit state for transverter sequencer / LNA power.
- **GUI status panel**: dial freq from WSJT-X, mode, WSJT-X UDP-link
  state, HackRF status, RX peak meter, waterfall.
- **Settings** in this 0.99.0 release are **INI / CLI only** —
  edit `%APPDATA%\Roaming\n6nu\HackRF RX Bridge.ini` or pass flags
  on the command line. A full Settings dialog GUI ships in 0.99.1.

### Command-line options

- `--rx-lna <gain>` — RX LNA, 0–40 dB step 8
- `--rx-vga <gain>` — RX VGA, 0–62 dB step 2
- `--rx-amp` — enable HackRF AMP (+14 dB wideband, worsens IMD)
- `--bias-tee` — enable HackRF bias tee
- `--if-offset <Hz>` — IF offset
- `--ppm <ppm>` — frequency correction
- `--rx-device <name>` — audio output device (default Line 1)
- `--linrad-gain <dB>` — Linrad/QMAP digital output gain (default 20)
- `--wsjtx-port <port>` — WSJT-X UDP listener port (default 2237)
- `--no-gui` — headless mode
- `--console` — open a debug console with full stderr log

`--help` shows the full set.

### Reporting

Send observations / decodes / bug reports to
**<n6nu@arrl.net>**. Useful info to include:

- HackRF serial / firmware version (`hackrf_info`)
- Windows version
- WSJT-X version + which real rig you're using
- The bridge log: relaunch with `hackrf-rx-bridge.exe --console`,
  reproduce the issue, copy/paste the console output
- For QMAP issues, also `qmap.ini` and a wideband-waterfall
  screenshot

### License

Copyright (C) 2026 **Andreas Junge, N6NU** &lt;<n6nu@arrl.net>&gt;.
Licensed under the **GNU General Public License v3 or later** —
see [`LICENSE`](LICENSE). Bundled third-party components are
documented in [`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md).
**No warranty.** Install and run at your own risk.

# HackRF RX Bridge — Beta Releases

Windows installer downloads for the **HackRF RX Bridge** — a Qt6
C++ application that adds wideband Q65 (QMAP) reception to a station
already running a real radio (IC-905, IC-705, FT-991A, etc.) for TX.

The bridge listens to **WSJT-X UDP** for the dial frequency, tunes
the HackRF to match, demodulates SSB to **VB-Cable Line 1** for
WSJT-X RX audio, and streams **96 kHz IQ to QMAP** for wideband Q65
decode. Your real rig keeps doing TX (and narrowband RX, if you
prefer). No interference with your existing CAT / audio setup.

Author: **Andreas Junge, N6NU** &lt;<n6nu@arrl.net>&gt;.

---

## Latest beta — v0.99.1

Download: **[hackrf-rx-bridge-0.99.1-setup.exe](hackrf-rx-bridge-0.99.1-setup.exe)**

Full per-version notes, system requirements and known limitations
are in [`RELEASE_NOTES.md`](RELEASE_NOTES.md).

### First-launch SmartScreen warning

The installer is **not code-signed** and is **64-bit only**
(Windows 10 / 11 x64). On first launch you will see:

> Windows protected your PC.
> Microsoft Defender SmartScreen prevented an unrecognized app from
> starting.

Click **More info → Run anyway**. You should only see this once
per binary. The same warning may appear once on the installed
`hackrf-rx-bridge.exe`; handle it the same way.

### What you'll need

- **Real radio + WSJT-X** — the rig is whatever you already have
  (IC-905, IC-705, FT-991A, etc.). WSJT-X drives it via Hamlib as
  always; this bridge does NOT replace that.
- **HackRF One USB driver (WinUSB)** — the installer offers to
  launch Zadig on completion to set this up. Skip if `hackrf_info`
  on your machine already prints firmware/serial without an error.
- **VB-Audio Virtual Cable** — <https://vb-audio.com/Cable/>.
  Provides the `Line 1` virtual sound device the bridge feeds.
- **WSJT-X 2.7+** with the UDP server enabled —
  Settings → Reporting → "Accept UDP requests" → port `2237`.
  Without this WSJT-X doesn't broadcast its dial freq and the
  bridge has nothing to track.
- **QMAP 0.6+** — Network input enabled, UDP port `50004`.

### WSJT-X audio routing for this bridge

| Setting | Value |
|---|---|
| Radio | your real rig, via Hamlib |
| PTT method | CAT |
| Sound output (TX) | the real rig's USB audio interface |
| **Sound input (RX)** | **`Line 1 (Virtual Audio Cable)`** ← fed by this bridge |
| Settings → Reporting → Accept UDP requests | **enabled**, port 2237 |

Launch order: **real rig → WSJT-X → HackRF RX Bridge → QMAP**.

### First-time configuration in the bridge

After install, launch the bridge, click **Settings…**, and:

1. Set **RX audio output** to `Line 1 (Virtual Audio Cable)` —
   the default is the system audio device, not Line 1, so you
   need to pick it explicitly the first time.
2. Set **RX LNA / RX VGA / AMP** for your operating conditions.
   Conservative defaults: LNA 16, VGA 16, AMP off.
3. Click **Apply**. Settings persist to
   `%APPDATA%\Roaming\n6nu\HackRF RX Bridge.ini` so subsequent
   launches come up the same way.

---

## Reporting

Send observations / decodes / bug reports directly to
**<n6nu@arrl.net>**. Useful information to include:

- HackRF serial / firmware version (`hackrf_info`)
- Windows version
- WSJT-X version + which real rig you're using
- The bridge log: relaunch with `hackrf-rx-bridge.exe --console`,
  reproduce, copy/paste the console output
- For QMAP issues, also `qmap.ini` and a wideband-waterfall
  screenshot

---

## License

Copyright (C) 2026 Andreas Junge, N6NU &lt;<n6nu@arrl.net>&gt;.
Licensed under the **GNU General Public License version 3 or later**;
see [`LICENSE`](LICENSE).

This program is distributed in the hope that it will be useful, but
**WITHOUT ANY WARRANTY**; without even the implied warranty of
**MERCHANTABILITY** or **FITNESS FOR A PARTICULAR PURPOSE**. Use it
at your own risk.

Bundled third-party components — including libhackrf (GPLv2),
FFTW3 (GPLv2+), Qt 6 (LGPLv3), SoXR (LGPLv2.1), libusb (LGPLv2.1+),
FFmpeg shared libraries (LGPLv2.1+), and Zadig (GPLv3, by Pete
Batard / libwdi) — are documented in
[`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md). Source code
for the bridge itself is available on request from N6NU under the
GPLv3 "written offer" provision (§6) at the email address above; a
public source-code repository will be linked here once the project
leaves beta.

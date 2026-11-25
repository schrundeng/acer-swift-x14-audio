## Custom DSP Speaker Tuning for Acer Swift X14 2026 (SFX14-73G)

My own custom internal bottom firing speaker correction and psychoacoustic dynamics stack measured and tuned for this very laptop.

This DSP aims to fix the default factory tuning that sounds honky, tune tonal balance to my taste, add loudness back, and synthesize low-end illusion.

> **Note: This DSP stack substantially reduces overall volume compared to stock tuning!!**

I made this myself because I cannot stand the terrible tuning that manufacturers do on their Windows laptops. Please keep in mind that there are maybe some mistakes with how I do the processing stack and my measurement is nowhere near accurate enough.

---

## Folder Structure

- `equalizer-apo/` : Windows system-wide DSP chain
- `easyeffects/`: Linux PipeWire / EasyEffects presets (to be done)

---

## Windows DSP (Equalizer APO)

### Prerequisites

1. [Equalizer APO](https://sourceforge.net/projects/equalizerapo/) (tested on 1.4.2)
2. [Peace Equalizer](https://sourceforge.net/projects/peace-equalizer-apo-extension/) _(optional, if importing `Internal Speaker.peace` directly)_
3. [Cockos ReaPlugs v2.36](https://www.reaper.fm/reaplugs/) (`reajs.dll`, `reacomp-standalone.dll`, `reaxcomp-standalone.dll`)
4. [LoudMax v1.47](https://loudmax.blogspot.com/) (`LoudMax64.dll`)

### Installation & Setup

1. Install **Equalizer APO**, **ReaPlugs** (make sure the JS effects folder stays alongside `reajs.dll`), and extract `LoudMax64.dll`.

2. Open Equalizer APO's `DeviceSelector.exe` (or Configurator) and verify APO is installed on your **Speakers (Realtek(R) Audio)** device.

3. Copy `equalizer-apo/config.txt` and `equalizer-apo/swift_internal.txt` into `C:\Program Files\EqualizerAPO\config\`.
   - _(Optional)_ If you use Peace GUI instead, import `equalizer-apo/Internal Speaker.peace` directly, note that Peace overwrites `peace.txt`, not `swift_internal.txt`, and does not load the external VST chain in `config.txt` on its own.

4. Open `C:\Program Files\EqualizerAPO\config\config.txt` (or `Editor.exe`) and update:
   - The `Device:` line to match your own Realtek speaker endpoint GUID.
   - The `C:\VST2 EqAPO\...` DLL paths if your ReaPlugs and `LoudMax64.dll` live in a different directory.

---

### Signal Chain Overview

1. **sstillwell/hugebooty** (`reajs.dll`)
   - Mix: 60%, Drive: 100%, Freq: 120 Hz
   - Sits before the high-pass EQ so it reads the raw sub-120hz fundamental frequencies, applies quadratic waveshaping to give illusion of "deeper bass" on small speakers, and runs a 12dB/oct high-pass at 120hz on its wet branch so ONLY warm upper-bass harmonics (>120Hz) are added without mud.

2. **Parametric EQ** (`swift_internal.txt`)
   - The speaker drivers exhibit A LOT of chassis resonance around 650 Hz and 1.15–1.18 kHz (the honk cuppy sound on most laptops, including this!!), harsh upper-mid shoutiness at 2.85–4.0 kHz, high frequency sizzle at 6.8–14.5 kHz, and natural steep low-end roll-off below ~150 Hz.
   - To fix that: 110 Hz HPQ (Q 0.71) to protect the small drivers from useless excursion, massive cuts at 650 Hz (-20.8 dB) and 1150 Hz (-21.9 dB) plus 166/320 Hz (-5.4/-11.3 dB) to kill the cuppy honk, and notches across 2.85–14.5 kHz + a 9 kHz high shelf (-7.1 dB) to tame shoutiness and sizzle.

3. **loser/stereoEnhancer** (`reajs.dll`)
   - Width Low: 100%, Crossover: 250 Hz, Width High: 160%
   - Keeps everything below 250 Hz untouched while widening mids and highs above 250 Hz to open up bottom-firing desk/surface reflections. Basically to give wider soundstage than the speaker physically sits

4. **ReaComp** (`reacomp-standalone.dll`)
   - 250 Hz sidechain HPF, -18.0 dB threshold, 3.0:1 ratio, 20 ms attack, 500 ms release, 6 dB knee, +3.0 dB makeup. Sidechain HPF prevents kicks from pumping vocals.
   - Functioning as glue compressor for consistent volume

5. **ReaXcomp** (`reaxcomp-standalone.dll`)
   - 3-band downward dynamics (-18.0 dB threshold across all bands, +6.0 dB master gain):
     - 0–250 Hz: 8.0:1, 10 ms attack, 20 ms release, +3.0 dB band gain
     - 250–4000 Hz: 8.0:1, 10 ms attack, 150 ms release
     - 4000–20000 Hz: 6.0:1, 6 dB knee, 15 ms attack, 150 ms release
   - I like the "multiband compressed" sound with how Apple do their DSP for MacBook, iPhone, etc speakers, so I applied that here.

6. **Pre-Limiter 24 dB/oct Butterworth High-Pass**
   - Cascaded 100 Hz HPQ filters (Q 0.5412 + Q 1.3065) right before limiting so residual sub-100 Hz would not eat limiter's headroom.

7. **LoudMax64** (`LoudMax64.dll`)
   - Threshold -6.0 dB, output ceiling just under 0 dBFS.
   - Total algorithmic DSP lookahead latency measured via Equalizer APO Benchmark.exe at 48 kHz: 60 samples (1.25 ms) (~2.34% single-core load).

---

### Measurement Setup & Limitations

- **How it was measured:** Raw speaker response was measured on a bare ceramic floor (asian house yk lah) using REW and an iSK CM-20 microphone (to capture bottom-firing reflections without desk resonance), then adjusted by ear to taste. Me brokey no UMIK-1, maybe I'll upload my measurement later...

- **Where this shines:** Tuned primarily for modern EDM, pop, and streaming platforms with loudness normalisation enabled (e.g., Apple Music with Sound Check, Spotify "Normal" normalize volume, around -16 to -14 LUFS), where tracks sit cleanly around the -18 dB compressor sweet spot.

- **Loud / 0 dBFS tracks:** Because there is no pre-stage auto-gain before the -18 dB compressor thresholds, masters pinned near 0 dBFS can sound heavily pumped. Lower the source app volume slightly on extra-loud tracks (looking at u YT Music)

- **Acoustic, classical, & quiet tracks:** Not ideal for dynamic acoustic music. `hugebooty` (100% drive) stacks extra saturation onto instruments that already have dense natural harmonics, and heavy multiband dynamics + LoudMax reduce dynamic range while pushing sustained chords into physical speaker breakup/muddiness (a dedicated dynamic-preserving `Clean` preset is planned under `easyeffects/`). There will be another one for Equalizer APO later on, can't promise.

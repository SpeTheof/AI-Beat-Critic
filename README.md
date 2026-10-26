# Aura Beat Critic v2

Drop a mix in, get it measured, then get roasted — with **FL Studio fixes that name your actual problem
frequencies** instead of a hardcoded 300 Hz.

Everything runs locally in the browser. No upload, no account, no build step, no dependencies.
One HTML file.

**Live:** https://spetheof.github.io/AI-Beat-Critic/

---

## What it actually measures

| Metric | Method |
|---|---|
| Integrated loudness | ITU-R BS.1770-4. K-weighting derived for any sample rate, 400 ms blocks at 75 % overlap, −70 LUFS absolute gate + −10 LU relative gate |
| Short-term / momentary | 3 s and 400 ms sliding windows, same weighting |
| Loudness range (LRA) | EBU Tech 3342 — gated 10th-to-95th percentile of short-term loudness |
| True peak | 4× oversampled via windowed-sinc interpolation on peak candidates (sample peak lies to you about intersample overs) |
| PLR / crest factor | Peak-to-loudness and peak-to-RMS |
| Phase correlation | Full-band normalized cross-correlation + **band-limited low-end correlation** from the cross-spectrum |
| Mono compatibility | K-weighted loudness of the mono fold vs stereo — a real level loss, not a guess |
| Spectral tilt | Least-squares fit of log power density vs log frequency, in dB/octave |
| Per-band deviation | Distance from the track's **own** tilt line, not a fixed "ideal" curve |
| Clipping | Per-channel runs of full-scale samples, longest run length |
| DC offset | Per-channel mean |

Spectrum analysis uses a radix-2 FFT with Welch averaging over 240 Hann-windowed 8192-point frames
spread across the track.

### Why tilt-relative balance

Comparing every mix to one hardcoded "ideal" EQ curve fails across genres — trap and acoustic folk
have wildly different tilts, and both can be internally balanced. So v2 fits a line through the mix's
own spectrum and reports each band's distance from it. Bars show **actionable cut/boost amounts**.

---

## Scoring

Seven weighted, individually visible categories — no single opaque formula:

`Headroom 18 · Dynamics 18 · Phase/Mono 18 · Balance 18 · Stereo 10 · Loudness 10 · Cleanliness 8`

Every roast carries the measurement that triggered it, so you can argue with the number instead of
the opinion.

---

## Reference A/B

Drop an optional second file to compare against a target track:

- Overlaid spectrum and loudness curves
- 8-metric delta table
- Per-band differences in dB
- Gain to match reference loudness
- Warns you when the loudness gap is making the tonal comparison lie

---

## Self-test

The engine verifies itself against published conformance signals. Open `index.html?selftest=1`:

- **EBU Tech 3341 case 1** — stereo 1 kHz sine at −23 dBFS peak → −23.0 LUFS ±0.1
- **ITU-R BS.1770** — mono 0 dBFS 1 kHz sine → −3.01 LKFS
- Correlation for identical, inverted, and mono signals
- True peak recovery on a deliberately undersampled 11.025 kHz sine (sample peak 0.636 → true peak 0.90)
- Pink noise synthesized in the frequency domain → tilt −3.01 dB/oct
- Clipping, DC offset, and silent-file handling

Current status: **17/17**.

---

## Known limits

- Files capped at ~4 minutes and 260 MB; longer files are refused with a message rather than hanging.
- `decodeAudioData` resamples to the browser's context rate, so true peak is measured post-resample.
  Matters if a number sits right on the −1.0 dBTP line.
- True peak uses a windowed-sinc approximation, accurate to ~±0.15 dB (BS.1770 Annex 2 allows ±0.5).
- Genre judgement is deliberately conservative — balance is measured against the mix's own trend, so
  a dark genre is not flagged as "dull" for being dark.

---

## Run it

Open `index.html` in a browser. That's the whole install.

For a local server (needed if you want to feed it files over HTTP rather than drag-and-drop):

```sh
python -m http.server 8000
```

---

## History

v1 scored phase-inverted stereo as 100 % mono-compatible, used an O(N²) DFT that froze the tab,
called an intro-vs-drop level gap "dynamic range", never measured LUFS or true peak, and leaked an
AudioContext per file until it died after ~6 uploads. The original is preserved as
`index.v1.backup.html`.

*Built with Aura.*

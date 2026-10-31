# piano-orchestra-separator

A piano / orchestra source separator that runs **entirely in the browser** — no server, no
network calls, no uploads. The shell is a single `index.html` (~32 KB); the model weights
(`models/*.js`) and the built-in demo clip (`demo.js`) are fetched only when you first need
them.

**Live demo:** <https://hhfsb.github.io/piano-orchestra-separator/>

## Features

* Pick a **start / end time** and process only that part of the track.
* **Listen to the original** before running anything.
* A **built-in 16 s demo clip** — Mozart, Piano Concerto No. 20 in D minor, K. 466
  (Yutong Sun, 2022 Cliburn Competition), 20:56–21:12, mono 44.1 kHz.
* Choose between three models:

  | model | size | params / stem | SI-SDR | speed (desktop) |
  |---|---:|---|---:|---|
  | **Student** (UMX20 teacher) | 4.1 MB | 1.9 M, 2-layer LSTM | 3.50 dB | ~2× faster than realtime |
  | **HDMC student** | 4.2 MB | 2.0 M, 2-layer LSTM | 3.52 dB | ~2× faster than realtime |
  | **UMX20 edge** | 15.3 MB | 7.3 M, 3-layer LSTM | 3.83 dB | ~0.7× realtime |

  SI-SDR is measured on 16 held-out piano-concerto excerpts against ground-truth stems.

* **Progress bar** during processing.
* Everything loads lazily — the shell is ~32 KB; a model script is fetched only when you pick
  it, and the demo clip only when you click "Use built-in clip".

## What it does

Takes a music recording and splits it into a **piano** stem and an **orchestra** stem, in the
browser, using an int8-quantised neural network implemented in plain JavaScript.

## What is running

* Student model distilled from the TASLP-2024 piano-concerto separator
  (Özer & Müller, *Source Separation of Piano Concertos Using Musically Motivated Augmentation
  Techniques*), retrained to 2.0 M parameters per stem.
* **int8 weights**, per-output-channel scales — 4.1 MB total for both stems, embedded in the page.
* 2-layer bidirectional LSTM (hidden 256) over an STFT magnitude spectrogram.
* STFT, inverse STFT, ratio masking and 6 s overlap-add are all implemented in plain JS.
* **No WebAssembly, no SIMD, no GPU.** That is deliberate: it gives a conservative lower bound
  for what a phone can do with this model.

## Measured performance

Verified headlessly in Node against the Python reference model:

| model | stem | SI-SDR vs Python |
|---|---|---:|
| Student | piano | 55.5 / 57.1 dB (L/R) |
| Student | orchestra | 50.7 / 54.2 dB |
| HDMC student | piano | 53.3 / 55.0 dB |
| HDMC student | orchestra | 55.6 / 59.8 dB |
| UMX20 edge | piano | 59.6 / 61.3 dB |
| UMX20 edge | orchestra | 47.1 / 51.8 dB |

Those differences are 16-bit WAV quantisation, not algorithmic.

Speed on a desktop CPU: **RTF ≈ 0.52** for the student model (12 s of audio in ~6 s), **RTF ≈ 1.49**
for UMX20 edge. RTF below 1.0 is faster than realtime. A phone is typically 3–6× slower on this
kind of float loop.

## Running locally

Open `index.html` in any modern browser. That's it — there is no build step and no server
required, and it works from `file://` (the model and demo files load as `<script>` tags, not
`fetch`, so a local folder works — keep `index.html`, `demo.js` and `models/` together).

To test on a phone: copy the file to the device and open it, or serve the folder:

```sh
python -m http.server 8000
```

## Deploying to GitHub Pages

```sh
git init
git add .
git commit -m "Piano/orchestra separator: pure-JS browser inference"
git branch -M main
git remote add origin https://github.com/<your-username>/piano-orchestra-separator.git
git push -u origin main
```

Then in the repository: **Settings → Pages → Source: Deploy from a branch → Branch: `main`,
folder: `/ (root)`**. The site appears at
`https://<your-username>.github.io/piano-orchestra-separator/` within a minute or two.

`.nojekyll` is present so GitHub serves the file as-is without Jekyll processing.

## Licence and attribution

Model architecture and training data follow the original work:

> Y. Özer and M. Müller, "Source Separation of Piano Concertos Using Musically Motivated
> Augmentation Techniques", IEEE/ACM TASLP, 32:1214–1225, 2024.

The upstream research code is at <https://github.com/yiitozer/pc-separation>. Check the
licensing terms for the pretrained weights before redistributing them commercially.

### Demo clip

The built-in clip is a 16 s excerpt of Mozart, Piano Concerto No. 20 in D minor, K. 466,
performed by Yutong Sun at the 2022 Cliburn Competition (20:56–21:12). It is included only
as a short demonstration input; all rights remain with the performers and recording owners.
Regenerate it with `edge/make_demo_sample.py` from your own copy of the recording.

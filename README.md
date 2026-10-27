# piano-orchestra-separator

A piano / orchestra source separator that runs **entirely in the browser** — no server, no
network calls, no uploads. Everything (model weights, DSP, UI) is inside a single
`index.html`.

**Live demo:** `https://<your-username>.github.io/piano-orchestra-separator/`

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

| stem | SI-SDR vs Python |
|---|---:|
| piano | 55.5 / 57.1 dB (L/R) |
| orchestra | 50.7 / 54.2 dB (L/R) |

Those differences are 16-bit WAV quantisation, not algorithmic.

Speed on a desktop CPU: **RTF ≈ 0.5** (12 s of audio in ~6 s). RTF below 1.0 is faster than
realtime. A phone is typically 3–6× slower on this kind of float loop, so expect RTF ≈ 1.5–3 —
usable as a batch job, not as realtime processing.

## Running locally

Open `index.html` in any modern browser. That's it — there is no build step and no server
required, and it works from `file://`.

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

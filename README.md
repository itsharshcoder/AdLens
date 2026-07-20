# AdLens

AdLens is a single-file, fully offline analyzer for ad creatives. Point it at an ad video and it grades the things that actually decide whether the ad works - the hook, whether it reads with the sound off, branding, the call-to-action, pacing, production quality - rolls them into a virality score, and hands you a ranked list of fixes and a second-by-second engagement curve. One command, no cloud, no API keys, nothing leaves your machine.

![Python](https://img.shields.io/badge/Python-3.11+-3776AB?logo=python&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-5C3EE8?logo=opencv&logoColor=white)
![Offline](https://img.shields.io/badge/runtime-100%25%20offline-2ea44f)
![Single file](https://img.shields.io/badge/one%20file-no%20setup-4169E1)
![License](https://img.shields.io/badge/license-MIT-green)

## Why I built it

Every brand asks the same question before it spends on distribution: *is this ad any good?* The honest way to answer it - running the creative past a panel, or A/B testing it live - costs money and time, and by then you've already paid to find out the hook was weak or the logo never showed up.

A lot of what makes an ad work is measurable before any of that: does something grab you in the first 3 seconds, does it still make sense on mute, does the CTA actually appear, is the pacing right for a feed. So I built a tool that checks all of it locally, from a single command, and explains what to fix - the kind of pre-flight review a creative strategist would do, but instant and repeatable.

The other hard requirement was reliability. This is meant to run inside a product, so it can't depend on some API that might rate-limit, change, or disappear. Every analysis runs entirely on the machine.

## What it does

Give it an ad video and it produces a full read across several layers:

- **Metadata**, read properly from ffprobe - real codec, profile, bitrate, true fps, rotation, audio channels (not the garbage FOURCC OpenCV often reports).
- **Visual quality** - sharpness, exposure clipping, contrast, colourfulness, colour temperature, blockiness, letter/pillar-box detection, dominant palette.
- **Pacing** - scene cuts and cuts-per-minute at the video's native frame rate, per-shot durations, fade detection.
- **Motion** - camera vs subject motion from optical flow, shake/stability, freeze frames.
- **Faces & hook** - face presence and count, shot types (wide/medium/close-up), rule-of-thirds framing, and a first-3-seconds hook check.
- **On-screen text & branding** - OCR (English **and** Hindi) with per-word confidence filtering, text coverage, CTA keyword detection, and a first-card / end-card branding check.
- **Audio** - integrated LUFS and loudness range (ITU-R BS.1770), peak dBFS, silence and clipping, tempo, spectral features, and a speech-vs-music split. Optional offline speech-to-text (Vosk) reads the spoken hook, pace (WPM) and spoken CTA.
- **Flash / strobe safety** - flags rapid luminance swings that can trigger photosensitivity.

## The score

On top of the raw measurements, AdLens computes nine brand-facing pillars (0-100):

**Hook** · **Scroll-Stopping power** · **Pacing & Retention** · **Sound-Off readiness** · **Branding** · **Call-to-Action** · **Emotional pull** · **Production quality** · **Platform fit** (Reels / TikTok / Shorts / Feed, with UI safe-zone checks)

These roll up into a weighted **Virality Score**, plus a **second-by-second engagement curve** with drop-off risk, your biggest strengths, and a list of **fixes ranked by their likely impact on reach**.

## Runs 100% offline - the design constraint

This was the whole point, so it's worth being precise:

- The AI models are **embedded in the file as base64** - a YuNet face detector (MIT) and a NanoDet object detector (Apache-2.0). Nothing is downloaded at run time.
- OCR and audio use your local `tesseract` and `ffmpeg` binaries; the Hindi OCR data is embedded too.
- **Every analysis run makes zero network calls.** The only thing that ever touches the network is the one-time `pip` install of a genuinely missing library on a fresh machine - and once installed, never again.
- Set `ADLENS_OFFLINE=1` on an air-gapped machine to forbid installs entirely (it then fails fast with manual `pip` instructions).

If a cloud service went down tomorrow, AdLens would not notice.

## Running it

There is no setup step. Just run it:

```bash
python adlens.py path/to/ad.mp4
```

On a brand-new machine, if a required library isn't installed yet, AdLens installs it once from PyPI and then continues - so you always get output from a single command. After that first run, everything is already present and the tool is fully offline.

**Hard dependencies:** `opencv-python (<5)`, `numpy`, `librosa`.
**Optional (degrade gracefully if missing):** `tesseract-ocr` (on-screen text / branding / CTA) and `ffmpeg` / `ffprobe` (audio + accurate metadata).

## What you get

Next to your video, AdLens writes three reports:

- `<video>_report.json` - every metric, for piping into anything else.
- `<video>_report.html` - a self-contained visual report (score gauge, colour-coded scorecard with plain-English explanations, prioritised fixes, transcript, on-screen text, attention curve).
- `<video>_report.pdf` - a clean, print-friendly version to email your team.

## Options

```bash
python adlens.py ad.mp4 --sample-rate 3      # frames/sec to sample (default 2)
python adlens.py ad.mp4 --lang hi            # OCR/speech language: auto | en | hi
python adlens.py ad.mp4 --no-pdf             # skip the PDF (HTML + JSON still written)
python adlens.py ad.mp4 --out result.json    # custom JSON path
python adlens.py --setup                     # pre-install deps without analysing
```

## Honest about the score

The Virality Score is a best-practices creative-QC heuristic, not a calibrated predictor of real-world performance. The technical measurements (LUFS, format and safe-zones, sharpness, pacing, hook presence, face framing) are reliable; the "will it go viral" number is an informed opinion, not a guarantee. Treat it as a fast, consistent second opinion that catches obvious problems before launch, not as a replacement for real testing.

## Tech & credits

Pure-Python, single file. OpenCV for vision, librosa/scipy for audio, pytesseract for OCR, Vosk for offline speech-to-text, fpdf2 for the PDF. Embedded models: **YuNet** face detector (MIT, Shiqi Yu) and **NanoDet-Plus** object detector (Apache-2.0, RangiLyu).

## License

MIT - see [LICENSE](LICENSE).

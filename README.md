# VoiceStudio Experiments — 2026-10-04

Everything here was generated **locally** on one machine (no cloud, no API keys for
speech): NVIDIA **RTX 3090** (24GB) + [VoiceStudio](https://github.com/debpalash/VoiceStudio)
v0.5.6 running as `ghcr.io/debpalash/voicestudio:latest` (Docker, CUDA).

## Models used (all local, ~8.6 GB total)

| Model | Size | Job |
|---|---|---|
| `k2-fsa/OmniVoice` | 2.4 GB | TTS: clone, design, stories, dub voice |
| `Systran/faster-whisper-large-v3` | 2.9 GB | ASR: best word timestamps (dubbing) |
| `deepdml/faster-whisper-large-v3-turbo-ct2` | 1.6 GB | ASR: 5× faster transcription |
| `csukuangfj/sherpa-onnx-whisper-tiny` | 245 MB | Live dictation, 90+ langs, CPU |
| `csukuangfj/sherpa-onnx-nemo-parakeet-tdt-0.6b-v3-int8` | 682 MB | Best EN/EU dictation accuracy |
| `KittenML/kitten-tts-mini-0.8` | 110 MB | Instant English preset voices, CPU |
| Argos offline packs | ~100–200 MB each | `en↔es`, `en→ar`, `ar→en` translators |

> What each model costs in VRAM/RAM/CPU, and what smaller machines need:
> see **[HARDWARE.md](HARDWARE.md)**.

## Experiments (in order)

### 1. Voice clone → new speech (`audio/01-clone-playwright-voice.wav`, 11.96 s)
Cloned `demo_voice.wav` (7.28 s reference) into profile *"Playwright Clone Test"*,
then generated a brand-new sentence with it:
> "This is a real clone test from the bundled demo voice, rendered locally on the
> RTX 3090. The voice was cloned from a seven second reference and speaks this
> new sentence."

### 2. Voice design (`audio/02-design-first-take.wav`, 7.1 s + `screenshots/01`)
Designed voice (Narrator, middle-aged · low pitch), script:
> "Welcome aboard. I was just a three-second clip a moment ago — now I can say
> anything you'd like, in your voice or mine."

### 3. KittenTTS preset voice (`audio/03-kittentts-default-voice.wav`, 11.2 s)
`engine=kittentts` via the local API, no cloning step:
> "Hello from Kitten, the tiny turbo voice. Eight presets, English only, renders
> in a blink on plain CPU."

### 4. Video dub EN→ES (`audio/04-dub-spanish-mix.wav` + `screenshots/02`)
`source.mp4` (11 s) → transcribe (2 timed segments) → Argos `en→es` → TTS → mux.
Status went Extracting → Transcribing → Translations ready → **Dub complete**.
Segments: 0:00–0:04.5 and 0:04.8–0:11.0.

### 5b. Arabic v2 — natural rate (`audio/08-dub-arabic-v2-natural-rate.wav`, 15.4 s mix)
The v1 Arabic sounded rushed because the job used `strict_slot` timing: 6.0 s of
speech squeezed into a 4.7 s slot (1.28×) and 8.7 s into 6.6 s (1.32×). Per 2026
best practice (tashkeel-aware text + natural timing over slot compression), v2
fixes two things: corrected MSA wording (التلاعب بالفيديو → دبلجة الفيديو,
feminine افتحي/ابدأي → neutral افتح/ابدأ, آلتك → جهازك) and re-rendered with
`stretch_video` timing — speech at natural rate (6.71 s + 8.03 s), video yields
instead of the voice. Same Speaker-1 clone for comparability.
Finding: the UI dub page blocks generic "Arabic" (*Choose a supported language*)
because the engine's 646 display names hold 21 dialects but no plain "Arabic";
the backend (`supported_languages = ["multi"]`) synthesizes `ar` fine, so v2
rendered via the API.

### 6. Transcription shootout (Whisper Tiny vs Parakeet v3, same clips)
`en_conversational.wav` → **both word-perfect**:
> "Schedule a meeting with Pat for Tuesday at 3 p.m. and remind me to bring the
> quarterly report."
`en_technical.wav` → near-tie on jargon (Tiny: "renderer.to 6", Parakeet:
"renderer.tisx"; Parakeet punctuates better).
`fr_reservation.wav` → Parakeet auto-detected French:
> "Bonjour, je voudrais réserver une table pour deux personnes à 20 heures."

### 7. Story render (`audio/07-lighthouse-story.m4b`, 0:59 + `screenshots/03`)
Sample story *"The Lighthouse at Wits' End"* (2 chapters, 151 words, pauses and
`[laughter]`/`[sigh]` markup) → M4B audiobook with chapters.

## Reproduce

```sh
# 1. Start VoiceStudio (GPU profile), note the API key it prints/requires
docker compose -f deploy/docker-compose.yml --profile gpu up
# 2. Open http://localhost:3900, complete setup (models above install one-click
#    from Settings → Models; no HuggingFace token needed — all are public)
# 3. Clone / Design / Transcribe / Dub / Stories as in the app workspaces
```

## Notes & licenses

- Audio here is synthetic and watermarked by VoiceStudio's invisible-watermark
  default. Voices used: the bundled demo voice + a test clone of it.
- VoiceStudio is AGPL-3.0; model weights carry their own licenses (review before
  commercial use); clone voices only with permission.
- Two hiccups hit during the session, both recovered: transient HF download
  stalls (resume/retry completed them) and a browser re-auth gate after backend
  idle-unload (re-enter `OMNIVOICE_API_KEY`).

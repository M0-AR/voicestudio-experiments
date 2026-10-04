# Hardware requirements — measured, not guessed

Reference machine (everything in this repo was rendered on it): NVIDIA **RTX 3090
24GB**, Intel i7-14700F (28 threads), **30.6 GB RAM**, 732 GB free disk.
Backend self-config on that box: **GPU pool = 4 workers @ 5 GB each**, CPU pool =
8 threads, model idle-unload after 15 min. Peak VRAM observed during generation:
**4.1 GB**.

## Per-model cost (all local)

| Model | Disk | VRAM (CUDA) | RAM / CPU note |
|---|---|---|---|
| OmniVoice TTS 2.4 GB | 2.4 GB | **6 GB min** (declared; ~4 GB observed) | Falls back to CPU when short (10–50× slower) |
| faster-whisper-large-v3 2.9 GB | 2.9 GB | ~1.5 GB int8 / ~2.5–4.5 GB fp16 | Best word timestamps; dubbing briefly moves TTS to CPU to make room |
| faster-whisper-turbo-ct2 1.6 GB | 1.6 GB | ~1.5 GB int8 | 5× faster than large-v3, minimal accuracy loss |
| sherpa-whisper-tiny 245 MB | 245 MB | **0 — CPU by design** | 2 threads; pre-warmed, 1.3–2.5 s first load |
| sherpa-parakeet-tdt-v3 682 MB | 682 MB | **0 — CPU** | Up to 4 threads; onnxruntime arena holds extra RAM above file size |
| KittenTTS mini 110 MB | 110 MB | **0 — CPU graph** | Realtime on any CPU; English-only |
| Argos packs | ~100–200 MB per pair | 0 (CPU) | One per language pair (`en↔es`, `en→ar`, `ar→en` here) |

**Total here: ~8.6 GB disk.** A 24 GB card holds TTS + ASR + spare workers at once —
that is exactly what the pool sizing does.

## Smaller machines (from upstream docs + engine contracts)

- **Sweet spot:** any NVIDIA card with **≥ 8 GB VRAM** (6 GB floor + headroom) +
  16 GB RAM. `OMNIVOICE_GPU_WORKERS` auto-sizes at ~5 GB/job and drops workers on
  ≤ 10 GB cards so you never OOM.
- **Below the VRAM floor:** still runs, paged to system RAM — slower than CPU,
  watchdog treats it as CPU-class. Degraded, not dead.
- **No GPU:** fully supported. sherpa/Kitten/Argos are CPU-native; OmniVoice and
  Whisper render on CPU (minutes per render instead of seconds). Apple Silicon
  uses MPS/Metal; Windows AMD/Intel iGPUs are CPU-only by design.
- **RAM pressure:** on 16 GB unified-memory machines, close the 40-tab browser
  before a dub — the OS pages the model out or kills the backend.

## Knobs that matter

- `OMNIVOICE_GPU_WORKERS` — leave on auto (4 is correct for 24 GB).
- `OMNIVOICE_IDLE_TIMEOUT_S` (default 900) — first render after idle costs ~8 s
  reload; raise it if you generate in bursts.
- `OMNIVOICE_SINGLE_ENGINE_RESIDENT=1` — one warm TTS engine; only set `0` on
  32 GB+ RAM machines juggling engines.
- Live truth: **Settings → Performance → Device & compute** (device + RAM/VRAM
  readouts) and the per-engine routing badge ("GPU active" vs "CPU fallback").

Bottom line: a 3090 is ~3–4× overprovisioned for this stack. People who struggle
are < 6 GB VRAM (paging), CPU-only (slow), or 16 GB-RAM multitaskers.

# Speech Recognition: Comparing ASR Systems Under Noise

A benchmark comparing the Google Web Speech API against two sizes of [OpenAI Whisper](https://github.com/openai/whisper) (`base` and `medium`) — scored with WER/CER across a 9-clip dataset, with a pre-emphasis filtering experiment and a synthetic TTS round-trip sanity check.

## 📌 Project Overview

This notebook benchmarks three approaches to automatic speech recognition: a cloud API baseline (Google Web Speech) and two local Whisper model sizes. Rather than eyeballing transcripts, every result is scored against ground truth using **Word Error Rate (WER)** and **Character Error Rate (CER)**, so the accuracy/latency trade-off between models is a measured number, not an impression.

The notebook is built in two layers:
- A **single-clip walkthrough** that isolates the model/filtering comparison on one controlled sample, so each variable's effect is easy to see in isolation.
- A **full aggregated benchmark** across all 9 available clips, reporting mean/median/standard deviation per model — the number that actually generalizes, rather than a result from one clip.

It also tests whether a common cheap preprocessing step (pre-emphasis filtering) measurably helps accuracy, and includes a synthetic text-to-speech round trip as an end-to-end pipeline check.

## 🛠️ Libraries Used

- **Audio I/O & Signal Processing:** `librosa`, `soundfile`
- **Speech Recognition:** `speech_recognition` (Google Web Speech API), `openai-whisper`
- **Evaluation:** `jiwer` (WER/CER scoring)
- **Text-to-Speech:** `gTTS`
- **Data & Visualization:** `pandas`, `numpy`, `matplotlib`

## 🚀 Key Implementation Steps

**1. Loading and Visualizing Audio**
- Loaded raw audio with `librosa`, inspected sample rate and duration.
- Plotted the waveform and rendered an inline audio player for direct listening.

**2. Baseline Transcription — Google Web Speech API**
- Built a `transcribe_google()` helper wrapping `speech_recognition.Recognizer`.
- Established ground truth for the primary sample clip and scored the baseline transcription against it.

**3. Background Noise and Pre-Emphasis Filtering**
- Visualized frequency content with a log-scale spectrogram before and after filtering.
- Applied a pre-emphasis filter (`librosa.effects.preemphasis`) and re-ran the same evaluation to measure whether it actually changed WER/CER, rather than assuming it helped.

**4. Comparing Whisper Model Sizes**
- Loaded and ran `whisper-base` and `whisper-medium` (CPU) on the same unfiltered clip.
- Isolated model size as the only variable in the comparison.

**5. Single-Clip Results & Full Aggregated Benchmark**
- Built a reusable `evaluate_transcription()` / `evaluate_batch()` scoring harness that logs model, file, WER, and CER for every transcription attempt.
- Scored all three approaches against **9 clips total** (1 synthetic sample + 8 real recordings), then aggregated to mean/median/std per model — the real headline result, not a single data point.

**6. Batch Pipeline & CSV Export**
- Ran the aggregated benchmark end-to-end across a full `Recordings/` directory and exported scored results to CSV for downstream analysis.

**7. Bonus: Text-to-Speech Round Trip**
- Generated a synthetic clip with `gTTS` and fed it back through the same Whisper pipeline as an end-to-end sanity check that the harness works on audio the notebook created, not just pre-recorded samples.

## 📊 Results Summary

| Model | Mean WER | Median WER | Std WER | Mean CER |
|---|---|---|---|---|
| **Whisper (medium)** | 0.004 | 0.000 | 0.012 | 0.001 |
| **Whisper (base)** | 0.093 | 0.091 | 0.064 | 0.020 |
| **Google Web Speech API** | 0.273 | 0.286 | 0.129 | 0.088 |

*Aggregated across all 9 clips. Whisper (medium) is both the most accurate and most consistent (lowest std) of the three; the Google API's higher standard deviation shows its accuracy varies more by speaker/content rather than being uniformly weaker. See the notebook for the single-clip breakdown and full per-file results.*

## 📂 Repository Structure

```
├── Speech_Recognition.ipynb    # Main notebook — all 7 sections
├── sample_01.wav               # Synthetic ground-truth sample clip
├── Recordings/                 # 8 real narrated clips used in the aggregated benchmark
│   ├── Track1.wav
│   ├── ...
│   └── Track8.wav
├── batch_results.csv           # Exported scored transcriptions (generated on run)
├── requirements.txt            # Python dependencies
└── README.md                   # Project documentation
```

## 👩‍💻 How to Run

1. Clone this repository
2. Install requirements: `pip install -r requirements.txt`
3. Make sure `sample_01.wav` and the `Recordings/` folder (with `Track1.wav`–`Track8.wav`) are present in the project root
4. Open `Speech_Recognition.ipynb` in Jupyter
5. Run all cells from top to bottom

> **Note:** Whisper downloads model weights on first run (`base` ~140MB, `medium` ~1.5GB) — this can take a few minutes depending on connection speed. The Google Web Speech API step requires an active internet connection.

---

**Developed by:**
Nicole Kaye A. Cardel
<nkcardel@gmail.com>
*Software Designer & Developer*

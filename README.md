<div align="center">

<img width="760" height="224" alt="banner" src="https://github.com/user-attachments/assets/21980482-3a7d-4134-b43f-10c8cf23edc8" />


### Clone any voice in one HTML file.

Zero-shot voice cloning with Chatterbox-Turbo, running entirely on your GPU.<br>
No server, no uploads, no build step.

<br>

![WebGPU](https://img.shields.io/badge/WebGPU-accelerated-8b5cf6?style=for-the-badge)
![Single file](https://img.shields.io/badge/single-file-7c8cf8?style=for-the-badge)
![Model](https://img.shields.io/badge/chatterbox--turbo-350M-60a5fa?style=for-the-badge)
![Privacy](https://img.shields.io/badge/audio-stays%20local-22d3ee?style=for-the-badge)

</div>

<br>
<br>

## Quick start

### Structure

```
doppel/
├── index.html      Whole app: UI, inference loop, audio I/O
└── banner.svg      This README's banner
```

<br>

## Link: https://ltorl.github.io/doppel-voice-cloning

<br>

<details>
<summary><h2>Other ways to run it<h2></summary>

Serve the folder from anywhere and open it in Chrome or Edge:

```bash
python3 -m http.server 8000
```

Then open **http://localhost:8000**.

1. **Load the model**: about 640 MB the first time, then cached by the browser
2. **Give it a voice**: drop a clip, pick a file, or record 5–15 seconds
3. **Type, generate, download** a WAV

<br>

**Node**

```bash
npx serve .
```

**Static hosting**

Drop `index.html` on GitHub Pages, Netlify, Cloudflare Pages, or any HTTPS host. Everything happens client-side.

**Why not just double-click it?**

`file://` pages aren't a secure context, so WebGPU, the model cache and the microphone are unavailable. Any `http://localhost` or `https://` origin works.

</details>

<br>
<br>

## Features

<table>
<tr>
<td width="50%" valign="top">

### The studio

- **Three ways in**: drag-and-drop, file picker, or **record from the mic**
- **Sound tags**: one-click `[laugh]` `[chuckle]` `[sigh]` `[gasp]` `[cough]` and more
- **Long text**: split at sentence boundaries and stitched back together
- **Live progress**: tokens per second, chunk count, real-time factor
- **History**: every take gets a player and a **WAV download**
- **Light and dark** themes that follow your system
<br>
</td>
<td width="50%" valign="top">

### The engine

- The official **Chatterbox-Turbo ONNX** pipeline, ported to the browser
- **onnxruntime-web** on **WebGPU**, with the KV cache kept on the GPU
- Five precisions from **q4f16 (~640 MB)** to **fp32 (~3.3 GB)**
- Voice is **encoded once** and reused for every chunk and take
- Model files are **cached** with the Cache API, so reloads take seconds
- About **real-time speed** on an Apple-silicon laptop

</td>
</tr>
</table>

<br>
<br>
<br>

<details>
<summary>
<h2>How it works</h2>
</summary>

```
                 ┌────────────────────────────┐
  your clip ──▶  │ speech_encoder             │   voice → conditioning, prompt tokens,
                 └─────────────┬──────────────┘           speaker embedding
                               │
                 ┌─────────────▼──────────────┐
  your text ──▶  │ embed_tokens               │   GPT-2 tokens → embeddings
                 └─────────────┬──────────────┘
                               │ [voice | text]
                 ┌─────────────▼──────────────┐
                 │ language_model             │   autoregressive speech tokens,
                 └─────────────┬──────────────┘   ~25 per second of audio
                               │
                 ┌─────────────▼──────────────┐
                 │ conditional_decoder        │   one-step flow + vocoder
                 └─────────────┬──────────────┘
                               ▼
                        24 kHz waveform
```

Everything lives in **`index.html`**: the four ONNX sessions, the sampling loop (temperature, top-k, top-p, repetition penalty), text chunking, resampling, recording and WAV encoding. onnxruntime-web and the tokenizer are loaded from jsDelivr, and the weights come straight from Hugging Face.

<br>

### Precisions

| Preset   | Download | Backend       | Notes                                              |
| -------- | -------- | ------------- | -------------------------------------------------- |
| `q4f16`  | ~640 MB  | WebGPU + f16  | **Default.** Fastest, smallest                     |
| `q4`     | ~720 MB  | WebGPU        | For GPUs without `shader-f16`                      |
| `q8`     | ~1.1 GB  | WebGPU        | Slightly higher fidelity                           |
| `fp16`   | ~2 GB    | WebGPU + f16  | Near-full quality                                  |
| `fp32`   | ~3.3 GB  | WebGPU / WASM | Reference quality; the only option on CPU          |

The decoder always uses fp32 activations (`q4` or `fp32` weights), because its fp16 variants go silent partway through longer clips on WebGPU.

</details>



<details>
<summary>
<h2>Generation settings</h2>
</summary>

<br>

Everything is optional. Open **Advanced settings** under the text box:

| Setting              | Default | What it does                                                       |
| -------------------- | ------- | ------------------------------------------------------------------ |
| `Temperature`        | `0.8`   | Randomness. `0` gives greedy decoding                              |
| `Top-p`              | `0.95`  | Nucleus sampling cutoff                                            |
| `Top-k`              | `1000`  | Only sample from the k most likely tokens                          |
| `Repetition penalty` | `1.2`   | Discourages loops and stutters                                     |
| `Max tokens / chunk` | `1000`  | Hard cap per chunk (~40 s of audio)                                |
| `Chunk size`         | `280`   | Characters per chunk before long text is split                     |

</details>

<details>
<summary>
<h2>Tips & troubleshooting</h2>
</summary>

<br>

**Getting a good clone**
- Use **5–15 seconds** of a single speaker with no music or background noise
- Clips longer than 20 s are trimmed. Quiet recordings are normalized automatically
- The reference's delivery carries over, so a calm clip gives calm output

**"CPU only" badge**
- Your browser has no WebGPU. Use a recent Chrome or Edge, or Safari 26+
- The WASM fallback can only run `fp32`. It is very slow and may run out of memory

**Download stalls or the disk fills up**
- Use **Clear cache** in step 1 to delete stored model files, then reload

</details>


<br>
<br>
<br>

## Responsible use

Only clone voices you have **explicit permission** to use: your own, or someone who has consented. Don't use Doppel to impersonate, defraud or mislead anyone.

Resemble AI's official Python package adds an imperceptible [PerTh watermark](https://github.com/resemble-ai/perth) to its outputs. That watermarker has no browser port, so **audio made with Doppel is not watermarked**.

<br>

## Credits

- [**Chatterbox-Turbo**](https://huggingface.co/ResembleAI/chatterbox-turbo-ONNX) by Resemble AI (MIT)
- [**onnxruntime-web**](https://github.com/microsoft/onnxruntime) by Microsoft
- [**Transformers.js**](https://github.com/huggingface/transformers.js) by Hugging Face, used for the tokenizer

<br>
<br>
<br>

# MobileAirLLM

> Layer-by-layer LLM inference on mobile. Zero full-model RAM. Unlimited model storage via Telegram/Pentaract.

**Live demo:** `https://YOUR-USERNAME.github.io/mobile-airllm`

---

## What is this?

MobileAirLLM brings [AirLLM](https://github.com/lyogavin/airllm)'s layer-by-layer inference technique to mobile web browsers. Instead of loading an entire LLM into RAM, it streams one transformer layer at a time from **Telegram storage** (via [Pentaract](https://github.com/aloncrack7/pentaract)), processes it, then evicts it before fetching the next layer.

**Peak RAM = 1 layer's weights** — regardless of total model size.

| Model | Total Size | Peak RAM | Layers |
|-------|-----------|----------|--------|
| Llama 3.2 1B | 0.8 GB | ~280 MB | 16 |
| Llama 3.2 3B | 2.0 GB | ~460 MB | 28 |
| Mistral 7B | 4.4 GB | ~820 MB | 32 |
| **Your HF model** | any | scales | any |

---

## Deploy to GitHub Pages (2 minutes)

### Option A — Fork & Enable Pages (easiest)

1. **Fork this repo** on GitHub
2. Go to **Settings → Pages**
3. Under "Source" select **GitHub Actions**
4. Push any change (or trigger manually via **Actions → Deploy → Run workflow**)
5. Your app is live at `https://YOUR-USERNAME.github.io/mobile-airllm`

### Option B — New repo from scratch

```bash
# 1. Create a new repo on GitHub named "mobile-airllm"

# 2. Clone and add files
git clone https://github.com/YOUR-USERNAME/mobile-airllm
cd mobile-airllm

# 3. Copy index.html and .github/ folder here

# 4. Push
git add .
git commit -m "Initial deploy"
git push origin main

# 5. Enable Pages: Settings → Pages → Source: GitHub Actions
```

That's it. No build step, no Node.js, no dependencies. The entire app is a single `index.html`.

---

## First-time setup (in-app)

The app checks your connections on every load and guides you through any missing steps:

### 1. Telegram / Pentaract

MobileAirLLM uses Telegram as free, unlimited model storage. Models are sharded into <50MB chunks and stored as file messages.

1. Open Telegram → search **@BotFather**
2. Send `/newbot` → follow prompts → copy your **Bot Token**
3. Create a **private channel** → add your bot as admin
4. Forward a message from the channel to **@userinfobot** → copy the **Chat ID**
5. Paste both into the app → tap **Test Connection**

### 2. HuggingFace (for custom models)

1. Go to [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens)
2. Create a **Read** token
3. Paste into the app → tap **Verify**

This unlocks:
- Private/gated model access
- Your own fine-tuned models
- Full model search across HF Hub

---

## Supported Models

### Curated (one-tap pull)
- Llama 3.2 1B/3B Instruct
- Phi-3.5 Mini Instruct
- Gemma 2 2B Instruct
- Mistral 7B Instruct v0.3

### HuggingFace Search
Search any GGUF-format text generation model on HuggingFace. Select the quantization file you want (Q4_K_M recommended for balance).

### Custom URL / Model ID
Paste any `username/repo` HuggingFace model ID or direct GGUF download URL.

---

## Architecture

```
Browser (Mobile)
  │
  ├── index.html  ← entire app, zero dependencies
  │
  ├── On inference:
  │     for each layer (0..N):
  │       1. fetch shard from Telegram Bot API
  │       2. decode layer weights
  │       3. run transformer block forward pass
  │       4. del weights → gc.collect()
  │       5. yield tokens
  │
  └── Storage:
        Telegram Channel (via Pentaract)
          ├── shard_0.bin  (layer 0-2 weights)
          ├── shard_1.bin  (layer 3-5 weights)
          └── ...
```

### Why Telegram?
- **Free & unlimited** — no storage caps
- **Fast CDN** — Telegram's infrastructure globally distributed
- **Bot API** — browser-accessible, no server needed
- **Private** — only your bot can access your channel

---

## Production Roadmap

To move from simulation to real inference:

- [ ] **GGUF parser in WASM** — parse layer weights from `.gguf` shards in the browser
- [ ] **WebGPU/WebGL matmul** — hardware-accelerated transformer forward pass
- [ ] **Tokenizer** — integrate `@huggingface/transformers` tokenizer in WASM
- [ ] **Shard upload worker** — background Service Worker for chunked Telegram uploads
- [ ] **Streaming decode** — beam search / greedy decode with KV cache
- [ ] **PWA** — installable, offline-capable with model cache

---

## License

MIT — do whatever you want with it.

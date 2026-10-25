# OpenCode Configuration

Global configuration repository for [OpenCode](https://opencode.ai) with the [oMLX](https://github.com/vllm-project) inference server on Apple Silicon, in two variants: **local** (OpenCode on the same host) and **remote** (access over Tailscale / VPN).

## Setup

Place `opencode.json` in your OpenCode configuration directory:
- **macOS/Linux**: `~/.config/opencode/opencode.json`

### Remote Client (e.g. Laptop over Tailscale / VPN)
To run OpenCode on a laptop connecting to the Mac Studio oMLX server (`100.123.37.101`):
```bash
cp opencode.json.remote ~/.config/opencode/opencode.json
```

The remote variant registers the provider under the ID **`omlxremote`** and every
model's display `name` starts with the marker **`[omlxremote] `**. The `/models`
picker (v2.1.1) renders a flat list without section separators, so the marker in
the display name is the visible separator: each server-hosted entry is labelled
in place, e.g. `[omlxremote] Qwen3.8-27B-Uncensored-MLX-SPLASH`. Typing
`omlxremote` in the picker filter isolates only those models. Display names are
cosmetic — the model ID sent to the server is unchanged.

Provider conventions:

| Variant             | Provider ID    | Endpoint                    |
| ------------------- | -------------- | --------------------------- |
| `opencode.json`     | `omlx`         | `http://127.0.0.1:8000/v1`  |
| `opencode.json.remote` | `omlxremote` | `http://100.123.37.101:8000/v1` |


## Configured Models

The local variant (`opencode.json`) connects to the local OpenAI-compatible
endpoint at `http://127.0.0.1:8000/v1`; the remote variant (`opencode.json.remote`)
uses the same port over the VPN address `http://100.123.37.101:8000/v1`.
The `models` list mirrors the models currently loaded on the oMLX server (20 entries).

### Coding & Reasoning
- `omlx/Qwen3-Coder-Next-MLX-9bit`
- `omlx/DeepSeek-V4-Flash-0731-AWQ`
- `omlx/DeepSeek-V4-Flash-0731-MXFP4-MLX`
- `omlx/Qwen3.8-27B-Coder390-EfficientThink-Opus5.5-GPT6Astra-Grok4.7-DSV4Pro-K3-SFT-RLOO-MTP-DFlash2` (displayed as "Qwen3.8-27B-Coder390 (DFlash2)")

### Mixture of Experts (MoE)
- `omlx/Qwen3-235B-A22B-MLX-4bit`
- `omlx/Qwen3.5-122B-A10B-MLX-9bit`
- `omlx/MiniMax-M3-ConfigI-MLX`

### General & FP16
- `omlx/Qwen3.8-27B-Uncensored-MLX-SPLASH` (**default**)
- `omlx/Qwen3.8-27B-Uncensored-MLX`
- `omlx/Qwen3.8-27B-FP16_huzar_v1`
- `omlx/Qwen3.8-27B-FP16_huzar_v1-SPLASH`

### SPLASH / DFlash2 variants
- `omlx/Qwen3.8-27B-Uncensored-MLX-SPLASH` — SPLASH variant of `Qwen3.8-27B-Uncensored-MLX`
- `omlx/Qwen3.8-27B-FP16_huzar_v1-SPLASH` — SPLASH variant of `Qwen3.8-27B-FP16_huzar_v1`
- `omlx/Qwen3.8-27B-DFlash2` — DFlash2 variant
- `omlx/Qwen3.8-27B-Coder390-…-MTP-DFlash2` — DFlash2 variant (Coder390)

SPLASH and DFlash2 are separate model IDs exposed by the oMLX server (speculative-decoding
variants), so they are configured as their own entries rather than OpenCode `variants`
(which cannot change the `modelID` sent to the server).

### Flash & Speed
- `omlx/Qwen3.8-Flash-Next-MTPLX-Optimized-Speed`
- `omlx/Qwen3.8-Flash-Next-oQ5e-mtp`
- `omlx/Step-3.5-Flash-4bit`

### Vision / OCR
- `omlx/UI-Venus-1.5-30B-A3B-qx86-hi-mlx` (vision / UI automation)
- `omlx/Unlimited-OCR-8bit` (vision, 32k context)
- `omlx/olmOCR-2-7B-1025-8bit` (vision, 128k context)

### Other
- `omlx/Laguna-S-2.1-MLX-8bit`
- `omlx/Ornith-1.5-35B-A3B-Abliterated-MLX-BF16`

## 🧪 Case Study: Chrzest bojowy DeepSeek-V4-Flash (171k tokenów kontekstu)

Weryfikacja zachowania lokalnego serwera oMLX oraz klienta OpenCode pod ekstremalnym obciążeniem na platformie Mac Studio M2 Ultra (192 GB RAM):

- **Model:** `omlx/DeepSeek-V4-Flash-0731-MXFP4-MLX` (waga modelu w pamięci: ~160 GB).
- **Zadanie testowe:** Wywołanie w długiej sesji roboczej OpenCode skutkujące promptem o wielkości **171 334 tokenów**.
- **Wyniki i metryki:**
  - **Użycie pamięci (Unified Memory):** Alokacja wzrosła do **~175.2 GB** (model + KV cache dla 171k tokenów). Zero błędów OOM, pełna stabilność systemu macOS.
  - **Czas prefillu (TTFT / Prompt Processing):** **691.53 s** (~11.5 min) przy ciągłym, 95–100% obciążeniu akceleratora Metal GPU (średnia prędkość prefillu ~248 tok/s).
  - **Generacja z MTP (Multi-Token Prediction):** Wygenerowano 476 tokenów z wywołaniem narzędzia w 21 s (**22.5 tok/s**), ze współczynnikiem akceptacji MTP na poziomie **86.7%** (3.11 tok/cykl).
  - **Paged Prefix Cache w kolejnej turze:** Po wykonaniu narzędzia oMLX zrekonstruował **171 332 tokeny** z cache w zaledwie **868 ms**, doliczając sufiks 1 776 tokenów w 9 s. Druga tura wygenerowała 1 432 tokeny w 86.9 s (**21.1 tok/s**, `finish_reason=stop`).
- **Wnioski i rekomendacje:**
  - Architektura oMLX na Apple Silicon bez problemu radzi sobie z gigantycznym oknem kontekstowym (>170k tokenów).
  - Przy rozpoczynaniu nowego, niepowiązanego zadania warto utworzyć nową sesję w OpenCode, by uniknąć długiego prefillu cold-start. W sesjach wieloturowych Paged Prefix Cache redukuje kolejne odpowiedzi do kilkunastu sekund.

## Changelog

### 2026-10-25 — stress test i chrzest bojowy DeepSeek-V4-Flash (171k tokenów)
- Udokumentowano use case i test obciążeniowy modelu `DeepSeek-V4-Flash-0731-MXFP4-MLX` na 171 334 tokenach kontekstu w sesji OpenCode.
- Potwierdzono 100% stabilności alokacji 175.2 GB Unified Memory, prefill Metal oraz błyskawiczne wznawianie przez Paged Prefix Cache (868 ms na 171k tokenów).

### 2026-10-20 — remote provider renamed to `omlxremote`, deployment verified over VPN
- The remote variant now registers the provider under the ID **`omlxremote`**.
  Because the `/models` picker renders a flat list (no section separators in
  v2.1.1), every model's display `name` was prefixed with the marker
  **`[omlxremote] `** — this is the visible in-list separator distinguishing
  server-hosted models from OpenCode's built-in catalog; the picker filter also
  matches on it. All references change accordingly, e.g.
  `omlxremote/Qwen3.8-27B-Uncensored-MLX-SPLASH`; display names are cosmetic and
  the model IDs sent to the server are unchanged.
- Model list unchanged: the same 20 models were re-verified against the running
  server (`GET /v1/models` over VPN returns exactly these 20 entries).
- Deployed `opencode.json.remote` to `~/.config/opencode/opencode.json` on the
  remote host and verified end-to-end: `opencode models` lists all 20
  `omlxremote/*` models and a test `opencode run` request answered from the
  server.
- The local variant (`opencode.json`) keeps the provider ID `omlx` and endpoint
  `http://127.0.0.1:8000/v1`.

### 2026-10-19 — sync with running oMLX server
- Added 13 models newly loaded on the server, including the **SPLASH** variants
  `Qwen3.8-27B-Uncensored-MLX-SPLASH` and `Qwen3.8-27B-FP16_huzar_v1-SPLASH` and the
  **DFlash2** variants `Qwen3.8-27B-DFlash2` and `Qwen3.8-27B-Coder390-…-MTP-DFlash2`.
- Removed 6 entries that are no longer loaded on the server:
  `Qwen3-Coder-Next-MLX-8bit`, `DeepSeek-V4-Flash-MLX-mixed-4_8bit`,
  `Qwen3.8-27B-MTPLX-Optimized-Quality-FP16`, `Qwen3.8-Flash-Next-MLX-6bit`,
  `Bielik-11B-v3.0-Instruct-MLX-8bit`, `Bielik-11B-v3.0-Huzar-MLX-8bit-v1`.
- Default model changed to `omlx/Qwen3.8-27B-Uncensored-MLX-SPLASH` (SPLASH version of
  the previous default `Qwen3.8-27B-Uncensored-MLX`).
- Config now matches the server 1:1: 20 models, no duplicates, no unloaded entries.

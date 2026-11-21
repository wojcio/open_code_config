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
The `models` list mirrors the **28 chat/vision models** configured on the oMLX server (`oMLX 0.8.2` + `Splash 1.7.0-m1`, 186 GB RAM ceiling).

### Flash & Speed (VLM + FP16 Activation Recast + Lightning MTP)
- `omlx/Qwen3.8-Flash-Next-oQ5e-mtp` (displayed at the top as `"Qwen3.8-Flash-Next-oQ5e-mtp (Qwen 3.8 Flash · Lightning MTP · VLM)"`, **67.5–73.0 tok/s**, vision enabled)
- `omlx/Qwen3.8-Flash-Next-oQ6e-mtp` (displayed as `"Qwen3.8-Flash-Next-oQ6e-mtp (Qwen 3.8 Flash · 6-bit · Lightning MTP · VLM)"`, **60.6–63.9 tok/s**, vision enabled)
- `omlx/Qwen3.8-Flash-Next-MTPLX-Optimized-Speed` (displayed as `"Qwen3.8-Flash-Next-MTPLX-Optimized-Speed (Qwen 3.8 Flash · N-gram · VLM)"`, **~73.5 tok/s**, vision enabled)
- `omlx/gemma-4-12B-it-oQ8e-mtp` (displayed as `"gemma-4-12B-it-oQ8e-mtp (Gemma 4 12B · 8-bit · Lightning MTP · VLM/Audio)"`, **51.4–68.4 tok/s**, vision enabled)
- `omlx/gemma-4-31B-it-oQ6e-mtp` (displayed as `"gemma-4-31B-it-oQ6e-mtp (Gemma 4 31B · 6-bit · Lightning MTP · VLM)"`, **32.4–34.0 tok/s**, vision enabled)
- `omlx/Ternary-Bonsai-2-27B-MTPLX-Optimized-Speed` (displayed as `"Ternary-Bonsai-2-27B-MTPLX-Optimized-Speed (2-bit Hadamard · 8.2 GB · VLM)"`, **31.4 tok/s**, vision enabled)
- `omlx/Step-3.5-Flash-4bit`

### GLM-5.3-Flash 320B MoE (18B Active · Lightning MTP · VLM)
- `omlx/GLM-5.3-Flash-oQ4e-mtp` (displayed as `"GLM-5.3-Flash-oQ4e-mtp (GLM 5.3 Flash 320B · 4-bit · Lightning MTP · VLM)"`, 173.9 GB RAM, **25.9–29.9 tok/s**, TTFT **0.17s**, vision enabled)
*(Note: `GLM-5.3-Flash-oQ2e-mtp` was evaluated and removed on 2026-11-13 in favor of `Qwen3.8-Flash-Next-oQ5e-mtp`.)*

### Coding & Reasoning
- `omlx/flint-oQ8e` (**54.8–74.1 tok/s**)
- `omlx/Qwen3.8-27B-Coder390-EfficientThink-Opus5.5-GPT6Astra-Grok4.7-DSV4Pro-K3-SFT-RLOO-MTP-DFlash2` (displayed as `"Qwen3.8-27B-Coder390 (VLM · MTP)"`, vision enabled)
- `omlx/Nanbeige4.3-33B-MLX-6bit` (displayed as `"Nanbeige4.3-33B-MLX-6bit (Dense 33B · 512k Context)"`, **512k context**, 19.6 tok/s)
- `omlx/DeepSeek-V4-Flash-0731-AWQ`
- `omlx/DeepSeek-V4-Flash-0731-MXFP4-MLX`
- `omlx/Qwen3-Coder-Next-MLX-9bit` (displayed as `"[HDD] Qwen3-Coder-Next-MLX-9bit"`, archived on external USB-C HDD)

### Mixture of Experts (MoE) & Archive (`[HDD]`)
- `omlx/MiniMax-M3-ConfigI-MLX` (vision enabled)
- `omlx/Qwen3.5-122B-A10B-MLX-9bit` (displayed as `"[HDD] Qwen3.5-122B-A10B-MLX-9bit"`, archived on external USB-C HDD)
- `omlx/Qwen3-235B-A22B-MLX-4bit` (displayed as `"[HDD] Qwen3-235B-A22B-MLX-4bit"`, archived on external USB-C HDD)

### General & 27B (Splash 1.6.1-m1.1 vs Native MLX DFlash2)
- `omlx/Qwen3.8-27B-Uncensored-MLX-SPLASH` (displayed as `"Qwen3.8-27B-Uncensored (SPLASH)"`, **default**, vision enabled, `69.2 tok/s` avg in `omlx_bench.sh`)
- `omlx/Qwen3.8-27B-Uncensored-MLX` (displayed as `"Qwen3.8-27B-Uncensored (MLX DFlash2)"`)
- `omlx/Qwen3.8-27B-FP16_huzar_v1-SPLASH` (displayed as `"[HDD] Qwen3.8-27B-FP16_huzar_v1 (SPLASH)"`, `[HDD]`)
- `omlx/Qwen3.8-27B-FP16_huzar_v1` (displayed as `"Qwen3.8-27B-FP16_huzar_v1 (MLX DFlash2)"`)

SPLASH variants are separate model IDs exposed by the oMLX server (`external_engine: "splash"` running `Splash 1.6.1-m1.1` with DFlash2 speculative decoding), so they are configured as their own entries rather than OpenCode `variants` (which cannot change the `modelID` sent to the server). The helper drafter checkpoint `Qwen3.8-27B-DFlash2` (`DFlash2DraftModel`) is excluded from the chat picker.

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

### 2026-11-21 — oMLX `0.8.2` (build `3849`), Splash `1.7.0-m1` & Add `Hy4-flash-oQ4h-mtp` + `Inkling-2-Mini-oQ4e-mtp` (**28 Models**)
- Upgraded Splash to **`1.7.0-m1`** and verified `oMLX 0.8.2` (`87` fields in `ModelSettings`, LoRA adapter support).
- Added **`Hy4-flash-oQ4h-mtp`** (`123.6 GB` RAM, VLM, `oQ4h`, Lightning MTP `81.9%`, `35.0 tok/s` avg / `42.9 tok/s` peak) and **`Inkling-2-Mini-oQ4e-mtp`** (`17.5 GB` RAM, VLM, DSpark `76.0%`, PASS `170.9s` / `38.2 tok/s` avg / `4778` chars in `omlx_bench.sh`) to `opencode.json` and `opencode.json.remote`.

### 2026-11-20 — oMLX `0.8.0` (build `3789`) & Splash `1.6.1-m1.1` + `MiniMax-M3` Fix Verified
- Zaktualizowano Splash do **`1.6.1-m1.1`** (`/opt/homebrew/bin/splash-m1`) oraz zsynchronizowano opis serwera ze stabilnym **`oMLX 0.8.0`** (`integrations.splash_activations: "bf16"`).
- Potwierdzono naprawę `jundot/omlx#5233` (PR `#5239`): `MiniMax-M3-ConfigI-MLX` przechodzi `omlx_bench.sh` na promptach >4k (`max_prompt=16376`, `25.1 tok/s` decode przez `masked sparse path`, `0` błędów Metal).

### 2026-11-12 — dodano 7 nowych modeli MLX/MTP (Gemma 4, GLM 5.3 Flash, Qwen 3.8 oQ6e, Bonsai-2, Nanbeige 4.3) oraz archiwizację `[HDD]`
- Rozszerzono listę modeli w `opencode.json` i `opencode.json.remote` z 20 do **27 modeli**:
  - `Qwen3.8-Flash-Next-oQ6e-mtp` (6-bit iMatrix, Lightning MTP, VLM, FP16 recast: **60.6–63.9 tok/s**, 141.1 GB RAM)
  - `GLM-5.3-Flash-oQ4e-mtp` (320B MoE / 18B active, Lightning MTP, VLM, 100% w RAM po podniesieniu limitu Metal do 186 GB: **25.9–29.9 tok/s**, TTFT **0.17s**, 173.9 GB RAM)
  - `GLM-5.3-Flash-oQ2e-mtp` (320B MoE / 18B active, 2-bit mixed iMatrix + 4-bit router/shared/MTP, VLM: **32.1–37.6 tok/s**, 102.9 GB RAM)
  - `gemma-4-12B-it-oQ8e-mtp` (Any-to-Any VLM/Audio, 8-bit, Lightning MTP, FP16 recast: **51.4–68.4 tok/s**, 13.0 GB RAM)
  - `gemma-4-31B-it-oQ6e-mtp` (Dense 31B VLM, 6-bit, Lightning MTP `gemma4_verify_kernel`, FP16 recast: **32.4–34.0 tok/s**, 26.3 GB RAM)
  - `Ternary-Bonsai-2-27B-MTPLX-Optimized-Speed` (2-bit Hadamard 27B VLM: **31.4 tok/s**, tylko **8.23 GB RAM**)
  - `Nanbeige4.3-33B-MLX-6bit` (Dense 33B Agent/Coder, okno **512k tokenów**: **19.6 tok/s**, 25.2 GB RAM)
- Oznaczono prefiksem **`[HDD]`** 3 starsze modele przeniesione na dysk zewnętrzny `/Volumes/Seagate_Expansion_Drive/LLM` (`[HDD] Qwen3-235B-A22B-MLX-4bit`, `[HDD] Qwen3-Coder-Next-MLX-9bit`, `[HDD] Qwen3.5-122B-A10B-MLX-9bit`), odzyskując 328 GiB na wewnętrznym SSD.

### 2026-11-06 — wyróżnienie Qwen 3.8 Flash na szczycie listy i oczyszczenie schematu OpenCode 2.2.5
- Przesunięto najszybsze modele `Qwen3.8-Flash-Next-oQ5e-mtp` (~62.6–97 tok/s z Lightning MTP + FP16 recast) oraz `Qwen3.8-Flash-Next-MTPLX-Optimized-Speed` (~73.5 tok/s z tablicą N-gramów + FP16 recast) na pierwsze dwie pozycje w `opencode.json` i `opencode.json.remote`.
- Rozbudowano etykiety wyświetlane w pickerze `/models` o frazę `(Qwen 3.8 Flash · ...)`, dzięki czemu wyszukiwanie zarówno ze spacją (`Qwen 3.8`), jak i bez (`Qwen3.8`) natychmiast filtruje oba modele.
- Usunięto wycofane w OpenCode 2.2.5 pole `"attachment": true` (obsługiwane natywnie przez `"modalities": { "input": ["text", "image"] }`), eliminując ostrzeżenia `configuration normalization diagnostic` w logach.

### 2026-11-04 — oMLX 0.7.4.dev1 & Splash 1.5.0-m1, VLM unlock, drafter cleanup
- Usunięto z listy wyboru OpenCode techniczny model pomocniczy `Qwen3.8-27B-DFlash2` (`DFlash2DraftModel`), pozostawiając dokładnie **20 modeli użytkowych**.
- Uporządkowano etykiety modeli 27B w obu wariantach (`opencode.json` i `opencode.json.remote`):
  - `Qwen3.8-27B-Uncensored (SPLASH)` vs `Qwen3.8-27B-Uncensored (MLX DFlash2)`
  - `Qwen3.8-27B-FP16_huzar_v1 (SPLASH)` vs `Qwen3.8-27B-FP16_huzar_v1 (MLX DFlash2)`
  - `Qwen3.8-27B-Coder390 (VLM · MTP)` (zamiast mylącego `(DFlash2)`)
- Odblokowano obsługę obrazów (`"input": ["text", "image"]`, `"attachment": true`) dla wszystkich modeli VLM (`Qwen3.8-27B-Coder390`, `Qwen3.8-Flash-Next-oQ5e-mtp`, `Qwen3.8-Flash-Next-MTPLX-Optimized-Speed`, `MiniMax-M3-ConfigI-MLX`, `Qwen3.8-27B-Uncensored-MLX-SPLASH`, obok `UI-Venus-1.5-30B`, `Unlimited-OCR-8bit` i `olmOCR-2-7B-1025-8bit`).

### 2026-10-28 — dodano model `flint-oQ8e`
- Dodano nowy model `flint-oQ8e` (kontekst 262 144 tokenów) do konfiguracji lokalnej (`opencode.json`) oraz zdalnej (`opencode.json.remote`).
- Lista skonfigurowanych modeli obejmuje obecnie 21 pozycji dostępnych na serwerze oMLX.

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

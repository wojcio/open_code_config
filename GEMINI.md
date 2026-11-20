# Antigravity Project Instructions: OpenCode Global Configuration

## 📌 Cel i Przegląd Projektu
Globalne repozytorium konfiguracji dla narzędzia OpenCode CLI (`~/.config/opencode`).
Zarządza definicjami providerów modeli AI, w szczególności lokalnego serwera inferencyjnego oMLX na Apple Silicon (Mac Studio M2 Ultra 192GB).

## 🏗️ Architektura i Decyzje
- **Główny provider:** `oMLX` (`oMLX 0.8.0` + `Splash 1.6.1-m1.1`, sufit pamięci 186 GB) pod adresem `http://127.0.0.1:8000/v1` (oraz `omlxremote` pod `http://100.123.37.101:8000/v1`) za pośrednictwem biblioteki `@ai-sdk/openai-compatible`.
- **Obsługa modeli:** Pełny zestaw **26 skonfigurowanych modeli** (1:1 z `model_settings.json` w oMLX; pomocniczy drafter `Qwen3.8-27B-DFlash2` wykluczony z listy czatowej):
  - **Szybkie modele hybrydowe Qwen 3.8 Flash (VLM + FP16 Recast + MTP):** `Qwen3.8-Flash-Next-oQ5e-mtp` (67.5–73.0 tok/s), `Qwen3.8-Flash-Next-oQ6e-mtp` (6-bit, 60.6–63.9 tok/s), `Qwen3.8-Flash-Next-MTPLX-Optimized-Speed` (~73.5 tok/s)
  - **Model GLM-5.3-Flash 320B MoE (18B akt./tok, VLM + Lightning MTP):** `GLM-5.3-Flash-oQ4e-mtp` (173.9 GB RAM, 25.9–29.9 tok/s, TTFT 0.17s; wariant `oQ2e-mtp` usunięty 13.11.2026 po testach porównawczych z Qwen 3.8 Flash)
  - **Szybkie modele Gemma 4, Bonsai-2 i Nanbeige 4.3:** `gemma-4-12B-it-oQ8e-mtp` (VLM/Audio, 51.4–68.4 tok/s, 13 GB), `gemma-4-31B-it-oQ6e-mtp` (VLM, 32.4–34.0 tok/s, 26.3 GB), `Ternary-Bonsai-2-27B-MTPLX-Optimized-Speed` (VLM, 2-bit Hadamard, 8.2 GB, 31.4 tok/s), `Nanbeige4.3-33B-MLX-6bit` (okno **512k**, 19.6 tok/s, 25.2 GB)
  - **Dedykowane do programowania i agentów:** `flint-oQ8e` (MoE 39.8B, 2.5B akt./tok, do 74.1 tok/s), `Qwen3.8-27B-Coder390-...-MTP-DFlash2` (`Qwen3.8-27B-Coder390 (VLM · MTP)`, VLM + Lightning MTP)
  - **Zaawansowany reasoning i architektura:** `DeepSeek-V4-Flash-0731-MXFP4-MLX`, `DeepSeek-V4-Flash-0731-AWQ`
  - **Modele 27B (pary `SPLASH` 1.6.1-m1.1 vs natywny `MLX DFlash2`):**
    - `Qwen3.8-27B-Uncensored-MLX-SPLASH` (`Qwen3.8-27B-Uncensored (SPLASH)`, domyślny model OpenCode, VLM; `69.2 tok/s` śr. w `omlx_bench.sh`) oraz `Qwen3.8-27B-Uncensored-MLX` (`Qwen3.8-27B-Uncensored (MLX DFlash2)`)
    - `[HDD] Qwen3.8-27B-FP16_huzar_v1-SPLASH` (`[HDD] Qwen3.8-27B-FP16_huzar_v1 (SPLASH)`, przeniesiony na archiwum HDD) oraz `Qwen3.8-27B-FP16_huzar_v1` (`Qwen3.8-27B-FP16_huzar_v1 (MLX DFlash2)`)
  - **Archiwum na zewnętrznym dysku HDD (`[HDD]`):** `Qwen3.8-27B-FP16_huzar_v1-SPLASH`, `Qwen3.5-122B-A10B-MLX-9bit`, `Qwen3-Coder-Next-MLX-9bit`, `Qwen3-235B-A22B-MLX-4bit` (podlinkowane z `/Volumes/Seagate_Expansion_Drive/LLM`)
  - **Pozostałe modele i Vision / OCR:** `MiniMax-M3-ConfigI-MLX` (VLM), `Step-3.5-Flash-4bit`, `Laguna-S-2.1-MLX-8bit`, `Ornith-1.5-35B-A3B-Abliterated-MLX-BF16`, `UI-Venus-1.5-30B-A3B-qx86-hi-mlx`, `olmOCR-2-7B-1025-8bit`, `Unlimited-OCR-8bit`
- **Pojedynczy plik konfiguracyjny:** Użycie wyłącznie `opencode.json` (wykluczenie `opencode.jsonc`), aby zapobiec konfliktom i niepożądanemu blokowaniu providerów.

## 📁 Struktura Plików
| Ścieżka | Rola / Przeznaczenie |
|---|---|
| `opencode.json` | Główna konfiguracja lokalna (baseURL: http://127.0.0.1:8000/v1) |
| `opencode.json.remote` | Konfiguracja dla maszyn zdalnych/laptopa (baseURL: http://100.123.37.101:8000/v1) |
| `.gitignore` | Ignorowanie lokalnych zależności (node_modules, pliki lock) oraz kopii zapasowych (*.bak) |
| `README.md` | Dokumentacja użycia i spis modeli |
| `GEMINI.md` | Kontekst projektu, architektura i stan dla Antigravity |

## 🔄 Ostatnie Zmiany i Stan Projektu
- **20 Listopada 2026 (oMLX `0.8.0` build `3789`, Splash `1.6.1-m1.1` i naprawa `MiniMax-M3` `#5233`):**
  - Zaktualizowano `/opt/homebrew/bin/splash-m1` do **`1.6.1-m1.1`** oraz zaktualizowano opis serwera do stabilnego **`oMLX 0.8.0`**.
  - Zweryfikowano naprawę `jundot/omlx#5233` (PR `#5239`): `MiniMax-M3-ConfigI-MLX` przechodzi `omlx_bench.sh` przy `max_prompt=16376` (`25.1 tok/s` decode, `masked sparse path`, brak błędów Metal), a następnie przeniesiono model (`155.61 GiB`) na zewnętrzny dysk Seagate HDD ze znacznikiem `[HDD]` w `opencode.json` i `opencode.json.remote`.
  - Zaktualizowano binarkę OpenCode z `v2.2.7` do **`v2.3.2`** (`opencode upgrade`, wszystkie 26 modeli `omlx/*` ładują się bez ostrzeżeń).
- **13 Listopada 2026 (Splash 1.6.1-m1, `[HDD]` Huzar-SPLASH i usunięcie `GLM-5.3-Flash-oQ2e-mtp` — 26 modeli):**
  - Zaktualizowano silnik Splash do **`1.6.1-m1`** (`/opt/homebrew/bin/splash-m1`) i zweryfikowano `Qwen3.8-27B-Uncensored-MLX-SPLASH` w `omlx_bench.sh` (`PASS` w `197.2s`, śr. **`69.2 tok/s`**, szczyt **`91.2 tok/s`**).
  - Oznaczono `Qwen3.8-27B-FP16_huzar_v1-SPLASH` jako `[HDD]` w `opencode.json` i `opencode.json.remote` (`[HDD] Qwen3.8-27B-FP16_huzar_v1 (SPLASH)` / `[omlxremote] [HDD] ...`).
  - Usunięto `GLM-5.3-Flash-oQ2e-mtp` z `opencode.json` i `opencode.json.remote` (27 $\rightarrow$ **26** modeli), uznając zdecydowaną przewagę `Qwen3.8-Flash-Next-oQ5e-mtp`.
  - Odnotowano serwer `oMLX 0.8.0rc1` oraz zgłoszenie błędu `MiniMax-M3-ConfigI-MLX` (`https://github.com/jundot/omlx/issues/5233`, >4k kontekstu na M2 Ultra).
- **13 Listopada 2026 (Aktualizacja `GLM-5.3-Flash-oQ2e-mtp` do 512k kontekstu):**
  - Podniesiono `limit.context` dla `GLM-5.3-Flash-oQ2e-mtp` z `262144` (256k) do **`524288` (512k)** w `opencode.json` i `opencode.json.remote` (z zachowaniem `output: 32768` oraz `262144` dla `oQ4e-mtp`). Po stronie serwera oMLX skorygowano również `temperature` dla `oQ2e-mtp` do `1.0`.
- **12 Listopada 2026 (Wdrożenie 7 nowych modeli MLX/MTP oraz archiwizacja `[HDD]` 3 starszych modeli Qwen):**
  - Dodano 7 nowych modeli do `opencode.json` i `opencode.json.remote` (łącznie **27 modeli**): `Qwen3.8-Flash-Next-oQ6e-mtp`, `GLM-5.3-Flash-oQ4e-mtp`, `GLM-5.3-Flash-oQ2e-mtp`, `gemma-4-12B-it-oQ8e-mtp`, `gemma-4-31B-it-oQ6e-mtp`, `Ternary-Bonsai-2-27B-MTPLX-Optimized-Speed` oraz `Nanbeige4.3-33B-MLX-6bit` (512k kontekstu).
  - Dodano oznaczenie `[HDD]` dla 3 starszych modeli Qwen (`Qwen3-235B-A22B-MLX-4bit`, `Qwen3-Coder-Next-MLX-9bit`, `Qwen3.5-122B-A10B-MLX-9bit`) przeniesionych na zewnętrzny dysk `/Volumes/Seagate_Expansion_Drive/LLM` (odzyskano 328 GiB na wewnętrznym SSD).
  - Uzupełniono listę `favorite` w `~/.local/state/opencode/model.json`.
- **6–7 Listopada 2026 (Splash 1.6.0-m1, wyróżnienie Qwen 3.8 Flash i chrzest bojowy 225k kontekstu w OpenCode 2.2.5):**
  - Zaktualizowano silnik `/opt/homebrew/bin/splash-m1` do **`Splash 1.6.0-m1`** (zarządca pamięci `One memory ledger`, optymalizacja `Idle KV` #644, ANE FFN split `share 0.26` dla $\ge 551$ wierszy: 54.1 ms vs 70.0 ms GPU, generowanie `Qwen3.8-27B-Uncensored-MLX-SPLASH` z prędkością **50.2–62.0 tok/s** i TTFT **0.39s**).
  - Przesunięto oba najszybsze modele `Qwen3.8-Flash-Next-oQ5e-mtp` oraz `Qwen3.8-Flash-Next-MTPLX-Optimized-Speed` na sam szczyt listy `"models"` w `opencode.json` oraz `opencode.json.remote` i dodano oba modele (wraz z `DeepSeek-V4-Flash-0731-MXFP4-MLX` i `DeepSeek-V4-Flash-0731-AWQ`) do `favorite`.
  - Dodano czytelne etykiety wyświetlane (`"Qwen3.8-Flash-Next-oQ5e-mtp (Qwen 3.8 Flash · Lightning MTP · VLM)"` oraz `"Qwen3.8-Flash-Next-MTPLX-Optimized-Speed (Qwen 3.8 Flash · N-gram · VLM)"`), umożliwiające filtrowanie w pickerze `/models` również po wpisaniu `Qwen 3.8` ze spacją.
  - Usunięto przestarzałe pole `"attachment": true` (zastąpione w OpenCode 2.2.5 przez `"modalities": { "input": ["text", "image"] }`), eliminując ostrzeżenia normalizacji konfiguracji przy starcie.
  - Potwierdzono w długiej sesji produkcyjnej OpenCode (185+ tur narzędziowych, kontekst rosnący aż do **225 280 tokenów**), że `Qwen3.8-Flash-Next-oQ5e-mtp` z `"activation_fp16_enabled": true` utrzymuje **42.3 – 58.8 tok/s** (w tym **51.9 tok/s** przy 218.8k kontekstu i TTFT 1.38–3.29s dzięki Paged Prefix Cache).
- **4 Listopada 2026 (oMLX 0.7.4.dev1 & Splash 1.5.0-m1):**
  - Usunięto techniczny model pomocniczy `Qwen3.8-27B-DFlash2` (`DFlash2DraftModel`) z `opencode.json` i `opencode.json.remote` (dokładnie 20 modeli użytkowych).
  - Rozróżniono czytelnie etykiety par modeli 27B: `(SPLASH)` vs `(MLX DFlash2)`, a `Coder390` oznaczono zgodnie z faktycznym silnikiem jako `Qwen3.8-27B-Coder390 (VLM · MTP)`.
  - Odblokowano modalność `"image"` oraz `"attachment": true` dla wszystkich 8 modeli VLM (`Coder390`, `Flash-Next-oQ5e-mtp`, `Flash-Next-MTPLX-Optimized-Speed`, `MiniMax-M3-ConfigI-MLX`, `Qwen3.8-27B-Uncensored-MLX-SPLASH`, `UI-Venus-1.5-30B`, `Unlimited-OCR-8bit`, `olmOCR-2-7B-1025-8bit`).
- Dodano model `flint-oQ8e` (kontekst 262k, output 32k) do konfiguracji lokalnej (`opencode.json`) i zdalnej (`opencode.json.remote`).
- Przeprowadzono chrzest bojowy i test obciążeniowy modelu `omlx/DeepSeek-V4-Flash-0731-MXFP4-MLX` na kontekście 171k tokenów (udokumentowano w sekcji poniżej oraz w README.md).
- Dodano `opencode.json.remote` z adresem `http://100.123.37.101:8000/v1` (Tailscale/VPN) umożliwiający uruchamianie OpenCode na laptopie ze zdalnym serwerem oMLX na Mac Studio.
- Zainicjalizowano repozytorium Git i podłączono remote `git@github.com:wojcio/open_code_config.git`.

## 🧪 Chrzest Bojowy i Use Case: DeepSeek-V4-Flash-0731-MXFP4-MLX (171k tokenów)
- **Kontekst testu:** Rzeczywiste zapytanie agenta OpenCode w rozbudowanej sesji roboczej (prompt wejściowy: **171 334 tokeny**).
- **Sprzęt:** Mac Studio M2 Ultra 192GB Unified Memory.
- **Wyniki testu:**
  - **VRAM / Unified Memory:** Szczytowe zużycie **~175.2 GB** bez ryzyka OOM.
  - **Prefill (Prompt Processing):** GPU Metal 95–100% obciążenia przez **691.53 s** (~11.5 min) z przepustowością ~248 tok/s.
  - **Generacja (Decoding):** Wywołanie narzędzia (`tool_calls`) przy 476 tokenach (22.5 tok/s) z aktywacją wieloetapowego MTP (accept rate 86.7%, 3.11 tok/cykl).
  - **Wydajność Paged Prefix Cache:** Druga tura odzyskała **171 332 tokeny** z cache w **868 ms**, skracając czas oczekiwania na pierwszy token z 691 s do zaledwie 18 s. Generacja 1 432 tokenów zajęła 86.9 s (21.1 tok/s, `stop`).
- **Wnioski:** Konfiguracja jest w 100% stabilna przy pracy z gigantycznym oknem kontekstowym. Przy nowych tematach zaleca się reset sesji OpenCode, aby uniknąć 11-minutowego prefillu początkowego.


# Antigravity Project Instructions: OpenCode Global Configuration

## 📌 Cel i Przegląd Projektu
Globalne repozytorium konfiguracji dla narzędzia OpenCode CLI (`~/.config/opencode`).
Zarządza definicjami providerów modeli AI, w szczególności lokalnego serwera inferencyjnego oMLX na Apple Silicon (Mac Studio M2 Ultra 192GB).

## 🏗️ Architektura i Decyzje
- **Główny provider:** `oMLX` (`oMLX 0.7.4.dev1` + `Splash 1.5.0-m1`) pod adresem `http://127.0.0.1:8000/v1` (oraz `omlxremote` pod `http://100.123.37.101:8000/v1`) za pośrednictwem biblioteki `@ai-sdk/openai-compatible`.
- **Obsługa modeli:** Pełny zestaw **20 skonfigurowanych modeli** (1:1 z `model_settings.json` w oMLX; pomocniczy drafter `Qwen3.8-27B-DFlash2` wykluczony z listy czatowej):
  - **Dedykowane do programowania i agentów:** `flint-oQ8e` (MoE 39.8B, 2.5B akt./tok), `Qwen3-Coder-Next-MLX-9bit`, `Qwen3.8-27B-Coder390-...-MTP-DFlash2` (`Qwen3.8-27B-Coder390 (VLM · MTP)`, VLM + Lightning MTP)
  - **Zaawansowany reasoning i architektura:** `DeepSeek-V4-Flash-0731-MXFP4-MLX`, `DeepSeek-V4-Flash-0731-AWQ`
  - **Duże modele MoE:** `Qwen3.5-122B-A10B-MLX-9bit`, `Qwen3-235B-A22B-MLX-4bit`, `MiniMax-M3-ConfigI-MLX` (VLM)
  - **Modele 27B (pary `SPLASH` 1.5.0-m1 vs natywny `MLX DFlash2`):**
    - `Qwen3.8-27B-Uncensored-MLX-SPLASH` (`Qwen3.8-27B-Uncensored (SPLASH)`, domyślny model OpenCode, VLM) oraz `Qwen3.8-27B-Uncensored-MLX` (`Qwen3.8-27B-Uncensored (MLX DFlash2)`)
    - `Qwen3.8-27B-FP16_huzar_v1-SPLASH` (`Qwen3.8-27B-FP16_huzar_v1 (SPLASH)`) oraz `Qwen3.8-27B-FP16_huzar_v1` (`Qwen3.8-27B-FP16_huzar_v1 (MLX DFlash2)`)
  - **Szybkie modele hybrydowe i VLM:** `Qwen3.8-Flash-Next-oQ5e-mtp` (VLM, FP16 recast + MTP), `Qwen3.8-Flash-Next-MTPLX-Optimized-Speed` (VLM, FP16 recast), `Step-3.5-Flash-4bit`, `Laguna-S-2.1-MLX-8bit`, `Ornith-1.5-35B-A3B-Abliterated-MLX-BF16`
  - **Specjalistyczne Vision / OCR:** `UI-Venus-1.5-30B-A3B-qx86-hi-mlx`, `olmOCR-2-7B-1025-8bit`, `Unlimited-OCR-8bit`
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
- **6 Listopada 2026 (Wyróżnienie Qwen 3.8 Flash i kompatybilność z OpenCode 2.2.5):**
  - Przesunięto oba najszybsze modele `Qwen3.8-Flash-Next-oQ5e-mtp` oraz `Qwen3.8-Flash-Next-MTPLX-Optimized-Speed` na sam szczyt listy `"models"` w `opencode.json` oraz `opencode.json.remote`.
  - Dodano czytelne etykiety wyświetlane (`"Qwen3.8-Flash-Next-oQ5e-mtp (Qwen 3.8 Flash · Lightning MTP · VLM)"` oraz `"Qwen3.8-Flash-Next-MTPLX-Optimized-Speed (Qwen 3.8 Flash · N-gram · VLM)"`), umożliwiające filtrowanie w pickerze `/models` również po wpisaniu `Qwen 3.8` ze spacją.
  - Usunięto przestarzałe pole `"attachment": true` (zastąpione w OpenCode 2.2.5 przez `"modalities": { "input": ["text", "image"] }`), eliminując ostrzeżenia normalizacji konfiguracji przy starcie.
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


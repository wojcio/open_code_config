# Antigravity Project Instructions: OpenCode Global Configuration

## 📌 Cel i Przegląd Projektu
Globalne repozytorium konfiguracji dla narzędzia OpenCode CLI (`~/.config/opencode`).
Zarządza definicjami providerów modeli AI, w szczególności lokalnego serwera inferencyjnego oMLX na Apple Silicon (Mac Studio M2 Ultra 192GB).

## 🏗️ Architektura i Decyzje
- **Główny provider:** `oMLX` pod adresem `http://127.0.0.1:8000/v1` za pośrednictwem biblioteki `@ai-sdk/openai-compatible`.
- **Obsługa modeli:** Pełny zestaw 13 zoptymalizowanych modeli MLX:
  - Dedykowane do programowania: `Qwen3-Coder-Next-MLX-8bit`, `Qwen3-Coder-Next-MLX-9bit`
  - Zaawansowany reasoning i kod: `DeepSeek-V4-Flash-0731-AWQ`, `DeepSeek-V4-Flash-MLX-mixed-4_8bit`
  - Duże modele MoE: `Qwen3-235B-A22B-MLX-4bit`, `Qwen3.5-122B-A10B-MLX-9bit`
  - Wersje ogólne i nieskwantyzowane: `Qwen3.8-27B-Uncensored-MLX`, `Qwen3.8-27B-FP16_huzar_v1`, `Qwen3.8-27B-MTPLX-Optimized-Quality-FP16`
  - Szybkie i multimodalne: `Qwen3.8-Flash-Next-MLX-6bit` (vision/obrazy), `Step-3.5-Flash-4bit`
  - Polskie modele konwersacyjne: `Bielik-11B-v3.0-Instruct-MLX-8bit`, `Bielik-11B-v3.0-Huzar-MLX-8bit-v1`
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
- Zainicjalizowano repozytorium Git i podłączono remote `git@github.com:wojcio/open_code_config.git`.
- Skonfigurowano i przetestowano 13 modeli na serwerze oMLX.
- Dodano `opencode.json.remote` z adresem `http://100.123.37.101:8000/v1` (Tailscale/VPN) umożliwiający uruchamianie OpenCode na laptopie ze zdalnym serwerem oMLX na Mac Studio.
- Przeprowadzono chrzest bojowy i test obciążeniowy modelu `omlx/DeepSeek-V4-Flash-0731-MXFP4-MLX` na kontekście 171k tokenów (udokumentowano w sekcji poniżej oraz w README.md).

## 🧪 Chrzest Bojowy i Use Case: DeepSeek-V4-Flash-0731-MXFP4-MLX (171k tokenów)
- **Kontekst testu:** Rzeczywiste zapytanie agenta OpenCode w rozbudowanej sesji roboczej (prompt wejściowy: **171 334 tokeny**).
- **Sprzęt:** Mac Studio M2 Ultra 192GB Unified Memory.
- **Wyniki testu:**
  - **VRAM / Unified Memory:** Szczytowe zużycie **~175.2 GB** bez ryzyka OOM.
  - **Prefill (Prompt Processing):** GPU Metal 95–100% obciążenia przez **691.53 s** (~11.5 min) z przepustowością ~248 tok/s.
  - **Generacja (Decoding):** Wywołanie narzędzia (`tool_calls`) przy 476 tokenach (22.5 tok/s) z aktywacją wieloetapowego MTP (accept rate 86.7%, 3.11 tok/cykl).
  - **Wydajność Paged Prefix Cache:** Druga tura odzyskała **171 332 tokeny** z cache w **868 ms**, skracając czas oczekiwania na pierwszy token z 691 s do zaledwie 18 s. Generacja 1 432 tokenów zajęła 86.9 s (21.1 tok/s, `stop`).
- **Wnioski:** Konfiguracja jest w 100% stabilna przy pracy z gigantycznym oknem kontekstowym. Przy nowych tematach zaleca się reset sesji OpenCode, aby uniknąć 11-minutowego prefillu początkowego.


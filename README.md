# OpenCode Configuration

Global configuration repository for [OpenCode](https://opencode.ai) with local [oMLX](https://github.com/vllm-project) inference server on Apple Silicon.

## Setup

Place `opencode.json` in your OpenCode configuration directory:
- **macOS/Linux**: `~/.config/opencode/opencode.json`

## Configured Models

All models connect to the local OpenAI-compatible endpoint at `http://127.0.0.1:8000/v1`:

### Coding & Reasoning
- `omlx/Qwen3-Coder-Next-MLX-8bit`
- `omlx/Qwen3-Coder-Next-MLX-9bit`
- `omlx/DeepSeek-V4-Flash-0731-AWQ`
- `omlx/DeepSeek-V4-Flash-MLX-mixed-4_8bit`

### Mixture of Experts (MoE)
- `omlx/Qwen3-235B-A22B-MLX-4bit`
- `omlx/Qwen3.5-122B-A10B-MLX-9bit`

### General & FP16
- `omlx/Qwen3.8-27B-Uncensored-MLX` (default)
- `omlx/Qwen3.8-27B-FP16_huzar_v1`
- `omlx/Qwen3.8-27B-MTPLX-Optimized-Quality-FP16`

### Flash & Multimodal
- `omlx/Qwen3.8-Flash-Next-MLX-6bit` (multimodal / vision)
- `omlx/Step-3.5-Flash-4bit`

### Polish Language
- `omlx/Bielik-11B-v3.0-Instruct-MLX-8bit`
- `omlx/Bielik-11B-v3.0-Huzar-MLX-8bit-v1`

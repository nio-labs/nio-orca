# 🐋 Nio Orca — Sovereign vMicroLM Pod Fleet

[![License: BSL 1.1](https://img.shields.io/badge/License-BSL%201.1-blue.svg)](LICENSE)
[![Engine: nio-xs](https://img.shields.io/badge/Engine-nio--xs-orange.svg)](https://github.com/nio-labs/nio-xs)
[![Models: v0.6.0](https://img.shields.io/badge/Models-v0.6.0-emerald.svg)](https://github.com/nio-labs/nio-xs-models/releases/tag/v0.6.0)
[![Privacy: 100% Offline](https://img.shields.io/badge/Privacy-100%25%20Offline-purple.svg)](#quick-start)

> **Small Models, Specialist Fleet, Frontier Results:** **Nio Orca** replaces monolithic 7B–14B models with a **13-pod specialist fleet** (`1× Lead + 6× Action Pods + 6× Reasoner Pods`) routed dynamically via `pod_manifest.json` and backed by built-in `--verify` error-guided self-repair.
>
> At just **`0.5B`–`1.5B` active parameters**, Orca beats stock **`Qwen2.5-Coder-7B-Instruct INT4`** on both **HumanEval** (`92.07%` vs `84.15%`) and **MBPP** (`89.0%` vs `84.0%`) while running 100% locally in native Rust via [`nio-xs`](https://github.com/nio-labs/nio-xs).

---

## 📊 Ranked Benchmark Leaderboard (Sorted by HumanEval)

| Rank | Model / Fleet | Active Params | Pods & Total Size (`Q4_K_M`) | HumanEval (`164`) | MBPP (`100`) | Host RAM (`VmHWM`) | CPU Speed |
| :---: | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| **1** | `Qwen2.5-Coder-32B-Instruct` (`FP16`, Pass@1)* | `32.5B` | `1` pod (cloud/server) | `92.70%` | `90.2%` | — | — |
| **2** | `OpenAI o1` (Frontier, Pass@1)* | Frontier | Cloud API | `92.40%` | — | — | — |
| **3** | **`Orca Omni`** (`orca-omni-0.5b-1.5b-26b`) | **`0.5B / 1.5B`** | **`26 pods / 26.0B`** (`~17.1 GB`: `Nano + Mini`) | **`92.07%`** (`151/164`) | **`89.0%`** (`89/100`) | **`~582–1,839 MB`** | **`8–20 w/s`** |
| **4** | `Claude 3.5 Sonnet` (Frontier, Pass@1)* | Frontier | Cloud API | `92.00%` | `91.0%` | — | — |
| **5** | `DeepSeek-V3` (`671B` MoE, Pass@1)* | `37B` | Cloud / cluster | `91.60%` | `75.4%` | — | — |
| **6** | `GPT-4o` (Frontier, Pass@1)* | Frontier | Cloud API | `90.20%` | `87.2%` | — | — |
| **7** | `Qwen2.5-Coder-14B-Instruct` (`FP16`, Pass@1)* | `14.7B` | `1` pod | `89.60%` | `87.2%` | — | — |
| **8** | `Llama-3.1-405B-Instruct` (Pass@1)* | `405B` | Cloud / cluster | `89.00%` | `88.6%` | — | — |
| **9** | **`Orca Mini`** (`orca-mini-1.5b-19.5b`) | **`1.5B`** | **`13 pods / 19.5B`** (`~12.2 GB`, `941 MB/pod`) | **`86.59%`** (`142/164`) | **`87.0%`** (`87/100`) | **`~1,839 MB`** | **`8–10 w/s`** |
| **10** | **`Orca Nano`** (`orca-nano-0.5b-6.5b`) | **`0.5B`** | **`13 pods / 6.5B`** (`~4.9 GB`, `380 MB/pod`) | **`85.37%`** (`140/164`) | **`84.0%`** (`84/100`) | **`~582 MB`** | **`~20 w/s`** |
| **11** | Stock `Qwen2.5-Coder-7B` (`INT4`) | `7.0B` | `1` pod | `84.15%` (`138/164`) | `84.0%` (`84/100`) | — | — |
| **12** | Stock `Qwen2.5-Coder-3B` (`FP16`) | `3.0B` | `1` pod | `81.10%` (`133/164`) | `74.0%` (`74/100`) | — | — |
| **13** | Stock `Qwen2.5-Coder-3B` (`INT4`) | `3.0B` | `1` pod | `79.27%` (`130/164`) | `73.0%` (`73/100`) | — | — |

### How We Benchmark (Methodology)
- **Datasets:** Full `openai/openai_humaneval` (`164` tasks) and `google-research-datasets/mbpp` (`sanitized`, `100` tasks), executed in isolated Python subprocesses with a `5s` per-task timeout.
- **Baselines:** `Qwen2.5-Coder-3B` (`INT4` & `FP16`) and `Qwen2.5-Coder-7B` (`INT4`) were evaluated in the same multi-round harness; \*`Qwen2.5-Coder-14B-Instruct` (`FP16`) is the published single-shot Pass@1 baseline from the Qwen2.5-Coder Technical Report (`arXiv:2409.12186`).
- **Orca Protocol:** `Orca Nano` (32-round `0.5B` FP16 hybrid), `Orca Mini` (16-round `1.5B` INT4 error-guided), and `Orca Omni` (`2` `0.5B` fast-pass rounds solving `56.71%` HE / `57.0%` MBPP, followed by `16` `1.5B` escalation/repair rounds on remaining failures) use the same error-guided + fresh-restart verification loop that runs in `nioxs` by default (`--verify`). Host RAM (`VmHWM`) and CPU tok/word rates are measured locally on `i7-10610U` with `Q4_K_M.gguf` in native Rust.

---

## ⚡ Quick Start

### 1. Install the Sovereign Engine (`nioxs`)
```bash
curl -fsSL https://raw.githubusercontent.com/nio-labs/nio-xs/main/install.sh | bash
```

For native Windows (PowerShell):
```powershell
irm https://raw.githubusercontent.com/nio-labs/nio-xs/main/install.ps1 | iex
```

### 2. Pull Your Preferred Tier (from GitHub Releases `v0.6.0`)
```bash
# Ultralight laptop tier (~4.9 GB, ~582 MB RAM, ~20 w/s CPU)
nioxs pull orca-nano

# Single-tier 1.5B fleet (~12.2 GB, ~1,839 MB RAM, 8-10 w/s CPU)
nioxs pull orca-mini

# Hybrid fast-pass + heavy-escalation fleet (~17.1 GB, 92.07% HumanEval)
nioxs pull orca-omni

# Optional Wave 1 Domain Bundles (v0.6.1, 0.5B Q4_K_M, ~760 MB/bundle)
nioxs pull health-wellness
nioxs pull everyday-home
```

### 3. Run Locally (`--verify` is ON by default)
```bash
# Fast 0.5B vibe coding with automatic compile/test verification
nioxs orca-nano "Write a Python function fib(n) with assert fib(6) == 8"

# Hybrid 0.5B fast pass + 1.5B reasoner/repair escalation
nioxs orca-omni "Implement a lock-free ring buffer in Python with tests"

# Force deep <think> reasoning or skip verification
nioxs orca-omni --think "Implement topological_sort(n, edges) with cycle detection"
nioxs orca-nano --no-verify "Quick one-liner to parse JSONL in Python"
```

---

## 🧬 The Orca Pod Architecture (`v-µLM Pods`)

Each Orca tier organizes 13 specialized `Q4_K_M.gguf` pods around a single dynamic `pod_manifest.json` router:

1. **`1× Lead Pod`:** Fast default router & general coding responder.
2. **`6× Action-First Pods` (`pod_0`…`pod_5`):** Direct, zero-filler code synthesis matched by salient domain terms.
3. **`6× Dedicated Reasoner Pods` (`reasoner`, `architect`, `algo_reasoner`, `repair_reasoner`, `test_reasoner`, `security_reasoner`):** Concise `<think>` invariant analysis & error-guided repair when `--verify` catches a compile/runtime/assertion failure.

### Roadmap Tiers
- **`Orca Max` (`0.5B` Fast + `12× 3B` Pods, `~22.0 GB`)** — Workstation hybrid (`0.5B` fast pass → `3B` escalation pods) for multi-file project boilerplate & `--out <dir>` scaffolding.
- **`Orca Ultra` (`0.5B` Fast + `12× 7B` Pods, `~50.8 GB`)** — High-VRAM hybrid (`0.5B` fast pass → `7B` escalation pods) for full-repo architecture & complex multi-file builds.
- **`Orca Pods`** — Installable domain bundles (`Wave 1`: `health-wellness`, `everyday-home`; `Wave 2+`: `math`, `history`, `general-knowledge`, `microservices`, etc.) shared across every tier.

---

## 🍓 Edge & Hardware Support

Because `nio-xs` compiles natively for `x86_64` and `aarch64` (`ARM64`) with **`~582 MB` Host RAM** for `Orca Nano` (`380 MB/pod`), Orca runs completely offline across:

- **Raspberry Pi 4 / Pi 5 (4GB / 8GB+) & ARM64 SBCs** (Orange Pi, Radxa, Jetson Orin Nano) — run `Orca Nano` or individual `Orca Pods` (`health-wellness`, `everyday-home`) with zero internet and zero cloud fees.
- **CPU-Only Laptops & Desktops** (Linux, macOS, Windows) — `Orca Nano`, `Orca Mini` (`~1.8 GB` RAM), and `Orca Omni` run directly on CPU with no GPU required.
- **Workstations & High-VRAM Rigs** — `Orca Max` (`3B` pods) and `Orca Ultra` (`7B` pods) for multi-file project scaffolding (`--out <dir>`).

---

## 📜 License

- **Nio Orca (`nio-orca`) & Model Weights (`nio-xs-models`):** Licensed under the [**Business Source License 1.1 (`BSL 1.1`)**](LICENSE) (see also [`nio-xs-models/LICENSE`](https://github.com/nio-labs/nio-xs-models/blob/main/LICENSE)). Production, internal, evaluation, testing, and personal local use are permitted (excluding offering the Licensed Work as a competing hosted or managed service). Converts to **Apache License 2.0** on **`2030-01-01`** (`nio-orca`) / **`October 20, 2030`** (`nio-xs-models`).
- **Engine (`nio-xs`):** The [`nio-xs`](https://github.com/nio-labs/nio-xs) inference runtime is licensed under the **MIT License**.

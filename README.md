# 🐋 Nio Orca — Sovereign vMicroLM Pod Fleet

[![License: BSL 1.1](https://img.shields.io/badge/License-BSL%201.1-blue.svg)](LICENSE)
[![Engine: nio-xs](https://img.shields.io/badge/Engine-nio--xs-orange.svg)](https://github.com/nio-labs/nio-xs)
[![Models: v0.6.0](https://img.shields.io/badge/Models-v0.6.0-emerald.svg)](https://github.com/nio-labs/nio-xs-models/releases/tag/v0.6.0)
[![Privacy: 100% Offline](https://img.shields.io/badge/Privacy-100%25%20Offline-purple.svg)](#quick-start)

> **Small Models, Specialist Fleet, Frontier Results:** **Nio Orca** replaces monolithic 7B–14B models with a **13-pod specialist fleet** (`1× Lead + 6× Action Pods + 6× Reasoner Pods`) routed dynamically via `pod_manifest.json` and backed by built-in `--verify` error-guided self-repair.
>
> At just **`0.5B`–`1.5B` active parameters**, Orca beats stock **`Qwen2.5-Coder-7B-Instruct INT4`** on both **HumanEval** (`92.07%` vs `84.15%`) and **MBPP** (`89.0%` vs `84.0%`) while running 100% locally in native Rust via [`nio-xs`](https://github.com/nio-labs/nio-xs).

---

## 📊 Benchmark Comparison vs. Stock `Qwen2.5-Coder-Instruct`

| Model / Fleet | Active Params | Pods & Total Size (`Q4_K_M`) | HumanEval (`164`) | MBPP (`100`) | Host RAM (`VmHWM`) | CPU Speed |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Stock `Qwen2.5-Coder-3B` (`INT4`) | `3.0B` | `1` pod | `79.27%` (`130/164`) | `73.0%` (`73/100`) | — | — |
| Stock `Qwen2.5-Coder-3B` (`FP16`) | `3.0B` | `1` pod | `81.10%` (`133/164`) | `74.0%` (`74/100`) | — | — |
| Stock `Qwen2.5-Coder-7B` (`INT4`) | `7.0B` | `1` pod | `84.15%` (`138/164`) | `84.0%` (`84/100`) | — | — |
| Stock `Qwen2.5-Coder-14B` (`FP16`, Pass@1)* | `14.7B` | `1` pod | `89.60%` | `87.2%` | — | — |
| **`Orca Nano`** (`0.5B × 13`) | **`0.5B`** | **`13 pods` (`~4.9 GB`, `380 MB/pod`)** | **`85.37%`** (`140/164`) | **`84.0%`** (`84/100`) | **`~582 MB`** | **`~20 w/s`** |
| **`Orca Mini`** (`1.5B × 13`) | **`1.5B`** | **`13 pods` (`~12.2 GB`, `941 MB/pod`)** | **`86.59%`** (`142/164`) | **`87.0%`** (`87/100`) | **`~1,839 MB`** | **`8–10 w/s`** |
| **`Orca Omni`** (`0.5B Fast → 1.5B Esc.`) | **`0.5B / 1.5B`** | **`26 pods` (`~17.1 GB`: `Nano + Mini`)** | **`92.07%`** (`151/164`) | **`89.0%`** (`89/100`) | **`~582–1,839 MB`** | **`8–20 w/s`** |

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
- **`Orca Max` (`0.5B` Fast + `12× 3B` Pods, `~22.0 GB`)** — Workstation hybrid (`0.5B` fast pass → `3B` escalation pods).
- **`Orca Ultra` (`0.5B` Fast + `12× 7B` Pods, `~50.8 GB`)** — High-VRAM hybrid (`0.5B` fast pass → `7B` escalation pods).
- **`Orca Pods`** — Installable domain bundles (`health-wellness`, `everyday-home`, `math`, `history`, `microservices`, etc.) shared across every tier.

# Other Qwen3.6-35B-A3B GGUFs

Setup offers two sizes of [Qwen3.6-35B-A3B](QWEN36.md) (Unsloth's UD-IQ4_XS and UD-IQ3_S). This page covers the other
GGUFs of the same model, and of a fine-tune of it, that people asked for: 37 files from three publishers, tested one at
a time against llama.cpp on the same file. 36 run correctly; one does not load (UD-Q8_K_XL, [below](#the-one-that-does-not-run)).
Setup installs each of them by name on this branch ([below](#installing-one)).
Back to [Qwen3.6-35B-A3B](QWEN36.md).

> **On this page:** [What was tested](#what-was-tested) · [How a pass is judged](#how-a-pass-is-judged) ·
> [Results](#results) · [What the engine needed](#what-the-engine-needed) ·
> [The one that does not run](#the-one-that-does-not-run) · [Installing one](#installing-one) · [Not tested](#not-tested)

## What was tested

| Publisher | Repository | Files |
| --- | --- | --- |
| Unsloth | [unsloth/Qwen3.6-35B-A3B-MTP-GGUF](https://huggingface.co/unsloth/Qwen3.6-35B-A3B-MTP-GGUF) @ `5bc3e23` | 19 |
| bartowski | [bartowski/Qwen_Qwen3.6-35B-A3B-GGUF](https://huggingface.co/bartowski/Qwen_Qwen3.6-35B-A3B-GGUF) @ `5c2410d` | 16 |
| Myric | [Myric/KAT-Coder-V2.5-Dev-MTP-APEX-GGUF](https://huggingface.co/Myric/KAT-Coder-V2.5-Dev-MTP-APEX-GGUF) @ `e4ae997` (`i-quality`, `i-quality-v2`) | 2 |

KAT-Coder-V2.5-Dev is a fine-tune of Qwen3.6-35B-A3B (`general.architecture` is `qwen35moe`, the same 40 layers of 256
experts). Every file carries the MTP draft layer, and every test ran with it.

Every file was checked against its SHA-256 from Hugging Face before the test. Measured on one PC: RTX 4070 Ti 12 GB
(driver 580.178), Ryzen 9 5950X, 64 GB DDR4, Linux. The engine was this branch built the way setup builds it
(`STRATA_MMQ_KQUANTS` off), CUDA. The reference is llama.cpp b11438 (`llama-server --fit on`) on the same file.

## How a pass is judged

Strata reads each of four prompts (three short chat prompts and one of 8,015 tokens), then is fed llama.cpp's own
greedy 40-token continuation one token at a time. At every position Strata's probabilities for the next token are
compared with llama.cpp's top 20: the KL divergence (llama.cpp || Strata), and whether the top token is the same.

**A file passes** when, on every prompt, the mean KL over the positions after the first is at most 0.05 and the top
token agrees at 90% or more of the positions. The first token after each prompt is reported on its own.

Why not "the generated tokens are identical": the first test of this series required that, and it failed 14 files
that turned out to be fine. Greedy text splits at the first near-tie (two tokens at 0.44 and 0.34, say) and everything
after it differs. The first token after the 8K prompt is the worst case: on the same file, llama.cpp's own CPU and GPU
runs differ there by a KL of up to 0.71. Files that run correctly measure 0.0003-0.009 on the teacher-forced test. Ten
files, and KAT-Coder's i-quality-v2, passed the stricter identical-tokens test as well; they are marked "identical"
below.

## Results

"Worst mean KL" is the highest of the four prompts' means; "top-1" counts the positions over all four prompts.
Speeds are on the 12 GB card, with the server's settings (`--expert-cache auto --prefill auto --kv int8 --spec 4`,
the draft layer from the same file): **write** is a 128-token answer to a short prompt, **8K read** is the
8,015-token prompt.

| File | Worst mean KL | Top-1 | Write (tok/s) | 8K read (tok/s) | Result |
| --- | ---: | ---: | ---: | ---: | --- |
| **Myric KAT-Coder** i-quality | 0.0039 | 125/126 | 81 | 3,941 | pass |
| Myric KAT-Coder i-quality-v2 | | identical (3 of 4; the 4th a 0.501/0.499 tie) | 86 | 3,754 | pass |
| **Unsloth** UD-IQ1_M | 0.0109 | 131/132 | 213 | 4,317 | pass |
| Unsloth UD-IQ2_XXS | | identical | 204 | 4,800 | pass |
| Unsloth UD-IQ2_M | 0.0054 | 150/155 | 202 | 4,761 | pass |
| Unsloth UD-Q2_K_XL | 0.0157 | 130/132 | 186 | 4,665 | pass |
| Unsloth UD-IQ3_XXS | 0.0088 | 157/160 | 154 | 4,642 | pass, new kernels |
| Unsloth UD-Q3_K_M | | identical | 119 | 4,566 | pass |
| Unsloth UD-Q3_K_XL | 0.0039 | 152/155 | 118 | 4,600 | pass |
| Unsloth UD-IQ4_NL | 0.0042 | 157/160 | 100 | 4,596 | pass, new kernels |
| Unsloth UD-Q4_K_S | | identical | 91 | 3,942 | pass |
| Unsloth UD-Q4_K_M | | identical | 86 | 3,969 | pass |
| Unsloth UD-Q4_K_XL | 0.0047 | 157/160 | 84 | 4,016 | pass |
| Unsloth MXFP4_MOE | 0.0036 | 158/160 | 83 | 3,943 | pass, new kernels |
| Unsloth UD-Q5_K_S | 0.0031 | 158/160 | 66 | 3,891 | pass |
| Unsloth UD-Q5_K_M | 0.0460 | 156/160 | 61 | 3,942 | pass |
| Unsloth UD-Q5_K_XL | 0.0507 | 156/160 | 60 | 3,931 | pass by judgment ([below](#ud-q5_k_xl)), new kernels |
| Unsloth UD-Q6_K | 0.0349 | 157/160 | 57 | 3,888 | pass, new kernels |
| Unsloth UD-Q6_K_XL | 0.0148 | 156/160 | 52 | 2,700 | pass, new kernels |
| Unsloth Q8_0 | | identical | 45 | 2,607 | pass |
| Unsloth UD-Q8_K_XL | | | | | **does not load** |
| **bartowski** IQ2_XS | 0.0357 | 159/160 | 199 | 4,691 | pass, new kernels |
| bartowski IQ2_M | 0.0081 | 158/160 | 177 | 4,729 | pass, new kernels |
| bartowski Q2_K | 0.0040 | 158/160 | 170 | 4,062 | pass, new kernels |
| bartowski IQ3_XXS | | identical | 129 | 4,338 | pass |
| bartowski Q3_K_M | 0.0050 | 132/132 | 120 | 4,079 | pass |
| bartowski IQ3_XS | 0.0052 | 159/160 | 118 | 4,317 | pass, new kernels |
| bartowski IQ3_M | 0.0035 | 157/160 | 114 | 4,120 | pass, new kernels |
| bartowski Q3_K_XL | 0.0053 | 154/155 | 107 | 3,960 | pass |
| bartowski IQ4_XS | | identical | 97 | 4,347 | pass |
| bartowski IQ4_NL | 0.0082 | 157/160 | 96 | 4,556 | pass, new kernels |
| bartowski Q4_0 | | identical | 93 | 3,862 | pass |
| bartowski Q4_K_M | | identical | 88 | 3,974 | pass |
| bartowski Q4_1 | 0.0062 | 155/160 | 84 | 3,956 | pass, new kernels |
| bartowski Q5_K_M | | identical | 71 | 3,975 | pass |
| bartowski Q6_K | 0.0044 | 159/160 | 58 | 3,851 | pass, new kernels |
| bartowski Q8_0 | 0.0186 | 154/155 | 44 | 2,615 | pass |

"New kernels" means the file did not run, or did not run correctly, before this branch ([what changed](#what-the-engine-needed)).
Totals out of fewer than 160 positions come from an earlier version of the test, which counted 39 positions per
prompt and stopped at the end-of-text token (on the 8K prompt that came after 9-15 tokens for some files).

### UD-Q5_K_XL

It is the one file over a limit: on the 8K prompt, the mean KL after the first position is 0.0507 (limit 0.05); the
median is 0.0056 and the other three prompts are well under. The excess is at positions 10-13 of the continuation. At
two of them, llama.cpp's CPU and GPU runs on the same file differ from each other by more (KL 0.14-0.44) than Strata
differs from llama.cpp's GPU run. At position 13 Strata is the outlier, on a token llama.cpp's CPU run also puts at
about 50/50. It is counted as a pass on that comparison; the measurement is as shown.

## What the engine needed

Fourteen files would not load, or loaded and gave wrong answers, before this branch: their experts or other weights
are in formats the engine had no GPU kernels for.

| Format | Where | Files |
| --- | --- | --- |
| Q6_K | expert gate/up (it was an opt-in build; now on by default, `-DSTRATA_Q6K_EXPERTS=OFF` drops it) | Unsloth UD-Q5_K_XL (one layer), UD-Q6_K, UD-Q6_K_XL; bartowski Q6_K |
| IQ3_S, IQ2_S, IQ2_XS, Q2_K | expert down | Unsloth UD-IQ3_XXS; bartowski IQ2_XS, IQ2_M, IQ3_XS, IQ3_M, Q2_K |
| IQ4_NL | expert gate/up | Unsloth UD-IQ4_NL; bartowski IQ4_NL |
| MXFP4 | expert gate/up/down; the file loader's block sizes | Unsloth MXFP4_MOE |
| Q4_1, Q2_K | attention and shared-expert weights, both for answers and for reading prompts | bartowski Q4_1, Q2_K |
| BF16 | expert gate/up/down | Unsloth UD-Q8_K_XL (still does not load, below) |

The Q6_K gate/up kernels add 8.8 KB of GPU code; the automatic expert cache of Flash-Next Coder and of Qwen3.6
UD-IQ4_XS was the same within its run-to-run spread (~30 slots) with and without them. With the branch's changes,
Flash-Next Coder (two prompts) and Qwen3.6 UD-IQ4_XS (four prompts, the 8K one included) give the same tokens as the
build before them, byte for byte. The kernel parity tests (`iq_parity`, `native_grouped_parity`, `qsa_parity`,
`native_expert_parity` with the new formats, at Qwen3.6's 2048 x 512 shape too) pass on the RTX 4070 Ti. The HIP
(gfx906 and four other targets) and SYCL builds compile; they were not run.

## The one that does not run

Unsloth's UD-Q8_K_XL (39 GB) stores its experts in Q8_0 and BF16, and one layer's attention and shared-expert weights
in BF16. The BF16 experts now have kernels. The BF16 attention and shared-expert weights do not: the engine's
answer path does not take them in that form, and changing how it reads BF16 tensors would also change how it reads
Flash-Next's. It is left unsupported. At 39 GB it does not fit the 16-32 GB PCs this model is for; Q8_0 (45 tok/s
above) is the near-lossless choice that runs.

## Installing one

Setup installs any of the 36 by name, from this branch's checkout:

```
./setup.sh --setup --family qwen36 --model UD-Q4_K_XL                 (Unsloth's)
./setup.sh --setup --family qwen36-bartowski --model Q4_K_M           (bartowski's)
./setup.sh --setup --family kat-coder --model i-quality-v2            (KAT-Coder)
START-HERE.bat --setup --family qwen36 --model UD-Q4_K_XL             (Windows)
```

Without `--model` it lists the family's sizes. For Unsloth's the two regular sizes come first and stay the suggestion
(UD-IQ4_XS, or UD-IQ3_S on a 16-24 GB PC); the others follow, each with its download, the RAM its experts take and
the speed above. bartowski's suggest IQ4_XS (IQ3_XXS on 16-24 GB), KAT-Coder its i-quality-v2. Setup downloads the
file from the pinned revision, checks its SHA-256, makes the pack (`tools/iq_pack.py --compat-bf16`) and writes the
server's config, the draft layer read from the same file, as for the regular sizes.

These files need an engine with the formats added on this branch. Setup looks for them in the engine (the text
"expert formats 2:" that `src/program/generate.cpp` names); a ready-made engine has none yet, so setup compiles the
engine on the PC (10-20 minutes, once). An engine compiled earlier from a checkout without them stops setup before
the download: run it again with `--build`.

The RAM each needs is its experts plus ~11 GB, the rule the regular sizes' measured lines follow: from ~20 GB
(UD-IQ1_M, bartowski IQ2_XS) to ~46 GB (the two Q8_0). A file whose experts do not fit the RAM uses the low-RAM mode
as the regular sizes do. `setup --check` does not list them one by one.

## Not tested

- AMD and Intel cards: the HIP and SYCL builds compile, but none of these files was run on them.
- Cards other than the RTX 4070 Ti, and smaller budgets than 12 GB. Files of 30 GB and more need that much RAM for
  their experts as well; on this 64 GB PC they all fit.
- Older NVIDIA cards (Pascal, Volta): there the engine reads prompts through the decode windows, which drifts with
  length (KL 0.06 against the batched prompt path at 8K, measured on Qwen3.6 UD-IQ4_XS on the RTX 4070 Ti).
- Answers longer than 40 tokens per prompt, and image input with these files (images: [QWEN36.md](QWEN36.md),
  Limits; tested with Ornith and KAT-Coder).
- Setup's install of these files end to end: its tests (`tools/test_setup_qwen36_more.py`) run it with the downloads,
  the compile and the GPU mocked; the files themselves were downloaded, checked and packed by the test scripts above.
- Faster K-quant prompt kernels (`-DSTRATA_MMQ_KQUANTS=ON`): not in setup's build, and not in these numbers. With them,
  UD-Q4_K_XL read the 8K prompt at 4,359 tok/s (on the #1402 branch).

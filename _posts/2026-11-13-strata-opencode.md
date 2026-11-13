---
title: "Strata + OpenCode: A Local Coding Agent On A 4060 Ti"
---
# Strata + OpenCode: A Local Coding Agent On A 4060 Ti

I replaced the cloud LLM behind my coding agent with a model running entirely on my own machine. [Strata](https://github.com/Niko1221/Strata) serves a quantized Qwen model locally through an OpenAI-compatible API, and [OpenCode](https://opencode.ai/) - an open-source coding agent, for me in its desktop app - talks to it as if it were any other provider. Nothing leaves my PC except git pushes.

This post covers the install, the one-command integration, and the two config files I tuned to make a locally-hosted agent usable: `opencode.json` for the context/compaction math, and a global `AGENTS.md` that teaches the agent about my hardware limits.

### The Hardware

*   GPU: NVIDIA RTX 4060 Ti with **16 GB VRAM**
*   RAM: **64 GB DDR4**

That is the entire budget. It is enough to run a capable coding model locally, but it is *tight*, and most of the customizations below exist because of that.

### Installing Strata

Strata is free and open source (MIT). On Windows, the CLI install is one line in PowerShell:

```powershell
irm -TimeoutSec 120 https://github.com/Niko1221/Strata/releases/latest/download/install.ps1 | iex
```

Open a new terminal afterwards, then launch:

```
strata launch
```

Follow the prompts to pick and download a model. I use **Qwen3.8-Flash-Next in the IQ3_S quantization**, which is what Strata's model guide recommends for ISTA-DASLab packs. On a 16 GB card with 64 GB of system RAM, Strata splits the model between VRAM and RAM - part on the GPU, the rest streamed from system memory. This is the key feature that makes the whole project possible on a 4060 Ti. Once the model is downloaded, I start it with a saved launch preset - `strata launch --preset "IQ3_S 256K"` - which is covered in the memory-math section below.

### Connecting OpenCode: The Connect Page

Strata exposes an OpenAI-compatible (and Anthropic-compatible) server at `http://127.0.0.1:8080/v1`. To wire it into a coding tool, start the terminal app:

```
strata launch
```

Then press **`7`** on the keyboard to jump to the **Connect** page. It lists the supported apps - OpenCode, Codex CLI, Claude Code and others - with each app's setup template ready to copy. Here is what it looks like with OpenCode selected:

<img src="https://i.postimg.cc/854vX2qw/connect-strata.png" alt="Strata's connect screen showing the OpenCode integration tab, with the config template written to ~/.config/opencode/opencode.json and a note that limit.input is required for compaction to work">

A few things worth reading off this screen:

*   **OpenCode is vendor-tested** (here against OpenCode 1.19.3), meaning the integration is expected to work out of the box.
*   Pressing **`c`** copies the ready-made config, and **`w`** writes it directly to `~/.config/opencode/opencode.json` for you (keeping a backup). That is where to get the template for the OpenCode integration - you don't have to write the provider block by hand.
*   The setup **needs `limit.input`**. Strata's template sets it (to 256048), and without it OpenCode never compacts - i.e. summarizes and prunes - its own conversation, and you will eventually just hit an out-of-context error mid-task.

After that, open the OpenCode desktop app and pick the local model (`strata/qwen3.8-flash-next-iq3_s`) from the model dropdown - the agent then runs entirely against the local model. The model picker shows it under Recent as **Qwen3.8-Flash-Next (local)** from the **Strata (local)** provider, alongside the reasoning-effort selector (I run it at Medium):

<img src="https://i.postimg.cc/zGN0MrM3/select-local-model.png" alt="OpenCode desktop app model picker with Qwen3.8-Flash-Next (local) from the Strata (local) provider selected under Recent, and a Medium reasoning-effort setting">

My config ended up looking like this (Strata's template plus my own edits - the API key is Strata's local one and is masked here, yours is generated on your machine):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "strata/qwen3.8-flash-next-iq3_s",
  "small_model": "strata/qwen3.8-flash-next-iq3_s",
  "autoupdate": false,
  "compaction": {
    "auto": true,
    "buffer": 32768
  },
  "share": "disabled",
  "provider": {
    "strata": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Strata (local)",
      "options": {
        "baseURL": "http://127.0.0.1:8080/v1",
        "apiKey": "sk-strata-****"
      },
      "models": {
        "qwen3.8-flash-next-iq3_s": {
          "name": "Qwen3.8-Flash-Next (local)",
          "tool_call": true,
          "reasoning": true,
          "limit": {
            "context": 262144,
            "input": 229376,
            "output": 32768
          }
        }
      }
    }
  }
}
```

### Why Each Customization In `opencode.json`

**`tool_call: true` and `reasoning: true`.** Without these flags OpenCode treats the model as a plain chat completion and won't drive it as an agent. Qwen3.8-Flash-Next supports native tool calling and thinking, so the coding agent loop (read file, edit, run, repeat) works properly.

**The `limit` block: 256k full context, 32k output, 224k input.** Strata's template sets `limit.input` to 256048, which lets the model fill essentially the entire context window before OpenCode reacts. I changed the numbers so the math is explicit:

*   `context: 262144` - the full 256k token context window.
*   `output: 32768` - I want a guaranteed 32k of output room. A coding agent that gets cut off mid-diff is useless, so 32k is my working minimum for a single response.
*   `input: 229376` - exactly 256k − 32k. Reserving the output budget up front means an input that fits the limit can always be answered with a full 32k response.

**`compaction.buffer: 32768` with `auto: true`.** This is the piece that pairs with the input limit. OpenCode compacts (summarizes older turns) when the free space in the window drops below the buffer. With the buffer set to 32k, compaction fires exactly when there is no longer room for a full 32k output - the model compacts *before* it would be strangled, never after. Without an `input` limit and a buffer, compaction never happens at all and the agent dies on a context overflow instead.

**`small_model` set to the same local model.** OpenCode uses a second, cheaper model for background chores like title generation. Pointing it at Strata too means zero cloud calls anywhere - no API keys, no accidental leakage of file paths or session titles to a third party.

**`share: "disabled"`.** OpenCode can publish session transcripts as share links. With a local-first setup the whole point is that nothing leaves the machine, so sharing is off at the config level rather than relying on memory.

**`autoupdate: false`.** The whole integration is tuned against one known-good combination of OpenCode and Strata. A silent background update can change compaction or provider behavior and break it overnight. I update OpenCode manually, after checking that Strata's Connect page still lists my version as vendor-tested.

### The Real Constraint: RAM And VRAM

Here is the honest part of the writeup. When the model is loaded, **Strata consumes nearly all of the system RAM and almost all of the VRAM**. In a typical working session, Task Manager reads **59.5 GB of the 64 GB RAM** and **15.0 GB of the 16 GB VRAM** in use. That leaves very little headroom for anything else - and this includes the agent's own shell commands.

The consequence: **testing has to be done with the LLM unloaded.** I cannot have the agent run the test suite, load a second model, or start a heavy service while it is itself being served by a model that owns all the memory. Anything heavier than a linter or a syntax check risks an out-of-memory crash - which, mid-session, often means losing the agent *and* the test run at once.

### Quitting Strata: Free The RAM And VRAM

So how do I get the memory back when it is time to test? In the `strata launch` terminal, press **`q`** to quit - Strata then asks what to do with the loaded model, and I select **"Stop serving and free memory"**. That shuts down the engine and the image encoder and gives back **both the 15 GB of VRAM and the 59 GB of pinned RAM** at once, instead of lingering as a background server. With the memory genuinely free, I can run the test suite, load a second model in ComfyUI, or do whatever else was impossible while the LLM was up - then bring the agent back in seconds with the same `strata launch --preset "IQ3_S 256K"` command (the model files stay in the OS file cache, so reloading is fast).

The habit matters more than the flag: press `q`, pick "Stop serving and free memory", *then* start the heavy task. Killing the terminal window or leaving Strata serving "just in case" keeps nearly the whole 64 GB pinned.

### Does The 256K Context Fit? The Memory Math

"Almost all of the memory" is a claim, so here is the arithmetic behind it, using the numbers from Strata's own model docs. My launch preset is minimal (API key redacted):

```json
{
  "name": "IQ3_S 256K",
  "family": "qwen",
  "model": "IQ3_S",
  "context": 262144,
  "port": 8080,
  "vision": "cpu",
  "tune": "quick"
}
```

Launched with:

```powershell
"C:\Users\Admin\AppData\Local\Strata\terminal\current\bin\strata.cmd" launch --preset "IQ3_S 256K"
```

**RAM side (64 GB DDR4).** Strata loads the IQ3_S model in shards. The first shard's MoE experts - **50 GB** - are copied into pinned system RAM. The second shard, a 29 GB n-gram lookup table, never touches RAM at all: it stays on the SSD and is read on demand. The whole model's weights are quoted at **54.8 GB of RAM + VRAM combined**, so nearly all of it lands here:

| RAM consumer | Size |
| --- | ---: |
| IQ3_S experts (shard 1, pinned) | 50.0 GB |
| KV cache for 262,144 tokens, 8-bit, streamed from RAM (~13.7 KB/token) | ~3.4 GB |
| **Total** | **~53.4 GB** |
| **Left for OS + apps** | **~10.6 GB** |

Strata's rule of thumb is that the model fits when RAM is at least the experts plus about 10 GB for the OS and other programs: 50 + 10 = 60 ≤ 64. ✓ And the 256K KV cache is what consumes the slack: 262,144 tokens × 13.7 KB ≈ 3.4 GB. That leaves ~10.6 GB - just at the ~10 GB the fit rule asks for. The docs even flag IQ3_S "with little else open" on a 64 GB PC, and Strata's own benchmarks stop at 128K for this quant, recommending 128K there; a longer context is accepted "with a note". My note: it fits arithmetically and runs, but *nothing else may be open* - which is exactly the discipline the rest of this post is about.

**VRAM side (16 GB on the 4060 Ti).** This is where the architecture does the heavy lifting, because context length barely grows VRAM:

| VRAM consumer | Size |
| --- | ---: |
| Non-expert weights (54.8 GB total − 50 GB experts in RAM) | ~4.8 GB |
| MTP draft layer (speculative decoding) | ~0.7-1 GB |
| KV cache resident in VRAM - only the ~32K-token window attention reads (32,768 × 13.7 KB) | ~0.4 GB |
| Vision encoder | **0** - reader runs on CPU |
| **Fixed total** | **~6 GB** |
| **Left for the expert cache (and Windows' own desktop use)** | **~10 GB** |

Two details make 256K safe on a 16 GB card:

*   **KV streaming.** Above 64K context Strata keeps the KV cache in system RAM and keeps only the ~32K-token window that attention actually reads resident in VRAM. A 256K conversation therefore costs ~0.4 GB of VRAM instead of the ~3.4 GB it would cost if the whole cache lived on the GPU - the extra ~3 GB shows up in system RAM instead, which is why the RAM budget above is the binding constraint.
*   **Vision on CPU.** Cards under 24 GB get the image reader on the CPU: a few seconds per picture, but **zero VRAM**. A GPU reader would cost 1-1.4 GB of the scarce 16 GB, so my preset pins `"vision": "cpu"` explicitly.

Whatever is left in VRAM goes to the adaptive expert cache - Strata keeps the hottest-used experts on the GPU and swaps them as the conversation moves. Every extra gig there is speed: in Strata's published benchmarks (on a 12 GB RTX 5070) IQ3_S writes 45-52 tokens/s and reads 32K-64K prompts at 1,400-1,600 tokens/s, easing off as the context fills; the 16 GB card and DDR4 here will land in a similar neighborhood, just slower.

**Measured on my machine.** In a real OpenCode session Task Manager confirms the picture: **59.5 GB used of the 64 GB RAM, and 15.0 GB used of the 16 GB VRAM.** Both match the tables once you attribute the slack honestly. On the RAM side the model's raw footprint is ~53.4 GB; the extra ~6 GB on top of it is Windows itself plus pinned-memory overhead and the engine's streaming buffers - so the truly free memory while the model runs is closer to **4-5 GB**, not the theoretical 10. On the VRAM side, the fixed ~6 GB from the table is real, but Strata's expert cache deliberately absorbs nearly all of the ~10 GB left over: the measured 15.0/16 is that cache doing exactly its job.

So the verdict holds: **256k of context fits on a 4060 Ti + 64 GB, estimate and measurement agree** - but the headroom that survives is a few GB, and that is precisely the space OpenCode, a browser and a test run would all try to occupy at once. Hence the next section.

### Teaching The Agent Its Limits: `AGENTS.md`

OpenCode reads a global `AGENTS.md` from `~/.config/opencode/` at the start of every session, and the local model can't be expected to know this context on its own. So my `AGENTS.md` is written as standing operating instructions for an agent that shares a small machine with the LLM it runs on:

*   **Files are the agent's memory.** `AGENTS.md`, `PLAN.md`, `PROGRESS.md` and `DECISIONS.md` split intent, current state, and history onto disk. A local session will get compacted (that's the 32k buffer doing its job) and new sessions start with zero memory - the files carry the project across both, so the agent never has to hold a whole project plan inside its context.
*   **"Do not run the test suite or any memory-heavy command."** This is the RAM/VRAM constraint stated verbatim. The agent implements, runs only lightweight static checks (`py_compile`, linting, type checks), then stops and hands the actual test run to me, with the LLM unloaded or reloaded as needed.
*   **Logging is the agent's eyes.** Because it can't see my test results unless they're on disk, `AGENTS.md` mandates the `logging` module, a `logs/latest.log`, per-case `[PASS]`/`[FAIL]` lines, and a final `RESULT: PASS`/`RESULT: FAIL` summary. When I say "the run finished", the agent reads the log file and diagnoses from it. If a failure can't be diagnosed from the logs, the first fix is to add more logging.
*   **One stage at a time, one commit per stage.** Each stage is committed before testing so the commit hash pins exactly what was tested, `PROGRESS.md` records the last-known-good commit and tags it, and reverting uses non-destructive `git revert`. If two fixes in a row fail, the agent must stop and recommend going back instead of thrashing - a local model burning its context on a doom loop is expensive in wall-clock time.
*   **Project-local venvs via `uv`, never system installs.** Another headroom guard: keeps the environment reproducible and prevents surprise installs eating disk and memory.

None of these are exotic ideas - they're ordinary engineering discipline, written down because a locally-hosted agent has no spare capacity to recover from sloppiness on its own.

### The Result

`strata launch` in a terminal, the OpenCode desktop app in its own window with the local model selected in the dropdown, and the coding agent runs against Qwen3.8-Flash-Next served from my own GPU and RAM: 256k of context, a guaranteed 32k of output per turn, compaction that fires exactly when the output budget runs out, and a workflow where testing happens on the human's schedule with the model unloaded. It's slower than the cloud and it demands more discipline - but it's private, it costs nothing per token, and it fits on a 4060 Ti.

# Using MegaCapybara

The launcher does everything on this page for you. This guide is for running without it: the presets, Linux
servers, and connecting clients to the API. Back to the [README](../README.md).

- [Presets: serving without the launcher](#presets-serving-without-the-launcher)
- [Linux](#linux)
- [Clients and the API](#clients-and-the-api)
- [Serving your network](#serving-your-network)
- [Unsigned programs](#unsigned-programs)

## Presets: serving without the launcher

The `presets` folder has ready-made start scripts: `.bat` files on Windows, `.sh` files on Linux. Every preset takes
the largest context that fits (`CONTEXT=auto`) and keeps 1 GiB of VRAM free (`KEEP_FREE=1`).

**The launcher's three**, all with one context pool shared by the conversations, and RAM and disk caches for idle
ones:

| Preset | Conversations | KV cache | Activations | One conversation up to | Vision |
|---|--:|---|---|---|---|
| `max_efficiency` | 8 | 4/4 | FP8 | 1,048,576 tokens (YaRN) | FP8 |
| `medium` | 4 | 6/4 | FP8 | 262,144 tokens | FP8 |
| `max_quality` | 1 | 6/6 | 16-bit | 262,144 tokens | BF16 |

**Six more**, one model and number of conversations each, with fixed slots unless the name ends in `_pool`. The
contexts are what `auto` gave on an RTX 5090 with nothing else on it:

| Preset | Model | Conversations | KV cache | Context |
|---|---|---|---|---|
| `Qwen3.8-27B-small_c1` | Small | 1 | 6/4 | ~634K tokens (YaRN) |
| `Qwen3.8-27B-small_c8` | Small | 8, fixed slots | 4/4 | ~90K each |
| `Qwen3.8-27B-small_c8_pool` | Small | 8, one pool | 4/4 | a ~723K pool, each up to 262,144 |
| `Qwen3.8-27B-medium_c1` | Medium | 1 | 6/4 | ~548K tokens (YaRN) |
| `Qwen3.8-27B-medium_c4` | Medium | 4, fixed slots | 4/4 | ~162K each |
| `Qwen3.8-27B-large_c1` | Large | 1 | 6/6 | 262,144 (the model's native context, no YaRN) |

### Running a preset

- **Start:** double-click a `.bat`. The server starts in a terminal window with a live dashboard.
- **Stop:** press **Q**, or close the window.
- **Your client's context size:** the start shows it on its Context line, *set your client's context size to N*.
  Enter N in your client (in a log, the line is `each conversation: up to N tokens`).
- **Override a value once:** options after the file name win, e.g. `presets\medium.bat --port 8081`.
  `megacapybara.exe --help` lists every option.

### Editing a preset

Open the `.bat` in a text editor and change the values at its top. Every preset has the same ones, each explained
right above it:

| Setting | Values |
|---|---|
| `MODEL` | the model file in `model\` |
| `CONTEXT_MODE` | `pool`: one context shared by all conversations, each taking what it uses; `fixed`: a slot each |
| `CONVERSATIONS` | how many are answered at once, 1 to 12 |
| `CONTEXT` | `auto` (the largest that fits), or a number of tokens: the pool's size, or each slot's |
| `KV` | the KV cache's bits: `6/6`, `6/4`, `8/8`, `4/4`, or `16/16` (fixed slots only) |
| `ROPE_SCALING` | `auto`, `off`, or `1m` (YaRN x4: one conversation up to 1,048,576 tokens) |
| `ACTIVATIONS` | `fp8`, or `bf16` (closer to the original, slower prompts) |
| `SPECULATOR` | `dflash2`, `mtp` or `off`; `DRAFT_LEN`: `0` (automatic) to `7` |
| `PREFILL_POLICY` | `balanced`, `decode-priority` (the writing agents stay fast) or `prefill-priority` (new prompts start sooner) |
| `LOOP_GUARD` | what happens when the model repeats itself: `stop`, `end_thinking`, `escape` or `off` |
| `VISION` | `fp8`, `bf16` or `off` |
| `RAM_CACHE`, `DISK_CACHE` | `auto`, GiB, or `0` for off (the disk cache needs the pool) |
| `KEEP_FREE` | GiB of VRAM left free for your desktop and other programs |
| `HOST`, `PORT` | `127.0.0.1` and `8080`; `0.0.0.0` serves your network (set `API_KEY` then) |
| `API_KEY` | a password clients must send as their API key; empty: none |

Unpacking a new release over the folder replaces the presets, so keep your own under a new name.

### Model files

Model files go in `model\` under their names on Hugging Face. The launcher downloads them, or get them from
[perkel/Qwen3.8-27B-MC](https://huggingface.co/perkel/Qwen3.8-27B-MC) (and
[-Uncensored-MC](https://huggingface.co/perkel/Qwen3.8-27B-Uncensored-MC)). Put the drafter
(`qwen3.8-27b-dflash2.mcapy`) and the vision tower (`qwen3.8-27b-vision-fp8.mcapy`) beside the model: without them,
answers are 2-4 times slower and images are refused.

`megacapybara.exe` started alone uses `settings.json` (8 fixed slots of 32K tokens) and the model in `model\`. With
several model files there, it picks Medium; choose another with `--model`.

## Linux

The Linux release, `MegaCapybara-<version>-linux-x64.tar.gz` on [Releases](../../../releases/latest), has the server,
the launcher and the same nine presets as `.sh` files, with the same settings at their top.

### On a desktop

Unpack it and run `./MegaCapybaraLauncher`: the same launcher as on Windows. It draws on the CPU, so it takes no GPU
memory, and works on X11, on Wayland desktops through XWayland (GNOME and KDE have it), and in WSL2's WSLg.

It uses the desktop's own X11, Cairo, Pango and libcurl libraries. If any is missing (a minimal install, or a fresh
WSL2 Ubuntu), `./MegaCapybaraLauncher` names it and offers to install it with your package manager (apt, dnf, zypper
or pacman). It asks before installing anything; started from a file manager, it opens a terminal to ask.

### Without a desktop (a server over SSH)

1. Unpack it: `tar -xzf MegaCapybara-<version>-linux-x64.tar.gz`, then `cd MegaCapybara`.
2. Download a model, the drafter and the vision tower into `model/` (no Hugging Face account needed):

   ```bash
   cd model
   for f in qwen3.8-27b-medium qwen3.8-27b-dflash2 qwen3.8-27b-vision-fp8; do
     curl -L --fail -C - -O "https://huggingface.co/perkel/Qwen3.8-27B-MC/resolve/main/$f.mcapy"
   done
   cd ..
   ```

   A preset whose model is missing prints the commands for it.
3. Start a preset: `presets/medium.sh`. Options after the file name override its values, e.g.
   `presets/medium.sh --port 8081`.

### Good to know

- **In a terminal** the server shows the same dashboard as on Windows: **Q** or Ctrl+C stops it, Ctrl+Shift+C copies
  what you select. **Without one** (nohup, a systemd service, a pipe) it prints log lines instead.
- **Older distributions** (glibc before 2.38, such as Ubuntu 22.04) cannot run this build; the presets and the
  launcher say so when they start.
- **The NVIDIA driver** cannot be installed like a library (it needs a restart). Without it, the server tells you how
  to install it for your system.
- **WSL2** works, with the Windows NVIDIA driver. Keep the models in Linux's own file system (e.g. under `~`): files
  under `/mnt/c` load several times slower.
- **As a service:** a systemd unit with `ExecStart=/opt/MegaCapybara/presets/medium.sh` (wherever you unpacked it)
  runs it at boot, its log lines going to the journal.

## Clients and the API

### OpenAI-compatible clients

OpenCode, Cline, Continue, Open WebUI and any other OpenAI-compatible client works:

| Field | Value |
|---|---|
| Base URL | `http://127.0.0.1:8080/v1` |
| API key | anything, or the server's password if it has one |
| Model | the name from `/v1/models` |
| Context size | the number the launcher or the server's start shows |

| Endpoint | What it does |
|---|---|
| `POST /v1/chat/completions` | chat, with streaming, tool calls, thinking (`reasoning_effort`, or `chat_template_kwargs.enable_thinking`) and images as base64 `data:` URLs |
| `POST /v1/completions` | plain text completion |
| `GET /v1/models` | the model's name and context length |
| `GET /health`, `GET /stats` | whether the server is up; its counters |

```bash
curl http://127.0.0.1:8080/v1/chat/completions -H "Content-Type: application/json" \
  -d '{"messages":[{"role":"user","content":"Write a haiku about capybaras."}]}'
```

### Claude Code and Anthropic's API

The server also speaks Anthropic's Messages API, so **Claude Code** and Anthropic's SDKs run on it. Use base URL
`http://127.0.0.1:8080` (without `/v1`) and any API key:

```bash
ANTHROPIC_BASE_URL=http://127.0.0.1:8080 ANTHROPIC_API_KEY=local claude --model qwen3.8-27b
```

In PowerShell on Windows:

```powershell
$env:ANTHROPIC_BASE_URL = "http://127.0.0.1:8080"; $env:ANTHROPIC_API_KEY = "local"
claude --model qwen3.8-27b
```

Also set `ANTHROPIC_DEFAULT_HAIKU_MODEL=qwen3.8-27b`, so Claude Code's background requests come here too.

What is supported:

- **`POST /v1/messages`**, streaming or not: text, images (base64), tools (`tool_use` / `tool_result`,
  `tool_choice`), thinking (`thinking.type` `enabled` or `adaptive`; `budget_tokens` ends the thinking when reached)
  and stop sequences. Thinking is off unless the request asks for it, as in Anthropic's API. Prompt reuse shows up as
  `cache_read_input_tokens`.
- **`POST /v1/messages/count_tokens`**, and `GET /v1/models` in Anthropic's format for Anthropic's SDKs.
- **Any model name** is accepted and echoed back; Qwen answers all of them.
- **`temperature: 1`**, Anthropic's default and the only value it allows with thinking, gives Qwen's recommended
  sampling instead; other values are used as given.
- **Long conversations:** give each conversation a large context (the presets do: up to 262,144 tokens). When a
  prompt does not fit, the server answers "prompt is too long" and Claude Code compacts the conversation.

Not supported: images by URL, PDF documents, and Anthropic's server tools (web search, code execution), which are
left out of the tools the model sees.

## Serving your network

The server listens on `127.0.0.1` only and asks for no API key. To serve your network, listen on `0.0.0.0` and give
it a password, set in any of these:

- the launcher's lock, next to the address;
- a preset's `API_KEY`;
- `"api_key"` in `settings.json`, or `--api-key`;
- the `MEGACAPYBARA_API_KEY` environment variable.

Clients send it as `Authorization: Bearer <key>` (OpenAI's clients) or `x-api-key: <key>` (Anthropic's; Claude Code
takes it from `ANTHROPIC_API_KEY`). Without it, everything but `/health` answers 401. The password travels in plain
HTTP, so on an untrusted network put TLS in front of the server.

## Unsigned programs

`megacapybara.exe` and `MegaCapybaraLauncher.exe` are not code-signed. On first start, Windows SmartScreen may show
*"Windows protected your PC"*: click **More info**, then **Run anyway**. Some antivirus programs flag unsigned
programs that download files (the launcher downloads models from Hugging Face). Each release lists the SHA-256 of its
archives, so you can check what you downloaded.

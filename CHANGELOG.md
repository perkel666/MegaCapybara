# Changelog

What changed in each release of MegaCapybara. Downloads are on the [releases page](https://github.com/perkel666/MegaCapybara/releases).

## 1.08 (2026-10-04)

- **See where each conversation's context is.** The monitor's Conversations tab shows VRAM, RAM and disk side by
  side, each conversation as a dot with its size; a prompt being read shows in its slot.
- **Statistics tab** (instead of the launcher's Server log): how the slots spent the last 10 minutes and the time
  since the server started, so you can see whether the clients send enough work or requests wait for context.
- **Smarter cache, the disk as the last resort.** Finished jobs' caches go first and are never written to disk;
  nothing idle for an hour stays on disk; an active conversation's state stays in RAM. On a busy hosted hour this
  removes thousands of disk round trips and the ~3 s wait they added to agent turns.
- The server's log tags every conversation (`[conv N]`), so its turns and its cache can be followed.

## 1.04 (2026-10-03)

- **Download counts on Hugging Face.** After the launcher downloads a model, it requests that model repository's
  `config.json` once, so Hugging Face counts the download. Nothing about you or your PC is sent.

## 1.03 (2026-10-03)

Conversations keep their cache when many agents share the server. Under memory pressure, 1.02 could drop a whole idle
conversation (often 100K–300K tokens) and read it again from scratch on its next turn.

- A conversation coming back from RAM no longer causes another one to be dropped: it swaps places with an idle one.
- Running out of pinned RAM moves another conversation to disk instead of dropping the one being moved.
- The conversations needed last make room first; one is never pushed out for one that is needed later.

## 1.02 (2026-10-03)

- First public release: the server and the launcher for Windows 11 and Linux, the presets, and model downloads from
  Hugging Face in the launcher.

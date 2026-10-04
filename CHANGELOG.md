# Changelog

What changed in each release of MegaCapybara. Downloads are on the [releases page](https://github.com/perkel666/MegaCapybara/releases).

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

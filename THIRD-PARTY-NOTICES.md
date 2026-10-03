# Third-party notices

MegaCapybara is under the MIT License (`LICENSE`), Copyright (c) 2026 Perkel's Software Corner
<bluefactorygames@gmail.com>. The material below keeps its own license. Every release archive has a `LICENSES`
folder with each license text.

## Libraries compiled into the programs

Compiled into the server (`megacapybara.exe` on Windows, `megacapybara` on Linux) and the launcher
(`MegaCapybaraLauncher.exe`, `MegaCapybaraLauncher`):

| Library | License | Copyright |
|---|---|---|
| FTXUI | MIT | (c) 2019 Arthur Sonzogni; contains easing functions from AHEasing (WTFPL) |
| cpp-httplib | MIT | (c) Yuji Hirose |
| nlohmann/json | MIT | (c) 2013-2025 Niels Lohmann; contains code by Bjoern Hoehrmann (MIT), Florian Loitsch (MIT), Evan Nemerson (Hedley) and the Abseil Authors (Apache-2.0) |
| xxHash | BSD-2-Clause | (c) 2012-2021 Yann Collet |
| stb_image v2.30 | MIT or public domain (either) | Sean Barrett; the Linux programs only (decoding images; Windows uses its own) |

## Data

The Linux server's Unicode normalization tables (the tokenizer's NFC; Windows uses its own) are generated from the
Unicode Character Database 15.0, Copyright (c) 1991-2023 Unicode, Inc., under the Unicode License v3.

The server implements the DFlash2 drafter's math as SGLang's DFlash2 code does (SGLang: Apache-2.0); no code is
copied.

## Not in the programs

- **Model weights** (the `.mcapy` files, downloaded from Hugging Face): Qwen3.8-27B by the Qwen team (Alibaba
  Cloud) and Qwen3.8-27B-DFlash2 by z-lab (inco.ai), both Apache-2.0. The package's `LICENSES\README.txt` and each
  model repository's card list each file, its source and what was changed.
- **NVIDIA CUDA Toolkit**: the CUDA runtime is linked statically into the server as a redistributable component under
  NVIDIA's CUDA Toolkit EULA (https://docs.nvidia.com/cuda/eula/); it stays under NVIDIA's terms, not MegaCapybara's
  license.
- **GCC's runtime** (libstdc++, libgcc) is linked statically into the Linux programs under the GCC Runtime Library
  Exception, which puts no terms on the program.

Qwen, NVIDIA, CUDA and Patreon are trademarks of their owners; MegaCapybara is not affiliated with or endorsed by them.

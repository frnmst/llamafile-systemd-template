<!--
SPDX-FileCopyrightText: 2026-2026 Franco Masotti (See /README.md)

SPDX-License-Identifier: MIT
-->

# llamafile Systemd Template

<a href="https://www.buymeacoffee.com/frnmst" target="_blank"><img src="https://img.shields.io/badge/-buy_me_a%C2%A0coffee-gray?logo=buy-me-a-coffee" alt="Buy Me A Coffee"></a>

Automatically download and run llamafiles as Systemd services, with suggested
role presets. Fine tuned for CPU inference.

<!--TOC-->

- [llamafile Systemd Template](#llamafile-systemd-template)
  - [Description](#description)
  - [Quickstart](#quickstart)
  - [GPU](#gpu)
    - [Intel iGPU](#intel-igpu)
  - [Preset configurations](#preset-configurations)
  - [Custom models](#custom-models)
    - [Ornith-1.5-9B-GGUF](#ornith-15-9b-gguf)
  - [Tested models](#tested-models)
    - [CPU](#cpu)
      - [Intel Core i7 9700 (CPU only)](#intel-core-i7-9700-cpu-only)
  - [Contributing, changelog and security](#contributing-changelog-and-security)
  - [Consulting and custom integrations](#consulting-and-custom-integrations)
  - [License](#license)
  - [Git forge mirrors](#git-forge-mirrors)
  - [Support this project](#support-this-project)
<!-- TOC by https://github.com/frnmst/md-toc ver. 9.1.0 -->

<!--TOC-->

## Description

llamafiles are all-in-one executables capable of bootstrapping AI LLMs in one
click or command. Some models are now capable of running quite well on older
(5+ years) CPUs with little RAM. Integrated GPUs also work by using
[Vulkan](https://www.vulkan.org/) with little extra setup. You need to
benchmark which method is faster for inference on your hardware.

This repo adds automations for downloading, running with static options and
model presets which makes them suitable for basic real-world use cases.

> [!IMPORTANT]
> All these steps have been tested on Debian 13 systems. You might need minor
> adjustments for your distro!

## Quickstart

1. install dependencies, create a user and a base directory for the models:

   ```shell
   apt install curl
   useradd -m -s /bin/bash -U llamafile
   passwd llamafile
   sudo -i -u llamafile
   mkdir /home/llamafile/llamafiles
   ```

2. to perform inference with older Intel CPUs (tested on an i7 9700) use this
   Systemd template file. You can change the environment variables based on
   your needs. Place it in `/etc/systemd/system/llamafile@.service`:

   ```ini
   [Unit]
   Description=Run a llamafile model %i
   Requires=network-online.target
   After=network-online.target

   [Service]
   # Config.
   Environment=HOME=/home/llamafile
   Environment=MODEL_DIR=/home/llamafile/llamafiles
   Environment=PORT=8001

   # When serving globally, use a reverse proxy instead of 0.0.0.0 here!
   Environment=HOST=127.0.0.1

   # Other values: 'auto'
   Environment=GPU=none

   # Other values: 'all'
   Environment=GPU_LAYERS=0

   User=llamafile
   Group=llamafile
   PrivateTmp=true

   # Newer models use this path.
   Environment=LLAMAFILE_BASE_URL=https://huggingface.co/mozilla-ai/llamafile_0.10/resolve/main

   # CPU optimizations.
   Nice=-10

   # Downloads the llamafile.
   ExecStartPre=/bin/bash -c \
       'if [ ! -f ${MODEL_DIR}/%i ]; then \
            curl --location --output ${MODEL_DIR}/%i ${LLAMAFILE_BASE_URL}/%i; \
            chmod 700 ${MODEL_DIR}/%i; \
       fi'
   ExecStart=/bin/bash -c \
       '${MODEL_DIR}/%i \
           --reasoning off \
           --flash-attn 1 \
           --gpu ${GPU} \
           --gpu-layers ${GPU_LAYERS} \
           --port ${PORT} \
           --host ${HOST} \
           --nologo \
           --server'

   [Install]
   WantedBy=multi-user.target
   ```

3. make Systemd discover the template:

   ```shell
   systemctl daemon-reload
   ```

4. execute and enable a model:

   ```shell
   systemctl enable --now llamafile@{model_filename}
   ```

   For example if you want to try Ministral-3-3B-Instruct-2512-Q4\_K\_M,
   the enable command could be:

   ```shell
   systemctl enable --now llamafile@Ministral-3-3B-Instruct-2512-Q4_K_M.llamafile
   ```

   You can select pre-built llamafiles from
   [this list](https://docs.mozilla.ai/llamafile/getting-started/pre-built-llamafiles).
   Use this pattern for other llamafiles you want to test. You can also
   [build your own](#custom-models) and place it in the models directory

5. [connect to the web UI](http://127.0.0.1:8001)
6. via the web UI, import a [JSON preset configuration](#preset-configurations)
   file based on a specific role. This step is required to have decent quality
   output. Go to:

   1. Settings
   2. Import/Export
   3. Import settings

> [!NOTE]
> During the first boot the terminal seems to hang: curl needs a few minutes
> to download the model in the background.

## GPU

### Intel iGPU

If you don't have a dedicated GPU you can try to use hardware acceleration
via the integrated GPU. llamafile will use the
[Vulkan API](https://wiki.archlinux.org/title/Vulkan) for this. Depending on
the system, this solution might be slower that using pure CPU.

1. install the Intel tool:

   ```shell
   apt install intel-gpu-tools
   ```

2. add the `llamafile` user to the `video` and `render` group:

   ```shell
   usermod -aG video llamafile && usermod -aG render llamafile
   ```

3. reboot
4. enable GPU use in the Systemd service defined earlier:

   ```ini
   # Other values: 'auto'
   Environment=GPU=auto

   # Other values: 'all'
   Environment=GPU_LAYERS=all
   ```

5. run the service and check GPU usage with:

   ```shell
   intel_gpu_top
   ```

## Preset configurations

[Configurations](./config) are tuned by model and role:

```
./config/{model_filename}/{role}.json
```

A good source of system prompts are the ones used by
[DuckDuckGo AI chat](https://duck.ai/chat?ia=chat&duckai=1).

## Custom models

You can create your llamafile starting with a GGUF file and a configuration.
[Follow these instructions](https://docs.mozilla.ai/llamafile/using-llamafile/creating_llamafiles).

### Ornith-1.5-9B-GGUF

- [homepage](https://huggingface.co/ornith-ai/Ornith-1.5-9B-GGUF)
- [direct model download](https://huggingface.co/ornith-ai/Ornith-1.5-9B-GGUF/resolve/main/Ornith-1.5-9B-Q4_K_M.gguf?download=true)
- `.args` file (precise coding tasks)

  ```
  -m
  /zip/Ornith-1.5-9B-Q4_K_M.gguf
  -fa
  on
  --temp
  0.6
  --top-p
  0.95
  --top-k
  20
  --min-p
  0.0
  --presence-penalty
  0.0
  --repeat-penalty
  1.0
  --reasoning
  off
  ```

## Tested models

> [!IMPORTANT]
> Numbers originate just from observation, not real benchmarks.

### CPU

#### Intel Core i7 9700 (CPU only)

| Model Name | Avg T/s | Speed notes |
|------------|---------|-------------|
| Ministral-3-3B-Instruct-2512-Q4\_K\_M | 9 | Quite fast |
| Ornith-1.5-9B-Q4\_K\_M | 4.5 | Limit of being acceptable |
| Qwen3.5-9B-Q5\_K\_S | 4 | Limit of being acceptable |

## Contributing, changelog and security

- [CONTRIBUTING](./CONTRIBUTING.md)
- [CHANGELOG](./CHANGELOG.md)
- [SECURITY](./SECURITY.md)

## Consulting and custom integrations

If you are an organization, enterprise or an individual relying on
llamafile-systemd-template and need help, custom integrations or feature
development, I'm available for contract-based freelance consulting:

- Email: <solvecomputersciencecollabs+llamafile-systemd-template@gmail.com>
- Freelancing: <https://blog.franco.net.eu.org/jobs/>

## License

MIT License

Copyright (c) 2026 Franco Masotti (See `./README.md`).

## Git forge mirrors

| URL | Type | Notes |
|-----|------|-------|
| https://github.com/frnmst/llamafile-systemd-template | RW | Official home |
| https://codeberg.org/frnmst/llamafile-systemd-template | RW | Mirror |
| https://framagit.org/frnmst/llamafile-systemd-template | RW | Mirror |
| https://repos.franco.net.eu.org/frnmst/llamafile-systemd-template | RW | Mirror |

## Support this project

- [GitHub Sponsors](https://github.com/sponsors/frnmst)
- [Buy Me a Coffee](https://www.buymeacoffee.com/frnmst)
- [Liberapay](https://liberapay.com/frnmst)

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
  - [llamafiles](#llamafiles)
    - [Pre-built](#pre-built)
    - [Custom](#custom)
      - [Templates](#templates)
  - [Quickstart](#quickstart)
  - [GPU](#gpu)
    - [Intel iGPU](#intel-igpu)
  - [Preset configurations](#preset-configurations)
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
[Vulkan](https://www.vulkan.org/) with some little extra setup.

This repo adds automations for downloading, running with static options and
model presets which makes them suitable for basic real-world use cases.

> [!IMPORTANT]
> All these steps have been tested on Debian 13 systems. You might need minor
> adjustments for your distro! You also need to benchmark which method is
> faster for inference on your hardware.

## llamafiles

### Pre-built

Mozilla AI provides
[this list](https://docs.mozilla.ai/llamafile/getting-started/pre-built-llamafiles)
of pre-built llamafiles.

### Custom

You can create your llamafiles starting with GGUF files and configurations.
[This search filter](https://huggingface.co/models?pipeline_tag=text-generation&num_parameters=min:3B,max:9B&library=gguf&sort=downloads) lists suitable small models for CPUs on Hugging
Face.

Once you select a model you must
[follow these instructions](https://docs.mozilla.ai/llamafile/using-llamafile/creating_llamafiles).
Often the correct models' parameters are listed on the README files hosted on
Hugging Face.

#### Templates

Tested templates are in the [custom\_llamafiles](./custom_llamafiles)
directory.

## Quickstart

1. install dependencies, create a user and a base directory for the models:

   ```shell
   apt install curl
   useradd -m -s /bin/bash -U llamafile
   passwd llamafile
   sudo -i -u llamafile
   mkdir /home/llamafile/llamafiles
   ```

2. to perform inference with older Intel CPUs (tested on an i7 9700) use the
   [Systemd template service file](./llamafile@.service). You can change the
   environment variables based on your needs. Place it
   in `/etc/systemd/system/llamafile@.service`
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
   [build your own](#custom) and place it in the models directory

5. [connect to the web UI](http://127.0.0.1:8001)
6. via the web UI, import a [JSON preset configuration](#preset-configurations)
   file based on a specific role. This step is required to have decent quality
   output. Go to: Settings -> Import/Export -> Import settings

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


## Tested models

> [!IMPORTANT]
> Numbers originate just from observation, not real benchmarks.

### CPU

#### Intel Core i7 9700 (CPU only)

| Model Name | Average Tok/s | Total system RAM (G) | Speed notes |
|------------|---------|-------------|------------------|
| Ministral-3-3B-Instruct-2512-Q4\_K\_M | 9 | 16 | Quite fast |
| Ornith-1.5-9B-Q4\_K\_M | 4.5 | 16 | Limit of being acceptable |
| Qwen3.5-9B-Q5\_K\_S | 4 | 16 | Limit of being acceptable |
| Qwen2.5-Coder-7B-Instruct-Abliterated-Q4\_K\_M | 4.5 | 16 | Limit of being acceptable |

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

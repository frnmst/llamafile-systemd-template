<!--
SPDX-FileCopyrightText: 2026-2026 Franco Masotti (See /README.md)

SPDX-License-Identifier: MIT
-->

# llamafile Systemd Template

<a href="https://www.buymeacoffee.com/frnmst" target="_blank"><img src="https://img.shields.io/badge/-buy_me_a%C2%A0coffee-gray?logo=buy-me-a-coffee" alt="Buy Me A Coffee"></a>

Automatically download and run llamafiles as Systemd services, with suggested
role presets.

<!--TOC-->

- [llamafile Systemd Template](#llamafile-systemd-template)
  - [Description](#description)
  - [Quickstart](#quickstart)
  - [Preset configurations](#preset-configurations)
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
[Vulkan](https://www.vulkan.org/) without extra setup, but you need to
benchmark which method is faster for inference on your hardware.

This repo adds a automations for downloading, running with static options and
model presets which makes them suitable for basic real-world use cases.

## Quickstart

1. create a user and a base directory for the models:

   ```shell
   useradd -m -s /bin/bash -U llamafile
   passwd llamafile
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

   # Use a reverse proxy instead of 0.0.0.0 here!
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
   systemctl enable --now llamafile@{model_relative_filename}
   ```

   For example if you want to try Ministral-3-3B-Instruct-2512-Q4\_K\_M,
   the enable command could be:

   ```shell
   systemctl enable --now llamafile@Ministral-3-3B-Instruct-2512-Q4_K_M.llamafile
   ```

   You can select pre-built llamafiles from
   [this list](https://docs.mozilla.ai/llamafile/getting-started/pre-built-llamafiles).
   Use this pattern for other llamafiles you want to test

5. [connect to the web UI](http://127.0.0.1:8001)
6. via the web UI, import a [JSON preset configuration](#preset-configurations)
   file based on a specific role. This step is required to have decent quality
   output. Go to:

   1. Settings
   2. Import/Export
   3. Import settings

## Preset configurations

[Configurations](./config) are tuned by model and role:

```
./config/{model_filename}/{role}.json
```

A good source of system prompts are the ones used by
[DuckDuckGo AI chat](https://duck.ai/chat?ia=chat&duckai=1).

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

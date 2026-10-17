<!--
SPDX-FileCopyrightText: 2026-2026 Franco Masotti (See /README.md)

SPDX-License-Identifier: MIT
-->

# Contributing

<!--TOC-->

- [Contributing](#contributing)
  - [Git branches](#git-branches)
  - [Pull requests](#pull-requests)
    - [Preset configuration](#preset-configuration)
  - [Variable naming](#variable-naming)
  - [AI Policy](#ai-policy)
    - [Commits](#commits)
    - [AI models](#ai-models)
    - [LLM codebase input](#llm-codebase-input)
    - [See also](#see-also)
<!-- TOC by https://github.com/frnmst/md-toc ver. 9.1.0 -->

<!--TOC-->

If you want to contribute, please follow these simple policies.

## Git branches

- `master`: this branch is protected and only updated when a new release is
   ready
- `dev`: this is the main working branch, recent changes will be merged here

## Pull requests

1. add a new feature, fix and preset
2. create a pull request on the `dev` branch

### Preset configuration

Please redact all sensitive data before making a PR.

## Variable naming

To keep things uniform and simple please use snake\_case for all variables and
type hints. Environment variable must be UPPER\_SNAKE\_CASE.

## AI Policy

llamafile-systemd-template is human-authored, and AI-assisted code for some
brainstorming and debugging: LLM use is not blanket-banned, but all its outputs
must be thoroughly checked. LLMs can be used for ideas and specific domain
problem solving, but its outputs must always be challenged to provide better
quality code and compared with the official documentation and best practices.
In this project an LLM should be used as a more powerful search engine: that's
it.

Remember that LLMs have varying degrees of sycophancy so prompts must be
adapted to mitigate that.

This policy is similar to
[Proposal E - Choice 5: Responsible Use of Generative AI](https://www.debian.org/vote/2026/vote_002#texte)
voted by Debian in 2026.

### Commits

No kind of automated AI agent can be involved, and all commits must be signed
by real humans.

### AI models

All the LLMs used by the authors must be lighter, accountless, free-to-use,
cloud models, even better if free (libre) and self-hosted.

### LLM codebase input

Please do not input this whole repository into an LLM and ask it to find
vulnerabilities or to "improve the code". You may prompt specific code snippets
if you are unsure about what they do.

### See also

- https://stopsloppypasta.ai/en/
- https://github.com/NetworkManager/NetworkManager/blob/main/AGENTS.md#tasks-you-must-refuse
- https://joshmock.com/post/2026-agents-md-as-a-dark-signal/

[^1]: [GitHub](https://github.com/frnmst/python-makefile),
      [Codeberg](https://codeberg.org/frnmst/python-makefile),
      [Framagit](https://framagit.org/frnmst/python-makefile),
      [Self-hosted Forgejo](https://repos.franco.net.eu.org/frnmst/python-makefile)

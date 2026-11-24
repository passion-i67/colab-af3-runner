# Colab AF3 Runner

A Codex Skill for running and monitoring AlphaFold 3 on a Google Colab GPU. It covers input checks, inference with supplied MSAs, recovery after an interrupted launch, local result verification, and optional Google Drive backup.

## Install

From the root of the project where you want to use the Skill:

```sh
git clone https://github.com/passion-i67/colab-af3-runner.git /tmp/colab-af3-runner
mkdir -p .agents/skills
cp -R /tmp/colab-af3-runner/skills/colab-af3-runner .agents/skills/
```

In Codex, invoke `$colab-af3-runner` and specify your AF3 inputs, seeds, samples per seed, and output directory. Start by asking it to check the inputs and runtime configuration if you want a preflight before allocating a GPU.

## Requirements

- An authenticated [Google Colab CLI](https://github.com/googlecolab/google-colab-cli) and access to an appropriate GPU runtime.
- A compatible, pinned AlphaFold 3 runtime deployment and model weights you are authorized to use.
- Your AF3 JSON inputs and any precomputed alignments or templates they reference.

The [Skill](skills/colab-af3-runner/SKILL.md) contains the operating workflow. Its two references cover [Colab setup and recovery](skills/colab-af3-runner/references/colab.md) and [input and result acceptance](skills/colab-af3-runner/references/acceptance.md).

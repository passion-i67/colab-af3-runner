# Colab AF3 Runner

English | [简体中文](README.zh-CN.md)

A Codex Skill for running AlphaFold 3 batches on Google Colab. Codex uses the Colab CLI to connect to an authorized GPU session, validate AF3 inputs, and process requests in the specified order. The Skill also describes how to resume an interrupted batch and verify and back up its outputs.

## Install

From the root of the project where you want to use the Skill:

```sh
git clone https://github.com/passion-i67/colab-af3-runner.git /tmp/colab-af3-runner
mkdir -p .agents/skills
cp -R /tmp/colab-af3-runner/skills/colab-af3-runner .agents/skills/
```

## Run a batch in Codex

```text
$colab-af3-runner
Use the Colab CLI to run the AF3 JSON files in ./inputs in the order in
./run_order.tsv. Check the supplied MSAs and preserve each input's seeds.
Produce five samples per seed and save each request in ./results.
If the connection drops, check the same session and task before retrying.
Resume from the first unfinished request after verifying saved results.
Verify local file sizes and SHA-256 against the runtime inventory, then
back up each verified result to my specified Google Drive folder.
```

1. **Connect and run.** Verify the Colab GPU session and the pinned AF3 deployment. Check the inputs, including supplied MSA, before processing the batch.
2. **Resume safely.** Use the recorded session and task IDs to reconnect first. If the runtime is gone, reconcile the previous attempt and resume from the first unfinished request using the saved manifest.
3. **Verify and back up.** After each request, check that all expected seeds and samples are present. Compare the local output inventory with the runtime inventory by path, byte size, and SHA-256. For a requested Drive backup, upload the verified local copy and confirm its Drive metadata.

## Related projects

| Project | What it covers |
| --- | --- |
| [ByteTora/alphafold3-skills](https://github.com/ByteTora/alphafold3-skills) | AF3 skills for input generation, batch inference, result interpretation, and remote GPU jobs using Docker, Singularity, and SSH. |
| [Google Colab CLI: colab-operator](https://github.com/googlecolab/google-colab-cli/blob/main/skills/colab-operator/SKILL.md) | General Colab operations, including GPU allocation, remote execution, file transfer, and Drive mounting. |
| [ColabFold AF3 notebook](https://github.com/sokrypton/ColabFold/blob/main/AlphaFold3_of3.ipynb) | A Colab notebook for interactive structure prediction, with options for OpenFold3 and DeepMind AF3 weights. |
| [AF_Cache](https://github.com/clami66/AF_cache) | A Nextflow pipeline for high-throughput AF2 and AF3 prediction that reuses MSAs. |

Colab AF3 Runner focuses on the handoff from Codex to Colab and back to local storage. It puts ordered batch execution, recovery from a recorded checkpoint, and verified result delivery into one workflow.

## Requirements

- An authenticated [Google Colab CLI](https://github.com/googlecolab/google-colab-cli) and access to an appropriate GPU runtime.
- A compatible, pinned AlphaFold 3 runtime deployment and model weights you are authorized to use.
- Your AF3 JSON inputs and any precomputed alignments or templates they reference.

The [Skill](skills/colab-af3-runner/SKILL.md) contains the operating workflow. Its two references cover [Colab setup and recovery](skills/colab-af3-runner/references/colab.md) and [input and result acceptance](skills/colab-af3-runner/references/acceptance.md).

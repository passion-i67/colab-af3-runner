# Colab AF3 Runner

[English](README.md) | 简体中文

这是一个用 Codex 在 Google Colab 上批量运行 AlphaFold 3 的 Skill。Codex 通过 Colab CLI 连接已授权的 GPU 会话，校验 AF3 输入，再按指定顺序运行。流程也涵盖断点续跑、本机结果校验和可选的 Google Drive 备份。

## 安装

在要使用这个 Skill 的项目根目录执行：

```sh
git clone https://github.com/passion-i67/colab-af3-runner.git /tmp/colab-af3-runner
mkdir -p .agents/skills
cp -R /tmp/colab-af3-runner/skills/colab-af3-runner .agents/skills/
```

## 在 Codex 中批量运行

```text
$colab-af3-runner
用 Colab CLI 连接 Colab，按 ./run_order.tsv 的顺序运行 ./inputs 里的 AF3 JSON。
校验预计算 MSA，保留每个输入的种子。每个种子生成 5 个样本，按请求保存到 ./results。
如果连接中断，先核对同一个会话和任务，再决定是否重试。
核验已有结果后，从第一个未完成的请求续跑。
按运行时清单核对本机文件的大小和 SHA-256，然后把每份通过校验的结果备份到我指定的 Google Drive 文件夹。
```

1. **连接并运行。** 确认 Colab GPU 会话和固定版本的 AF3 环境。按清单运行前，先检查输入和预计算 MSA。
2. **断点续跑。** 优先用已记录的会话 ID 和任务 ID 恢复连接。如果运行时已退出，先核实上次尝试的状态，再根据保存的清单从第一个未完成的请求续跑。
3. **校验和备份。** 每个请求完成后，检查预期的种子和样本是否齐全，按路径、文件大小和 SHA-256 比对本机与运行时清单。需要 Drive 备份时，上传通过校验的本机副本，并核对 Drive 返回的文件信息。

## 相关项目对比

| 项目 | 主要覆盖范围 |
| --- | --- |
| [ByteTora/alphafold3-skills](https://github.com/ByteTora/alphafold3-skills) | AF3 Skill 集合，涵盖输入生成、批量推理、结果解读，以及通过 Docker、Singularity 和 SSH 运行远程 GPU 作业。 |
| [Google Colab CLI 的 colab-operator](https://github.com/googlecolab/google-colab-cli/blob/main/skills/colab-operator/SKILL.md) | 通用 Colab 操作，包括 GPU 分配、远程执行、文件传输和 Drive 挂载。 |
| [ColabFold AF3 Notebook](https://github.com/sokrypton/ColabFold/blob/main/AlphaFold3_of3.ipynb) | 在 Colab Notebook 中交互式运行结构预测，可选 OpenFold3 或 DeepMind AF3 权重。 |
| [AF_Cache](https://github.com/clami66/AF_cache) | 面向高通量 AF2 和 AF3 预测的 Nextflow 流水线，支持复用 MSA。 |

Colab AF3 Runner 聚焦从 Codex 提交 Colab 作业到本机接收结果的流程，将按序批量推理、断点恢复和结果验收串在一起。

## 使用条件

- 已完成认证的 [Google Colab CLI](https://github.com/googlecolab/google-colab-cli)，以及可用的 Colab GPU。
- 兼容且固定版本的 AlphaFold 3 运行环境，以及有权使用的模型权重。
- AF3 JSON 输入及其引用的预计算比对和模板。

[Skill 正文](skills/colab-af3-runner/SKILL.md) 描述完整流程。两份参考分别说明 [Colab 配置与故障恢复](skills/colab-af3-runner/references/colab.md) 和 [输入与结果验收](skills/colab-af3-runner/references/acceptance.md)。

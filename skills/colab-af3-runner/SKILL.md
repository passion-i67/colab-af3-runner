---
name: colab-af3-runner
description: Run, resume, monitor, and verify AlphaFold 3 inference on a Google Colab GPU from Codex, including precomputed-MSA batches, local result collection, and optional Google Drive backups.
---

# Colab AF3 Runner

Use the project's existing AF3 runner, validator, and task manifests whenever available. Otherwise, use the official Google Colab CLI with a version-pinned, Colab-compatible AF3 entrypoint. Read [Colab setup and recovery](references/colab.md) for environment setup, CLI syntax, or connection failures.

## Choose the operation

- **Plan or inspect:** Read inputs, installed CLI help, session status, and existing manifests. Provision a GPU only for an authorized run.
- **Run or resume:** Establish the task order and completion criteria, then launch the next eligible request.
- **Monitor or verify:** Follow the recorded session and task IDs. Check the heartbeat, terminal status, and saved output without launching another task.
- **Clean up:** Release only the exact session created for this work once its final state and artifacts have been accounted for.

## Run and resume

1. Identify the source inputs, model and implementation, seeds, samples per seed, GPU, output destination, and any backup destination. Keep the source inputs intact. Record each input's SHA-256 and planned order. For a batch with precomputed MSA, verify its chain mapping, A3M query sequences, and exact transport before running. See [Input and result acceptance](references/acceptance.md).
2. Check Colab authentication, active sessions, and available compute units. Record a unique session ID before requesting the GPU. Confirm the assignment, GPU type, and kernel connection before working on the runtime. On an ambiguous allocation, reconcile the same session ID before requesting another.
3. Verify the pinned AF3 environment and the authorized weights on the runtime. Check the full weight file against a trusted size and SHA-256. A runtime can download authorized weights directly from the official source and save results locally, with Drive backup performed separately. Mount Drive only when the selected workflow needs it, and verify the mount in that runtime.
4. Run a parser and featurization preflight before inference. Verify the received input hash, chain definitions, ligand or CCD handling, and planned seeds. For supplied MSAs, use the pinned runner's inference-only settings and keep external MSA lookups disabled.
5. Give each launch a task ID scoped to its session and request. Record the intended launch before submitting. Wait for a durable remote receipt or status record. If the response is lost, query the exact task ID and heartbeat. Resume monitoring if it is live. Apply the replacement rules in [Colab setup and recovery](references/colab.md) before considering another launch.
6. Require a successful exit and verify the expected result files and sample count. Verify the local files against the remote inventory before marking that request complete. If requested, upload a verified local copy to the confirmed Drive destination and check the resulting file metadata. Record this outcome independently of inference and local delivery.
7. Continue the manifest's order on the same healthy runtime where practical. Skip completed requests using verified artifacts. Preserve failures and recover an interrupted request without overwriting a delivered result.
8. Once outputs and pending recovery evidence are accounted for, stop the owned runtime and verify its absence. Report inputs and weights by hash, model version, seeds, results, exceptions, cloud backups, and cleanup status. Keep prediction completion distinct from any downstream biological validation.

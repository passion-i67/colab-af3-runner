# Colab setup and recovery

## Bind the session

Find the `colab` executable in the shell the project uses. On Windows, a project may run it inside WSL. Use the same installation and Google account throughout a session. Inspect the installed CLI help before using the following sequence:

```sh
colab --auth=oauth2 sessions
colab --auth=oauth2 usage
colab --auth=oauth2 new --session "$session" --gpu L4
colab --auth=oauth2 status --session "$session"
colab --auth=oauth2 exec --session "$session" --file kernel_probe.py --timeout 60
```

Set a fresh `session` identifier before requesting the runtime. A kernel probe can print the Python version and run `nvidia-smi`. L4 was used in the verified workflow. Choose a different GPU only if it is appropriate for the pinned implementation, input size, and user's compute budget.

Use the CLI's official authentication flow if there is no working session. Keep OAuth URLs with temporary parameters, codes, cookies, and credential files out of logs. If the provider requires a password or two-step verification, hand it to the user. If a waiting GPU would incur charges without useful work, report the wait, use a bounded authorization window, and stop the owned session if the window expires.

## AF3 environment and storage

Use a pinned deployment for the selected runner. Record the implementation and commit, Python and JAX versions, GPU and backend, relevant inference flags, and weight checksum. A native Colab adapter and the upstream AlphaFold 3 runner can expose different flags. Check the installed runner's help and validate the environment on the selected GPU before launch.

The verified L4 profile used `alphafold3-colabfold==3.1.18` with the [matching adapter entrypoint](https://github.com/sokrypton/alphafold3/tree/v3.1.18), Python 3.13.16, JAX and JAXLIB 0.11.1 with CUDA 13 plugins, and Tokamax 0.0.11. Use the project's accepted bootstrap for this environment when available. Independently recreate and probe the environment when using a different runtime image or installer.

After parser and featurization checks pass, the corresponding adapter invocation for supplied inline MSA has the following shape. Supply absolute runtime paths and the requested sample and recycle counts through the job configuration. Preserve any other pinned inference flags from that configuration.

```sh
XLA_PYTHON_CLIENT_PREALLOCATE=false \
XLA_FLAGS=--xla_gpu_enable_triton_gemm=false \
  "$af3_python" "$af3_entrypoint" \
  --model=alphafold3 --json_path="$input_json" --model_dir="$weights_dir" \
  --output_dir="$result_dir" --run_inference=true --run_data_pipeline=false \
  --use_msa_server=false --resolve_msa_overlaps=false --download_weights=false \
  --flash_attention_implementation=xla --num_diffusion_samples="$samples" \
  --num_recycles="$recycles"
```

- When approved weights are available from an official source, downloading from the runtime reduces dependence on host upload speed and Drive mounting. Respect model-weight access and use terms. Use a temporary resumable download and promote it only after size and SHA-256 verification. Honor the user's network requirements.
- When using Drive for weights or results, mount it on the target runtime through the project's established flow, then verify the mount and access to the exact source or destination. A Drive connector used for post-download backup is a separate operation.
- Treat runtime storage as temporary. Download required results as each request finishes and preserve verification and launch receipts locally before cleanup.

## Recover a disconnected or slow run

| Observation | Action |
| --- | --- |
| Allocation timed out or returned a service error | Query the same session ID and active assignments. Retry with bounded backoff only after confirming the allocation is absent and accounting for cost. |
| Kernel or TLS connection cannot be verified | Reconnect and probe the same runtime. Preserve the exception. A network response alone cannot determine whether a remote inference started. |
| Launch response was lost | Query the task's durable remote status and heartbeat. Use the same task ID for observation. A repeated launch transport is appropriate only when a verified lock or idempotency mechanism prevents a second worker from starting. |
| Runtime stopped and outputs were not delivered | Confirm the old session is absent, archive the previous attempt, and check for recoverable outputs. For an eligible retry, use a fresh session-scoped task ID and preserve the input, seeds, and settings. |
| Large inline input stalls in transfer | Compress it and send bounded, verified chunks through the existing authenticated session. Reassemble atomically and verify the original JSON SHA-256. Reuse byte-identical MSA payloads by hash within the session where helpful. |

Use the installed CLI's exact stop syntax, normally `colab --auth=oauth2 stop --session "$session"`. Check `sessions` or `status` afterward to confirm the task's assignment is gone. Other active sessions are independent.

References: [Google Colab CLI](https://github.com/googlecolab/google-colab-cli), [AlphaFold 3 installation and weight access](https://github.com/google-deepmind/alphafold3/blob/main/docs/installation.md), [AlphaFold 3 performance guidance](https://github.com/google-deepmind/alphafold3/blob/main/docs/performance.md).

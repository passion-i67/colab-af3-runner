# Input and result acceptance

## Precomputed inputs

- Verify the input bundle and manifest before editing working copies. Use the runner's supported AlphaFold 3 JSON dialect and version.
- Map paired and unpaired A3M payloads to the specified chains. Compare the first A3M sequence with the corresponding JSON query. Apply query-only MSA to a chain only when the input specification calls for it.
- When converting to inline fields, remove incompatible path alternatives and retain the original chain IDs, copy counts, ligands, templates, seeds, and requested sampling settings. Confirm that every referenced ligand component exists in the runtime CCD and that the selected runner resolves it before inference.
- Validate every request with the same parser or implementation intended for inference. Compare the original working-copy SHA-256 with the file received by the runtime. Check Stage A exit status and relevant feature validation before launch. Keep external MSA searches off when the user supplies the alignments for inference only.

## Durable results

- Tie each result to the request ID, exact input hash, model version, seed, and recorded task ID. A success message alone is insufficient.
- Derive expected output names and counts from the pinned runner. Typical outputs include the ranked model CIF, confidence and summary JSON, ranking scores, input data JSON, and per-seed and per-sample structure and confidence files. Check each configured seed and sample.
- Inventory non-empty relative paths, byte sizes, and SHA-256 hashes. When writes may still be settling, compare two consecutive remote inventories. Download to a temporary destination and verify its inventory before promoting it to the final path. Reject archive entries that escape the intended directory.
- Record separate outcomes for inference, local delivery, backup, and cleanup. Preserve source inputs, previous outputs, and failed-attempt receipts. Make a replacement attempt only after reconciling the old task and any recoverable outputs.
- For optional Drive backup, reuse a confirmed private destination. Archive each verified request separately, look for an existing copy before retrying an uncertain upload, and keep a receipt with the uploaded file ID, filename, and reported size. Compare a remote checksum when the connector exposes it.

References: [AlphaFold 3 input specification](https://github.com/google-deepmind/alphafold3/blob/main/docs/input.md), [AlphaFold 3 output specification](https://github.com/google-deepmind/alphafold3/blob/main/docs/output.md).

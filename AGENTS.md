# CLOVERWOOD-STUDIO-PREVIEW — AGENT RULES

## Local Mac build/test cleanup — OWNER LOCKED 2026-09-09
- This rule is mandatory for every agent and every local Mac build, test or CI run in this repository.
- Before a run, measure free disk space. For Xcode, 3D or other large runs, do not start below 20 GiB free; clean stale repo-scoped disposable artifacts first.
- Every run MUST route regenerable outputs to a unique disposable repo-scoped root under `${TMPDIR:-/tmp}/cloverwood-ci/<repo>/<run-id>/` whenever the tool supports an explicit path.
- Xcode must use `-derivedDataPath <run-root>/DerivedData`; SwiftPM must use `--scratch-path <run-root>/swiftpm`; test results, temporary logs, caches, temp clones/worktrees and run-only captures must also stay under the run root when practical.
- Register cleanup with shell `trap` / `finally` semantics where supported so cleanup runs after PASS, FAIL, cancellation or early exit.
- Immediately after each run, delete all run-generated disposable artifacts and verify the run root no longer exists.
- A run is not `PASS`, `READY`, or merge-ready if cleanup was skipped, cleanup verification failed, or the disk-space receipt is missing.
- Preserve source, authored assets, locked references, intentional QA evidence/receipts, signed/release archives and user files. Never delete those as cleanup.
- Never delete whole simulators, runner installations, repositories, keychains, credentials or broad user/global caches merely to free space.
- After an interrupted/crashed run, the next agent must sweep stale repo-scoped disposable run directories before starting new work.
- Record free-space-before, free-space-after, cleanup status and any intentionally retained evidence path in the run receipt.

## Mandatory local-run wrapper — OWNER LOCKED 2026-09-09
- Every local Mac build, test or CI command MUST run through `bash scripts/cloverwood-local-run.sh -- <command> [args...]` so disposable storage, disk gating, signal-safe cleanup and the cleanup receipt are enforced.
- For Xcode, 3D or other heavy Mac workloads set `CLOVERWOOD_EXCLUSIVE_MAC=1` so two heavy jobs cannot compete for CoreSimulator/build storage on the same Mac.
- Do not bypass the wrapper to obtain PASS/READY. If a tool technically cannot be nested under the wrapper, reproduce the same disposable run-root + trap/finally + deletion verification contract and emit an equivalent cleanup receipt.

# ADR-0028: Output publication with disposable control state

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-10-16 |
| Owner | Tech lead, Security reviewer, Product owner |
| Phase | P0.1 (decision), P3.8 (contract tests) |
| Related | PRD §16.4, §17, §17.1, §17.2, §22 (AC 19); technical-spec §11.1, §18; ADR-0002, ADR-0010 |

## Context

ADR-0010 and technical-spec §11.1 defined a seven-step state machine: an advisory lock per target, a staging directory, one retained backup, a recovery journal with fsync points, and recovery that restores or completes an interrupted publication on the next run. P0.7 had to prove it with injected crashes on APFS and ext4.

The backup and journal protect the previous output, but the previous output is not irreplaceable data. Every FlowFrame output is a deterministic function of source files that the build never modifies (PRD §16.1, ADR-0010 item 2). Its loss is repaired by running the same build again, and committed outputs can also be restored from version control. The parts of ADR-0010 that protect user data are the refusal of forbidden output directories and the refusal to overwrite anything that is not a FlowFrame artifact. Those parts do not depend on the backup or the journal.

The advisory lock is different: it costs little, and without it a build that cleans up leftover state could delete the staging directory of a concurrent build.

## Alternatives considered

- **Keep ADR-0010 as accepted.** Restores the previous output after an interrupted publication, at the cost of the journal, recovery logic, crash-window tests and a P0 spike.
- **Disposable control state without a lock: stage, verify, swap, delete; recover by rebuilding.** Smallest design, but concurrent builds can delete each other's staging and fail with confusing execution errors. Rejected.
- **Disposable control state under the existing advisory lock.** Adopted.
- **Replace the three files one by one inside the target.** Leaves a mixed artifact set after a crash, which the ownership check would then reject as modified. Rejected.

## Decision

1. ADR-0010 items 2 (forbidden output directories), 3 (supported filesystems), 4 (ownership recognition), 5 (single-file overwrite protection) and 6 (control directory is untracked) remain in force unchanged. Item 1 remains in force except for the state machine it references, which this ADR replaces.
2. The private directory `<parent>/.<name>.flowframe/` remains the only path outside the target that a build writes. It holds a FlowFrame ownership marker file, the lock file and disposable state only. A build deletes leftover state in it only while holding the lock and only when the marker is present.
3. A build takes the per-target advisory lock (POSIX `fcntl`, ADR-0002 item 7) before touching the control directory and holds it until publication ends. A competing writer fails immediately with a stable execution diagnostic (exit 4). The lock file is never deleted to break a held lock.
4. A build prepares and verifies the complete artifact set in a staging directory with a name unique to the run. Validation and render failures leave the target untouched and remove only that staging directory.
5. Publication: if the target exists and passes the ownership check, rename it into the control directory. Then rename staging to the target, and delete the old output. If the second rename fails, the build renames the old output back once and reports an execution error (exit 4). There is no journal, no retained backup and no restore on a later run.
6. An interrupted publication can leave the target missing, never mixed or partially written. The next build removes leftover control state under the lock and treats a missing target as a first publication. The diagnostic for an interrupted or failed publication says to rerun the build. It does not promise that the previous output was restored.

## Consequences

- Removed from v0.1: the recovery journal and its fsync points, backup retention and journal-aware recovery, and the P0.7 crash-window spike. Publication behavior is proven by contract tests in P3.8.
- Changed public behavior: after an interrupted publication, the previous output may be missing until the next build. AC 19 becomes "a render or validation failure never modifies the previous output; an interrupted or failed publication never leaves a mixed artifact set and is repaired by rerunning the build; forbidden output directories and non-artifact overwrites are refused".
- If the sources became invalid after an interrupted publication, the previous output can be restored only from version control. Documentation states this.
- The advisory lock and its exit-4 diagnostic are unchanged, so ADR-0002 item 7 is unaffected.
- Documents changed together with this ADR: README "Planned CLI"; PRD §17, §17.1, §17.2, §22 AC 19; technical-spec §11.1, §18, §19, §22; implementation-plan P0.7, P0 exit gate, P3, P6.3, P6.5, P8.3 and the traceability matrix; amendment note in ADR-0010.

## Verification

- Contract tests for every forbidden output directory and for refusing non-artifact content (unchanged from ADR-0010).
- An interruption injected before, between and after the two renames leaves either the complete old target, no target or the complete new target, and the next build publishes a complete set.
- A failure of the second rename restores the old output and returns exit 4.
- A control directory without the ownership marker is never cleaned.
- A second build into a locked target fails immediately with exit 4 and leaves the first build's staging intact.

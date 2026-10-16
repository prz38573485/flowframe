# ADR-0017: Command stage matrix

| Field | Value |
|---|---|
| Status | Accepted (amended by ADR-0029) |
| Date | 2026-09-30 |
| Owner | Tech lead, Product owner |
| Phase | P1 |
| Related | PRD §15.1, §17; technical-spec §8, §16 |

## Context

The PRD said command-specific stage boundaries were defined in §17, but §17 stated only that `validate` does not invoke D2. It was unclear whether `validate` runs selection, IR invariants or presentation policy, which determines whether `validate`, `review` and `build` agree.

## Decision

All commands call one validation service. Stages run as follows:

| Stage | `validate` | `review` (default modes) | `compile` | `render` | `build` |
|---|---|---|---|---|---|
| YAML loading and schemas | ✓ | ✓ | ✓ | config and theme only | ✓ |
| Cross-document semantics | ✓ | ✓ | ✓ | — | ✓ |
| Selection and family rules (with a View) | ✓ | ✓ | ✓ | — | ✓ |
| Presentation and asset policy (sanitizer, contrast, footer) | ✓ | ✓ | ✓ | theme and fonts | ✓ |
| Family IR and FFI invariants (with a View) | ✓ | ✓ | ✓ | — | ✓ |
| D2 generation and lint | — | — | ✓ | lint of the input | ✓ |
| `d2 validate` and render | — | — | — | ✓ | ✓ |
| Composition and final SVG verification | — | — | — | body verification | ✓ |
| Manifest | — | — | — | — | ✓ |

Rules:

1. For the same inputs, `validate --model M --view V`, default `review` and `build` report identical diagnostics for every stage up to and including IR.
2. `validate` without `--view` runs loading, schemas, model semantics and configuration/theme policy.
3. `validate --theme` runs theme schema and asset policy only.
4. `validate --all` and `review --all` are post-MVP; in v0.1, `build --all` is the only whole-project command.
5. No command writes files except `compile`, `render` and `build`.

## Consequences

- Selection must be implemented before `validate` is complete (plan P2.7 before P2.8).

## Verification

- A differential test over the whole corpus compares diagnostics from `validate`, `review` and `build` up to the IR stage.

## Amendments

- 2026-10-16, [ADR-0029](0029-v0-1-command-set.md): The `render` column is dropped. The `review` column applies from P7: `review` runs the same stages as `validate` with a View before source-conformance. Rule 1 and the differential test compare `validate` with `build`, and from P7 also with `review`. Under rule 5 only `compile` and `build` write files.

# ADR-0030: One owning document per normative rule

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-10-16 |
| Owner | Product owner, Tech lead |
| Phase | P0.1 |
| Related | PRD §1, §25; technical-spec §1; implementation-plan §1; ADR-0001 |

## Context

ADR-0001 removed duplicated lists of open decisions. The normative content itself is still duplicated: the same rule is stated in full, or in a slightly different form, in two to four places. Examples in the current documents:

| Topic | Stated in |
|---|---|
| D2 executable resolution, pinning and checksum verification | README "Planned CLI", PRD §17, technical-spec §3, ADR-0006 |
| Output publication, control directory and recovery | README "Planned CLI", PRD §17.2, technical-spec §11.1, ADR-0010 |
| Manifest fields, resources and asset-processing records | PRD §16.1, technical-spec §15, ADR-0013, ADR-0020 |
| Decoration bands and width formula | PRD §13.6, technical-spec §12.3, ADR-0005 |
| Logo sanitizer processing | PRD §16.5, technical-spec §13.2, implementation-plan P0.5 |
| Exit codes, categories and precedence | PRD §17.1, technical-spec §16, ADR-0016 |

Each copy has to be kept consistent by hand. Several recent changes consisted mainly of re-aligning copies after review, and every further amendment repeats that cost. The PRD also carries implementation detail (processing order, internal file names, hashing layouts), so product readers have to read through design text to find behavior.

Two related gaps exist in the decision process:

- ADR-0001 item 3 requires every P0 ADR to be **Accepted** before the P0 exit gate, so a P0 ADR that is rejected after its experiment would block the gate.
- The register says that changing a decision requires a superseding ADR, but not how an ADR that replaces only some items of an accepted ADR is recorded.

## Alternatives considered

- **Keep the current layout and rely on review to catch drift.** This is how the documents work today, and it is where the drift came from.
- **Give every normative rule one owning document; other documents link to it with at most one sentence of summary.** Adopted.
- **Merge PRD and technical specification into one document.** Removes the duplication but mixes product decisions with implementation design, which ADR-0001 roles keep separate (Product owner and Tech lead).
- **Reduce ADRs to links.** Removes the last copy, but an ADR then no longer records what was decided at the time, and the history of a rule is lost when the owning document changes. Rejected.

## Decision

1. **PRD** owns externally observable behavior: inputs and their schemas, vocabulary, selection semantics, CLI commands and options, exit codes, outputs and their contracts, accessibility and security guarantees as promises, and acceptance criteria. It does not describe processing order, internal modules, algorithms or internal resource file names.
2. **Technical specification** owns implementation design: module boundaries, processing pipelines, algorithms, internal resources, and how each PRD guarantee is achieved.
3. **Normative rule files** under `rules/` (for example `rules/svg-logo-profile-v1.md`, `rules/accessibility-rules.md`) own detailed matrices that the PRD promises; the PRD states the promise and links to the file.
4. **ADRs** own the rationale, the rejected alternatives, the values of their open parameters and the decision as worded when it was accepted. The decision text is a point-in-time record and is not edited afterwards. The current normative wording lives in the owning document; if the two differ, the owning document prevails, and such a difference may exist only because a later ADR amended or superseded the decision.
5. **Implementation plan** owns sequencing, work packages and acceptance gates. It links to rules and does not restate them.
6. **README** is an overview for new readers. It contains no normative rules and links to the owning document.
7. A summary sentence that points to an owner is allowed. A second full statement of a rule, a table or an algorithm is not.
8. An ADR that replaces only some items of an accepted ADR leaves the older ADR Accepted. The older ADR's status names the amending ADR, and a dated amendment note at its end lists the replaced items. The older decision text stays unchanged.
9. ADR-0001 item 3 is amended: every ADR with phase P0.x MUST be **Accepted or Rejected** before the P0 exit gate.

## Consequences

- One follow-up change, with no behavior change, moves every duplicated rule to its owner and replaces the copies with links. The table above is the minimum scope of that change. It is made after ADR-0026 and ADR-0031 are decided, so that text about to be removed is not moved first.
- The PRD becomes noticeably shorter. Implementation detail moves to the technical specification, for example the sanitizer processing order, the publication state machine and the decoration formula.
- A future amendment touches one document per rule, plus links when a section moves.
- ADR-0027, ADR-0028 and ADR-0029 already use the amendment notes of item 8.
- Documents changed together with this ADR: ADR register (`docs/adrs/README.md`); implementation-plan P0 exit gate; amendment note in ADR-0001.

## Verification

- After the follow-up change, a reviewer can take each topic in the table and find exactly one full statement of it.
- The documentation check of ADR-0001 is extended so that every cross-document link to a PRD or technical-specification section resolves.

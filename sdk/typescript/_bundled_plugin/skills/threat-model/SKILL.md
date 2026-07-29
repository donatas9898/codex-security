---
name: threat-model
description: Use when Codex is already in the threat-modeling phase of a security scan, the user explicitly invokes $threat-model, or the user explicitly asks to create, update, or persist a repository threat model. Do not use as the primary trigger for full PR, commit, branch, patch, or repository scans.
---

# Security Threat Model

## Objective

Establish the repository-scoped threat model at the path defined in `../../references/scan-artifacts.md`. Reuse a cached model only when its final `Repository` and `Version` lines match the current target and its authoritative guidance still matches the current resolved security policy.

`AGENTS.md` or resolved `SECURITY.md` guidance can be that authoritative source when it is sufficiently specific about the repository's product surfaces, trust boundaries, attacker-controlled inputs, assumptions, or security scan guidance to serve as the threat model.

If no threat model is provided, generate a repository-scoped threat model to be used in future bug discovery. The threat model should holistically cover the entire repository and should make it obvious:

- what assets or privileges matter
- what trust boundaries exist
- what inputs are attacker-controlled
- what invariants the code must preserve
- what repository-wide failure modes would matter most

## Artifact Resolution

The path references in this skill are the default locations for this phase.
If the user explicitly provides a different path for a required input or output, use the user-provided path instead of the corresponding default path referenced in this skill.
If a required input is still missing, stop and ask the user for it before continuing.
Use the shared scan artifact path conventions in `../../references/scan-artifacts.md`.

## Workflow

1. Resolve `target_id`, the current version (revision for an immutable Git tree, snapshot digest otherwise), and the repository-scoped threat model path using `../../references/scan-artifacts.md`.
2. Before inspecting repository source, reusing cached guidance, or generating a threat model, read `../../references/security-guidance.md` and the current policy resolved for the scan target. Resolve the policy first if the coordinator did not supply it.
3. If the repository-scoped threat model exists, reuse it only when its final `Repository` and `Version` lines match the current target and, when authoritative security guidance is available, the cached threat-model body matches that guidance without alteration. Regenerate the repository-scoped threat model when the applicable policy or provided guidance has changed. Never modify a completed scan's sealed artifacts or historical threat model.
4. If a threat model or authoritative security scan guidance is provided or referenced:
   - preserve it unchanged as the threat model body
   - treat that body as the only threat model source of truth
   - do not expand, summarize, or reinterpret the body
   - `AGENTS.md` is acceptable here when it is clearly being used as the security scan guidance or threat model source for this scan and is sufficiently repository-specific to stand in for a threat model
5. Otherwise, generate a repository-scoped threat model using the checklist below.
6. Before finalizing this phase, sanity-check that:
   - the threat model is repository-scoped rather than being centered around any specific scan target
   - it describes repository-wide primary product or runtime surfaces and trust boundaries before covering any narrower examples
   - any vulnerability-class discussion is about repository-context classes, not findings about any current diff
7. Append the exact `Repository` and `Version` lines from `../../references/scan-artifacts.md`, then write the threat model to the repository-scoped path.

## Threat Model Generation Guidance

Generate and structure the threat model using `references/threat-model-guidance.md`.

## Hard Rules

- A provided threat model or authoritative security scan guidance is authoritative. Keep its body unchanged and append only the required cache footer.
- Never reuse a cached threat model after its applicable `SECURITY.md` or provided authoritative guidance has changed, even if the target identity and Git revision are unchanged.
- Preserve completed scan artifacts and their historical threat models. Apply updated policy to a new scan.
- Threat model generation must stay at repository scope unless the user explicitly asks for narrower scope.
- Do not turn this phase into findings about any current diff.
- Do not let the current scan target, touched subsystem, or changed directories become the center of gravity for this phase unless the user explicitly asks for that narrower scope.
- In large monorepos, avoid centering `personal/`, `test/`, `tests/`, `docs/`, `examples/`, or one-off developer tooling unless repository evidence shows those are real deployed or privileged workflow surfaces.
- Call out trust boundaries and assumptions explicitly.
- Keep references to vulnerability types at the level of repository-context classes, rather than any diff findings.
- Persist the threat model output to the repository-scoped threat model path from `../../references/scan-artifacts.md`.

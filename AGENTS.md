# AGENTS.md

## Scope

These instructions apply to the whole `carla-pedestrians` repository unless a more specific `AGENTS.md` exists in a subdirectory.

This is a research fork. Prefer verified, minimal, reversible changes over broad cleanup or speculative redesign.

## Project purpose

This repository is the umbrella CARLA pedestrian-simulation project used to investigate generation of a PedSynth++-style synthetic dataset for an Aalto University research project on **offline pedestrian-behavior annotation**.

The broader research question is whether synthetic pedestrian-behavior supervision can reduce the amount of real human labeling needed on datasets such as LOKI. The target behavior ontology and final model inputs are still being defined. This repository is currently about **understanding and, if needed, extending synthetic data generation**, not implementing the final behavior model.

Relevant paper:

- Riaz, Wielgosz, López Peña, *ARCANE-PedSynth: Synthetic Multi-Pedestrian Datasets with Behavioural Crossing Annotations* (2026), arXiv:2605.24950.

## Repository structure

This is an umbrella repository with Git submodules. Inspect `.gitmodules` before making assumptions.

The main submodule relevant to current work is:

- `pedestrians-scenarios/` — PedSynth++-style scenario and dataset generation.

The current generator of interest is the `free_drive_front_cam_v2` path under `pedestrians-scenarios`. Verify exact paths in the checked-out submodule before editing.

Other submodules are dependencies and should not be changed unless the task specifically requires it.

## Submodule discipline

- Do not update unrelated submodules.
- Do not replace pinned revisions with arbitrary latest upstream commits.
- Changes to `pedestrians-scenarios` must be committed in that repository first; then update the parent repository's submodule pointer in a separate parent commit.
- Preserve the distinction between the upstream repositories and our forks/branches.
- Do not modify `.gitmodules` merely to make a local remote convenient.
- Before substantial work, report the checked-out commit/branch of the relevant submodule and whether it contains the current PedSynth++ generator.

## Current research state

The authors provided a Hugging Face PedSynth++ release containing 533 clips. Local inspection found that its per-clip annotations include items such as:

- RGB-linked 2D pedestrian boxes,
- pedestrian/track identifiers,
- behavior labels,
- crossing labels,
- distance-to-ego,
- visibility fields.

However, that local release did **not** contain explicit pedestrian 3D positions/3D boxes, and no `.ply`, `.bin`, or `.npz` LiDAR/DVS files were found. Its inspected sensor metadata marked LiDAR and DVS disabled.

The current arXiv paper describes modalities beyond what appears in that release. Treat this as an unresolved paper/code/release mismatch, not as something to silently reconcile.

The practical objective is **not bit-for-bit reproduction of the original 533 clips**. The objective is to determine whether this public framework can generate a scientifically useful PedSynth++-style dataset for our downstream behavior-labeling research.

Potentially useful fields may include RGB, stable pedestrian IDs, 2D boxes, metric pedestrian motion/3D trajectories, ego motion, behavior labels, pose, road context, or raw sensors. **This is not yet a finalized modality contract.**

## Research rules

Always distinguish among:

1. what the paper reports;
2. what the checked-out code actually implements;
3. what an inspected released dataset actually contains;
4. what CARLA makes available internally but the exporter does not save;
5. our hypotheses or proposed extensions.

Do not present one category as another.

In particular:

- Do not assume simulator-private state should become a model input.
- Do not assume LiDAR/DVS are needed merely because the paper mentions them.
- Do not assume exact CARLA actor state is equivalent to what a real dataset can provide.
- Do not claim exact reproducibility unless seeds, versions, generation parameters, success/failure selection, and post-processing are all known.
- Keep observable behavior distinct from future intention/prediction.
- Preserve uncertainty when source behavior states or release semantics are ambiguous.

## Immediate workflow

Until this phase is explicitly closed:

1. inspect before editing;
2. understand the existing generation path and annotation schema;
3. get the provided Docker/CARLA setup running;
4. generate **one small unmodified test clip**;
5. inspect its files and annotation schema;
6. compare it with the Hugging Face release and the paper;
7. only then propose the smallest dataset-generation extension needed for the research.

Do **not** launch a hundreds-of-clips generation run without explicit approval.

Do **not** implement the downstream behavior model in this repository unless explicitly requested.

## Environment

The existing project is Docker-oriented. The pinned CARLA server setup uses CARLA 0.9.13.

Use Docker for the actual CARLA runtime unless there is concrete evidence that a different setup is required.

A Conda environment may be used for repository-side analysis, plotting, schema inspection, lightweight scripts, or other tooling that does not need the CARLA runtime. Do not duplicate the full Docker/CARLA dependency stack in Conda without a reason.

Before adding dependencies:

- check whether the dependency already exists in a submodule/container;
- prefer the existing project setup;
- explain why a new host/Conda dependency is required.

## Coding guidance

- Make the smallest change that answers the current research question.
- Preserve existing behavior unless the task explicitly changes it.
- Avoid unrelated refactors while establishing reproducibility.
- Prefer clear, explicit data fields and units over implicit conventions.
- When adding exported quantities, document coordinate frame, units, timestamp/frame alignment, and provenance.
- Maintain stable pedestrian/frame identifiers.
- Avoid hidden post-processing that changes labels or coordinates without recording it.
- Add comments for non-obvious CARLA coordinate/sensor conventions, not for obvious Python.
- Do not add abstractions for hypothetical future modalities.

## Validation

For code changes, use the narrowest meaningful validation first:

- syntax/import checks;
- existing tests, if relevant;
- deterministic schema/unit tests where possible;
- one short CARLA generation run for generator changes.

For generated data, inspect at least:

- expected files exist;
- annotation rows align with frame IDs;
- pedestrian IDs remain stable within a clip;
- units and coordinate frames are explicit;
- no missing/duplicated timestamps or rows were introduced;
- new fields agree with simple independent checks where possible.

Never report a test or CARLA run as successful unless it was actually run. If the environment prevents validation, state exactly what was not tested.

## Data and repository hygiene

- Never commit generated datasets, videos, sensor dumps, model weights, credentials, or machine-specific secrets.
- Keep large outputs outside the Git repository.
- Do not commit local `.env` files containing machine-specific paths or credentials.
- Do not overwrite the downloaded Hugging Face dataset; treat it as reference input.
- Small synthetic fixtures for tests are acceptable only if intentionally created and genuinely useful.

## Git workflow

Current research work should remain isolated on the dedicated development branch unless instructed otherwise.

- Do not force-push, rebase shared branches, or rewrite history unless explicitly requested.
- Commit submodule changes in the submodule first.
- Commit parent submodule-pointer updates separately.
- Keep commits focused enough that experimental generator changes can be reverted cleanly.

## Before proposing a substantial extension

First provide:

1. the observed limitation in the current generator/output;
2. the downstream research need it blocks;
3. the minimum proposed extension;
4. the exact exported representation, including units and coordinate frame;
5. how it can be compared with real target datasets such as LOKI;
6. how it will be validated on a tiny run.

Do not turn a plausible extension into a project commitment without this analysis.

# Shawn Life OS — Documentation & Agent Handoff

This repository is the canonical home for Shawn Life OS documentation, architecture decisions, validation reports, source manifests, and agent handoff/progress.

## Repository split

- `lazerta/shawn-life-os` — application source code, tests, migrations, build/CI, release tags.
- `lazerta/shawn-life-os-docs` — specs, architecture, handoff, validation reports, research, and operational continuation state.
- Google Drive — large binary artifacts, dependency bundles, ZIPs/JARs, and archival copies that do not belong in Git.

## Start here

1. Read [`handoff/AGENT_HANDOFF_PROGRESS_AND_WORKAROUNDS.md`](handoff/AGENT_HANDOFF_PROGRESS_AND_WORKAROUNDS.md).
2. Check the latest validation report under `validation/`.
3. Use the code repository as the source-of-truth for implementation state.

## Handoff rule

Any meaningful change to architecture, validation maturity, known blockers, workarounds, accepted fixes, baseline commit/tag, or next-agent action must update the handoff in this repository.

Git history is the authoritative change log for handoff/documentation from this point forward. Google Drive remains an archive/large-artifact workspace, not the sole canonical handoff store.

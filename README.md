# Shawn Life OS — Documentation Repository

This repository stores versioned, relatively stable Shawn Life OS documentation: specifications, architecture decisions, validation reports, release notes, research summaries, and source manifests.

## Repository split

- `lazerta/shawn-life-os` — canonical application source code, tests, migrations, build/CI, and release tags.
- `lazerta/shawn-life-os-docs` — stable/versioned specs, architecture decisions, validation reports, research, and documentation snapshots.
- Google Drive `Shawn Life OS - Agent Workspace/lib/AGENT_HANDOFF_PROGRESS_AND_WORKAROUNDS` — canonical **live handoff / progress / workaround process document**.
- Google Drive workspace — large binary artifacts, dependency bundles, ZIPs/JARs, and operational working files that do not belong in Git.

## Source-of-truth rules

1. **Code state:** `lazerta/shawn-life-os`.
2. **Live agent handoff / progress / current workaround state:** Google Drive handoff document.
3. **Stable documentation and validation snapshots:** this repository.
4. **Large release/validation binaries:** Google Drive.

The frequently changing handoff is intentionally not mirrored on every small update into Git. When a milestone stabilizes, a report/spec snapshot may be committed here for durable version history.

# Shawn Life OS — Documentation Repository

This repository stores versioned, relatively stable Shawn Life OS documentation: specifications, architecture decisions, validation reports, release notes, research summaries, source manifests, and long-lived engineering governance.

## Start here

- `ENGINEERING_PRINCIPLES.md` — long-term project constitution: Document First, Open Source First, Adapter/Port First, Code First / Model Last, Spec/Test/Real-Data Driven, Evidence Before PASS.
- `ARCHITECTURE_INVARIANTS.md` — durable architecture invariants, including deterministic facts, immutable + append-only history, provenance, rebuildable projections, and code pre-commit regression rules.

## Repository split

- `lazerta/shawn-life-os` — canonical application source code, tests, migrations, build/CI, and release tags.
- `lazerta/shawn-life-os-docs` — stable/versioned specs, architecture decisions, validation reports, research, documentation snapshots, and durable governance.
- Google Drive `Shawn Life OS - Agent Workspace/lib/AGENT_HANDOFF_PROGRESS_AND_WORKAROUNDS` — canonical live handoff / progress / workaround process document.
- Google Drive workspace — large binary artifacts, dependency bundles, ZIPs/JARs, and operational working files that do not belong in Git.

## Source-of-truth rules

1. **Code state:** `lazerta/shawn-life-os`.
2. **Live agent handoff / progress / current workaround state:** Google Drive handoff document.
3. **Stable documentation, durable engineering principles, architecture invariants, and validation snapshots:** this repository.
4. **Large release/validation binaries:** Google Drive.

The frequently changing handoff is intentionally not mirrored on every small update into Git. When a milestone stabilizes, a report/spec snapshot may be committed here for durable version history.

Documentation-only commits do not require the application regression suite. Any commit that can affect executable behavior, migrations, build/CI/runtime configuration, or packaging is code-bearing and must pass the full pre-commit regression gate defined in `ARCHITECTURE_INVARIANTS.md`.
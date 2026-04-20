# Claude Code Skills

Custom skills (slash commands) for [Claude Code](https://claude.ai/code).

## Skills

### `/investigation`
Scaffolds structured investigations in `scratch/` for empirical research. Covers tracing, architecture archeology, bug investigation, technical exploration, and design research.

### `/sleap`
Reference for working with SLEAP pose estimation files (`.slp`, `.h5`, `.pkg.slp`). Covers the `sleap-io` 0.5.x Python API, common patterns, and known pitfalls.

## Installation

Copy any skill folder into `~/.claude/skills/` (global) or `.claude/skills/` (project-specific):

```bash
cp -r sleap ~/.claude/skills/
cp -r investigation ~/.claude/skills/
```

Then use `/sleap` or `/investigation` in Claude Code.

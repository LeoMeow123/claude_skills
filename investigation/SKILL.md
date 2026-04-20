---
name: investigation
description: >
  Scaffolds a structured investigation in scratch/ for empirical research and documentation.
  Use when the user says "start an investigation" or wants to: trace code paths or data flow
  ("trace from X to Y", "what touches X", "follow the wiring"), document system architecture
  comprehensively ("document how the system works", "archeology"), investigate bugs
  ("figure out why X happens"), explore technical feasibility ("can we do X?"), or explore
  design options ("explore the API", "gather context", "design alternatives").
  Creates dated folder with README. NOT for simple code questions or single-file searches.
---

# Set up an investigation

## Instructions

1. Create a folder in `{REPO_ROOT}/scratch/` with the format `{YYYY-MM-DD}-{descriptive-name}`.
2. Create a `README.md` in this folder with: task description, background context, task checklist. Update with findings as you progress.
3. Create scripts and data files as needed for empirical work.
4. For complex investigations, split into sub-documents as patterns emerge.

## Investigation Patterns

These are common patterns, not rigid categories. Most investigations blend multiple patterns.

**Tracing** - "trace from X to Y", "what touches X", "follow the wiring"
- Follow call stack or data flow from a focal component to its connections
- Can trace forward (X → where does it go?) or backward (what leads to X?)
- Useful for: assessing impact of changes, understanding coupling

**System Architecture Archeology** - "document how the system works", "archeology"
- Comprehensive documentation of an entire system or flow for reusable reference
- Start from entry points, trace through all layers, document relationships exhaustively
- For complex systems, consider numbered sub-documents (01-cli.md, 02-data.md, etc.)

**Bug Investigation** - "figure out why X happens", "this is broken"
- Reproduce → trace root cause → propose fix
- For cross-repo bugs, consider per-repo task breakdowns

**Technical Exploration** - "can we do X?", "is this possible?", "figure out how to"
- Feasibility testing with proof-of-concept scripts
- Document what works AND what doesn't

**Design Research** - "explore the API", "gather context", "design alternatives"
- Understand systems and constraints before building
- Compare alternatives, document trade-offs
- Include visual artifacts (mockups, screenshots) when relevant
- For iterative decisions, use numbered "Design Questions" (DQ1, DQ2...) to structure review

## Figure Generation

When creating figures iteratively, use version suffixes to track iterations:

**Naming Convention:**
- Scripts: `01_panel_A_concordance.v1.py`, `01_panel_A_concordance.v2.py`, etc.
- Outputs: `Fig2A_concordance.v1.png`, `Fig2A_concordance.v2.png`, etc.
- Only remove version suffix when user confirms it's final: `Fig2A_concordance_final.png`

**Why:**
- Keeps iteration history visible
- Easy to compare versions
- Avoids cluttered folders with confusing names like `_clean`, `_new`, `_final2`
- User can always go back to earlier versions

**Example workflow:**
```
01_panel_D_heatmap.v1.py  → Fig2D_heatmap.v1.png   (first attempt)
01_panel_D_heatmap.v2.py  → Fig2D_heatmap.v2.png   (fixed colors)
01_panel_D_heatmap.v3.py  → Fig2D_heatmap.v3.png   (added gradient)
01_panel_D_heatmap.py     → Fig2D_heatmap_final.png (user approved)
```

## Best Practices

- Use `uv` with inline dependencies for standalone scripts; for scripts importing local project code, use `python` directly (or `uv run python` if env not activated)
- Use subagents for parallel exploration to save context
- Write small scripts to explore APIs interactively
- Generate figures/diagrams and reference inline in markdown
- For web servers: `npx serve -p 8080 --cors --no-clipboard &`
- For screenshots: use Playwright MCP for web, Qt's grab() for GUI
- For external package API review: clone to `scratch/repos/` for direct source access

## Workstation Environment

This workstation is accessed remotely via SSH from a laptop. Key specs:
- **CPU**: AMD Threadripper PRO 9995WX (96 cores / 192 threads, up to 5.4 GHz)
- **RAM**: 512 GB DDR5 ECC
- **GPU**: 4x NVIDIA RTX PRO 6000 Blackwell (96 GB GDDR7 each, 384 GB total VRAM)
- **Storage**: 4 TB NVMe + 7.68 TB SATA SSD
- **OS**: Ubuntu 24.04 LTS
- **Python**: use `uv` for environment and package management (not conda, not pip directly)

## Session Management (tmux)

All work happens inside tmux so sessions survive SSH disconnects and laptop sleep.

- **Reconnect**: `tmux attach -t main` (or `tmux attach -t main -d` to force-detach stale clients)
- **New session**: `tmux new -s main`
- **List sessions**: `tmux ls`
- A systemd service auto-starts the `main` tmux session on boot
- When launching long-running processes, use a dedicated tmux window/pane so you can detach and check back later

## GPU & System Monitoring

Before launching GPU work, always check current resource usage.

| Tool | What it shows | Command |
|---|---|---|
| **nvtop** | Per-process GPU/VRAM usage, all 4 GPUs, temps, fans | `nvtop` |
| **btop** | Per-core CPU, RAM, disk I/O, network | `btop` |
| **gpustat** | Quick GPU one-liner with PIDs | `gpustat -p` |
| **nvidia-smi** | Detailed GPU state, driver info | `nvidia-smi` |

**Monitoring tmux layout**: dedicate one tmux window with nvtop (top pane) + btop (bottom pane) for at-a-glance system health.

**Automated GPU logging** (for long jobs):
```bash
nvidia-smi dmon -s pucvmet -d 60 -o DT >> scratch/gpu-log.csv &
```

## Resource Management for Long-Running Jobs

- Use `CUDA_VISIBLE_DEVICES` to pin GPUs per job (e.g., `CUDA_VISIBLE_DEVICES=0,1,2` for inference)
- Use `nice -n 10 ionice -c 2 -n 6` to lower CPU/IO priority for background inference, keeping interactive work responsive
- Run long inference in a dedicated tmux window, or as a systemd service with `Restart=on-failure` for crash recovery
- Check GPU memory and utilization before launching new work — don't assume GPUs are free
- No hard cgroup separation — resource needs vary per task, allocate flexibly

## Video Inference at Scale

For months-long inference over thousands of videos:

- **Launch pattern**: `CUDA_VISIBLE_DEVICES=0,1,2 nice -n 10 python inference.py --config prod.yaml`
- **Logging**: redirect output with `2>&1 | tee scratch/inference.log` or write structured JSON lines for machine-parseable progress
- **Progress tracking**: write a progress file (e.g., completed video count) that can be checked with `cat` or `tail` without interrupting the job
- **Crash recovery**: design inference scripts to be resumable — track which videos are done and skip them on restart
- **systemd service**: for true persistence across reboots, run inference as a systemd service (see `scratch/2026-02-25-workstation-workflow-upgrade/README.md` for template)

## Important: Scratch is Gitignored

The `scratch/` directory is in `.gitignore` and will NOT be committed.

- NEVER delete anything from scratch - it doesn't need cleanup
- When distilling findings into PRs, include all relevant info inline
- Copy key findings, code, and data directly into PR descriptions
- PRs must be self-contained; don't reference scratch files

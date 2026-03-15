# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ARIS (Auto-claude-code-research-in-sleep) is an automated ML research workflow system built on Claude Code. It enables autonomous idea discovery, iterative review loops, and paper writing for machine learning research.

## Architecture

```
skills/                    # Claude Code skills (26 total)
├── research-pipeline/     # End-to-end workflow orchestration
├── idea-discovery/        # Workflow 1: idea generation → validation
├── idea-creator/          # Brainstorming ideas with external LLM
├── novelty-check/         # Verify idea novelty via literature search
├── research-lit/          # Literature survey (arXiv, Semantic Scholar)
├── research-review/       # External LLM as senior reviewer
├── auto-review-loop/      # Workflow 2: review → fix → re-review (up to 4 rounds)
├── run-experiment/        # Deploy experiments to remote GPU via SSH+screen
├── paper-plan/            # Outline with claims-evidence matrix
├── paper-figure/          # Generate plots from experiment data
├── paper-write/           # Workflow 3: section-by-section LaTeX writing
├── paper-compile/         # Compile LaTeX to PDF
└── ...                    # Additional utility skills

mcp-servers/              # Custom MCP servers for external LLM integration
├── llm-chat/             # Generic OpenAI-compatible LLM chat
└── minimax-chat/          # MiniMax-specific server

tools/
└── arxiv_fetch.py         # CLI tool for arXiv paper search/download
```

## Core Workflows

### Workflow 1: Idea Discovery
```
/idea-discovery "research direction"
  → /research-lit (survey literature)
  → /idea-creator (brainstorm 8-12 ideas, pilot experiments)
  → /novelty-check (verify novelty)
  → /research-review (external LLM feedback)
  → outputs IDEA_REPORT.md
```

### Workflow 2: Auto Review Loop
```
/auto-review-loop "topic"
  → External LLM (GPT-5.4 via Codex MCP) reviews work
  → Claude Code implements fixes
  → Re-review (up to 4 rounds)
  → Stop when score >= 6/10 or max rounds reached
```

### Workflow 3: Paper Writing
```
/paper-write "venue"  # venue: ICLR, NeurIPS, ICML
  → Generate LaTeX structure from paper plan
  → Write each section (Abstract → Introduction → Related Work → Method → Experiments → Conclusion)
  → Generate figures from experiment data
  → Compile PDF
```

## Supported Venues

- ICLR 2026 (`templates/iclr2026.tex`)
- NeurIPS 2025 (`templates/neurips2025.tex`)
- ICML 2025 (`templates/icml2025.tex`)

## Key Skills Reference

| Skill | Purpose |
|-------|---------|
| `/research-lit "topic"` | Search literature (arXiv, local PDFs, Zotero, Obsidian) |
| `/idea-discovery "direction"` | Full idea discovery pipeline |
| `/auto-review-loop "topic"` | Autonomous review iteration |
| `/run-experiment "description"` | Deploy to GPU server via SSH+screen |
| `/paper-figure "plan-or-data"` | Generate publication-quality plots |
| `/paper-write "venue"` | Write LaTeX paper section by section |
| `/paper-compile` | Compile LaTeX to PDF |
| `/feishu-notify "message"` | Send notification via Feishu (optional) |

## State Persistence

Long-running loops (like auto-review-loop) persist state to JSON files:
- `REVIEW_STATE.json` - tracks round, score, pending experiments
- `AUTO_REVIEW.md` - cumulative review log

This allows recovery after context compaction.

## Installation

To use these skills with Claude Code, copy them to the global skills directory:

```bash
# Install all skills globally
cp -r skills/* ~/.claude/skills/

# Or install specific skills
cp -r skills/idea-discovery ~/.claude/skills/
cp -r skills/auto-review-loop ~/.claude/skills/
```

## Configuration Files

- `~/.claude/feishu.json` - Optional Feishu webhook configuration for notifications
- Project `CLAUDE.md` - Should contain remote server info for experiment deployment

## External Dependencies

- Claude Code CLI
- Python 3.x (for MCP servers and tools)
- LaTeX environment (pdflatex, latexmk) - for paper compilation
- SSH access to GPU server - for experiment deployment

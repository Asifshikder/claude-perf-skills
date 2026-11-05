# claude-perf-skills

Three global [Claude Code](https://claude.com/claude-code) skills that use parallel sub-agents.

| Skill | Purpose |
|---|---|
| `/perf-analysis` | Read-only analysis of requirements, implementation, and functionality; evidence-backed report |
| `/perf-plan` | Turn a requirement, prompt, image, or spec into a verified, step-by-step plan file |
| `/perf-execute` | Implement a plan or prompt with parallel agents, then verify and review |

Typical flow: `/perf-analysis <area>` → `/perf-plan <requirement>` → `/perf-execute PLAN-<slug>.md`

## Install

Copy the skill folders into your global skills directory.

macOS/Linux:
```bash
mkdir -p ~/.claude/skills && cp -r skills/* ~/.claude/skills/
```

Windows (PowerShell):
```powershell
New-Item -ItemType Directory -Force "$HOME\.claude\skills" | Out-Null
Copy-Item -Recurse -Force skills\* "$HOME\.claude\skills\"
```

Restart Claude Code (or start a new session) to pick them up.

## License

MIT

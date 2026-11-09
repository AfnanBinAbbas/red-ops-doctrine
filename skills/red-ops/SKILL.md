---
name: red-ops
description: Red team engagement doctrine for CTFs and authorized targets. Use when running or orchestrating offensive security operations (RCE, SQL dump, PII objectives). Loads the deduplicated vuln index, OPSEC doctrine, and engagement playbook on demand.
---

# Red Ops Doctrine

The canonical reference set for red team engagements lives in `~/.config/opencode/redteam/`:

1. **PLAYBOOK.md** — engagement workflow, workspace layout, kill chain, evidence standard, CTF mode.
2. **OPSEC.md** — minimal-noise doctrine (scan hygiene, payload hygiene, post-exploitation cleanup).
3. **VULN_INDEX.md** — deduplicated map of attack classes → single owning agent → exact neuro-sploit playbook on disk (`~/.neurosploit-app/agents_md/vulns/`) and claude-red skill.

## Doctrine (non-negotiable)
- Single scan run per target phase; read checkpoints instead of re-running scans.
- One owner per attack class — never read another agent's playbook.
- Prefer passive/quiet techniques; stop at loud by default.
- Objectives only: RCE, SQL dump, PII (or CTF flags). Prove impact before claiming.

Read the specific file for the class you are attacking. Never bulk-load the 196 neuro-sploit playbooks into context.
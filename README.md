# red-ops-doctrine

The harness brain — canonical doctrine every red-* agent and phase script obeys.

```
doctrine/PLAYBOOK.md     engagement workflow, workspace layout, kill chain,
                         evidence standard, pipeline contract, CTF mode
doctrine/OPSEC.md        minimal-noise doctrine: origin protection, scan hygiene,
                         payload hygiene, post-exploitation cleanup
doctrine/VULN_INDEX.md   deduplicated map: attack class -> single owning agent
skills/red-ops/SKILL.md  opencode skill entry point (loads the three files above)
```

## Install

```bash
mkdir -p ~/.config/opencode/redteam ~/.config/opencode/skills/red-ops
cp doctrine/*.md ~/.config/opencode/redteam/
cp skills/red-ops/SKILL.md ~/.config/opencode/skills/red-ops/
```

## Non-negotiables

- Single scan run per phase — read checkpoints instead of re-running
- One owner per attack class — never bulk-load specialist playbooks
- Prefer passive/quiet; stop at loud by default
- Objectives only; prove impact before claiming (request+response, command output)

Works with `red-agents` (definitions) and `red-conductor` + phase repos (mechanics).
Authorized targets only.

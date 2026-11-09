# OPSEC Doctrine

Every red-* agent practices this. Speed and volume are never excuses to be loud. Default to quiet; go loud only when the phase/objective requires it.

## 1. Origin protection (do first)
- All direct traffic to target infrastructure goes through a channel you control: opencode's outbound via your VPN/VPS/proxy per engagement. Never enage from a naked home IP for anything but CTF VPNs.
- CTF/lab platforms (HTB/THM): route ONLY through their VPN/network. Never scan the lab gateway/router, other students' boxes, or anything outside your assigned target.

## 2. Scan hygiene
- nmap: prefer `-sS` (SYN) over `-sT`; `-T3` default, `-T2` on apparently-guarded hosts; `--max-retries 1`; use decoys (`-D RND:5`) only on real engagements, never on lab (noise for others). Full `-p-` sweeps are fine on labs, wasteful+loud elsewhere.
- Web fuzzers: throttle (`ffuf -t 20 -rate N`), randomized realistic User-Agent (`-u`), cap requests; don't run multiple fuzzers against the same host concurrently.
- nuclei: loud + recognizable — use `-silent -timeout` and a curated `-t` template set on guarded hosts; only ever `-u` a filtered list (never default everything).
- Batch & throttle API/hammering so you look like a user, not a scanner.

## 3. Payload & exploit hygiene
- Avoid signature-flagged defaults: default `msfvenom linux/x64/shell_reverse_tcp` and `shell.php`-style webshell names are the first strings EDR/SOC regexes catch. Rename webshells to look like assets (`media.php`, `cache.php`), use staged+encoded payloads or living-off-the-land where viable.
- Prefer encrypted channels: HTTPS bounce, SSH tunnels, encrypted C2 (Ankou ghost-relay TLS/QUIC). Never exfil in plaintext across monitored segments if you can avoid it.
- sqlmap: don't brute the whole wordlist blasting — use specific `--technique` (E/B/U), `--delay 1`, `--random-agent`, and stop at one authoritative proof/limited dump; full dumps only on approved targets. Read `config` from a temp path, don't leave sqlmap artifacts.
- Persistence/C2 artifacts: only on engagements that authorize them; otherwise command-and-exit.

## 4. Post-exploitation
- On the box, use existing binaries first (`python3`, `perl`, `curl`, `wget`, `bash /dev/tcp`), drop tooling only when needed, and remove your uploads + reset `.bash_history` when done (CTF/lab cleanup). Do not delete or alter other entities' data — cleanup of YOUR artifacts only.
- Disconnect your reverse listeners and kill your shell processes when the objective is met.
- Keep all evidence locally in `~/redteam-runs`; nothing exfil'd to external services.

## 5. Log noise
- HTTPS over HTTP everywhere possible; avoid leaking creds in URLs/query strings.
- Tidy request volume: aim for fewer, better-aimed requests rather than massive canned wordlists where one well-chosen request works.

## Violations
Any agent that defaults to loud/observable behavior without a target-driven reason is wrong. When you catch yourself about to go loud, ask: is there a quieter way to the same objective?
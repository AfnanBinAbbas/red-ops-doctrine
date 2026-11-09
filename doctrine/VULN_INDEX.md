# Red Team Vuln Index (deduplicated)

Single source of truth for which playbook owns which attack class. **One owner per class — no agent reads another agent's playbook.** Ordered by objective value (RCE > SQL dump > PII). Files exist on disk; read the one you need, on demand — never bulk-load.

Full playbook list: `~/.neurosploit-app/agents_md/vulns/`. claude-red methodology: `snailsploit-claude-red/Skills/<cat>/*.md`.

## PIPELINE CONTRACT (noise reduction)
- recon.json → services.json → hosts.json → endpoints.json → findings-raw.md → loot/. Each phase consumes the previous phase's files. **Run every scan exactly once.** If the output file exists, read it; never start a duplicate scan.
- `red-survey` is the ONLY agent that scans the network/ports/hosts/nuclei/dirbusting.
- `red-recon` is the ONLY agent that does OSINT/WHOIS/DNS/exposure-file discovery.
- `red-exploit-web` is the ONLY reader of web vuln playbooks (RCE/SQL/PII via web).
- `red-exploit-infra` is the ONLY reader of infra/service playbooks.
- `red-privesc` owns escalation playbooks (post-foothold).
- Any agent that needs data from another phase reads the checkpoint file — never re-runs the scan.

## Bounty map: web RCE / command execution classes (owner: red-exploit-web)
| Class | neuro-sploit playbook |
|---|---|
| Command injection | `command_injection.md` |
| Local/remote file include | `lfi.md`, `rfi.md` |
| Remote code via file upload | `file_upload.md` |
| SSTI (all engines) | `ssti.md`, `ssti_jinja2.md`, `ssti_freemarker.md`, `ssti_thymeleaf.md`, `ssti_velocity.md` |
| Insecure deserialization | `insecure_deserialization.md`, `pickle_deserialization.md`, `yaml_deserialization.md` |
| Expression/EL injection | `expression_language_injection.md` |
| Server/request smuggling chain→RCE | `http_smuggling.md`, `h2c_smuggling.md`, `http2_request_smuggling.md`, `request_smuggling`(http_desync_*) |
| XML + XXE | `xxe.md`, `xxe_oob_exfiltration.md`, `xxe_billion_laughs.md` |
| Prototype pollution (server-side) | `server_side_prototype_pollution.md`, `prototype_pollution.md` |
| Log4Shell/JNDI | `log4shell_jndi.md` |
| Vulnerable/outdated component | `outdated_dependency_cve.md`, `cve_known_exploitation.md`, `eol_*` |
| Path traversal → read → maybe RCE | `path_traversal.md`, `arbitrary_file_read.md` |
| Known CVE hunting | `cve_hunter.md`, `cve_version_fingerprint.md`, `cve_poc_finder.md` |

## SQL dump classes (owner: red-exploit-web; red-exfil eats the result)
| Class | neuro-sploit playbook |
|---|---|
| SQLi union | `sqli_union.md` |
| SQLi blind/time/error | `sqli_blind.md`, `sqli_time.md`, `sqli_error.md` |
| SQLi auth bypass | `login_sqli_bypass.md` |
| NoSQLi | `nosql_injection.md` |
| ORM injection | `orm_injection.md` |
| SOAP/XML injection | `soap_injection.md`, `xpath_injection.md` |
| GraphQL injection | `graphql_injection.md` |
| SQL creds harvesting (files) | `env_file_exposure.md`, `git_exposed_repo.md`, `backup_file_exposure.md` (owner: red-recon, surface-only → exploit uses) |

## PII / auth & data-exposure classes (owner: red-exploit-web)
| Class | neuro-sploit playbook |
|---|---|
| IDOR/BOLA/BFLA | `idor.md`, `bola.md`, `api_bola_numeric_ids.md`, `api_bola_chained.md`, `bfla.md` |
| Broken access / mass assignment | `access_control_bypass.md`, `mass_assignment.md` |
| Auth bypass | `auth_bypass.md` |
| JWT attacks | `jwt_alg_confusion.md`, `jwt_kid_injection.md`, `jwt_jwk_injection.md`, `jwt_manipulation.md`, `weak_jwt_secret_bruteforce.md` |
| OAuth/OIDC/SAML | `oauth_misconfiguration.md`, `oidc_misconfig.md`, `saml_signature_wrapping.md` |
| Session/2FA | `session_fixation.md`, `two_factor_bypass.md`, `twofa_bypass_techniques.md`, `refresh_token_abuse.md` |
| XSS → cookie/PII steal | `xss_reflected.md`, `xss_stored.md`, `blind_xss.md`, `xss_dom.md`, `dom_xss_spa.md`, `mutation_xss.md`, `postmessage_vulnerability.md` |
| CSRF (state-changing PII ops) | `csrf.md` |
| SSRF (cloud metadata → PII/keys) | `ssrf.md`, `ssrf_cloud.md`, `aws_imds_v2_bypass.md`, `azure_imds_exposure.md`, `gcp_metadata_ssrf.md` |
| Open redirect / password reset | `open_redirect.md`, `password_reset_poisoning.md`, `host_header_injection.md` |
| Info leak sources | `information_disclosure.md`, `sensitive_data_exposure.md`, `misconfig_exposed_files.md`, `api_key_exposure.md` |
| LLM/RAG PII leakage | `llm_pii_leakage.md`, `llm_rag_poisoning.md` (only if LLM surface in scope) |

## Infra / service classes (owner: red-exploit-infra)
| Class | neuro-sploit playbook |
|---|---|
| Default / weak creds | `default_credentials.md`, `misconfig_default_creds.md`, `weak_password.md`, `brute_force.md` |
| Exposed admin / dashboards | `exposed_admin_panel.md`, `misconfig_exposed_dashboards.md`, `k8s_exposed_dashboard.md`, `k8s_exposed_kubelet.md` |
| Cloud IAM privesc | `cloud_iam_privesc.md`, `k8s_rbac_misconfig.md` |
| SSRF→cloud entry | `cloud_metadata_exposure.md` |
| Cache/desync on infra | `cache_poisoning.md`, `http_desync_cl_te.md`, `http_desync_te_cl.md` (web-bordering; only if reached from infra path) |

## Post-foothold escalation classes (owner: red-privesc)
| Class | neuro-sploit playbook |
|---|---|
| Linux/Windows privesc | `privilege_escalation.md` |
| Container escape | `container_escape.md`, `container_escape_advanced.md` |
| Docker socket | `docker_socket_exposure.md` |
| Cloud privesc from box | `cloud_iam_privesc.md` |
| CVE escalation | `cve_exploit_scripter.md` |

## claude-red skill categories (supplementary, per-class)
web, auth, cloud, exploit-dev, fuzzing, infrastructure, active-directory, wireless, mobile — read `<cat>/` only for your owned class. Same single-owner rule.
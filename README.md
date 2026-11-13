# Ahmad Ramadhan

**Cloud & Automation Engineer** — I build and operate automation platforms on Linux, ship them to
the cloud, and document the engineering behind them.

[![Email](https://img.shields.io/badge/email-r3conxploit@protonmail.com-6D28D9?style=flat-square&logo=protonmail&logoColor=white)](mailto:r3conxploit@protonmail.com)
[![Open to](https://img.shields.io/badge/open%20to-Cloud%20%2F%20DevOps%20%2F%20Platform%20roles-0B7285?style=flat-square)](#open-to)
![Based in](https://img.shields.io/badge/based%20in-Indonesia-495057?style=flat-square)

---

## About

I work at the intersection of **systems, automation, and security**: self-hosted workflow
platforms, Linux servers, containerised services, and AI-agent tooling. Most of what I publish is
the part engineers usually skip — architecture decisions, runbooks, failure analysis, and
verification output instead of screenshots.

Currently focused on **Cloud Engineering** (AWS, infrastructure automation),
**Linux systems administration**, and **agentic automation** (n8n, LLM-powered agents).

```text
Cloud Engineering   ·  AWS EC2/EBS/SG, Cloudflare Tunnel, cost-aware architecture
Linux & Ops         ·  Ubuntu, systemd, Docker, bash automation, log/disk budgeting
Automation          ·  n8n workflows, Telegram bots, deterministic parsers, Google Sheets API
Cybersecurity       ·  hardening, least privilege, secret management, authorized lab work
AI Agents           ·  agent orchestration, tool use, prompt/workflow design
```

---

## Featured project

### [n8n-telegram-aws-migration](https://github.com/r3conxploit/n8n-telegram-aws-migration)

Moved a self-hosted **n8n instance + two Telegram bots** off a Windows laptop onto
**AWS EC2 `t3.micro` (ap-southeast-2)** — Docker-pinned runtime, SQLite state restored into a
named volume, ingress via **Cloudflare Tunnel** so the host exposes **zero application ports**
(no ALB, no inbound 443, no TLS certificates on the instance).

| | |
|---|---|
| **Migrated state** | 20 workflows · 7 credentials · 104 executions (24 MB archive) |
| **Verified** | public `/healthz` 200 · `cloudflared` active · survive reboot without intervention |
| **Cost** | ≈ $13/month all-in (EC2 + EBS + free tunnel) |
| **Delivered** | 6 docs, 8 ADRs, 5 idempotent scripts, 10-case troubleshooting catalogue |
| **Hardest part** | disk exhaustion (8 GB → 30 GB), 502-vs-200 ingress isolation, SQLite consistency + encryption key |

📄 [`README`](https://github.com/r3conxploit/n8n-telegram-aws-migration) ·
🏛️ [`docs/decisions.md`](https://github.com/r3conxploit/n8n-telegram-aws-migration/blob/main/docs/decisions.md) ·
🧯 [`docs/troubleshooting.md`](https://github.com/r3conxploit/n8n-telegram-aws-migration/blob/main/docs/troubleshooting.md)

---

## Use cases & problem solving

Six operational cases from running this stack for real — each one documented as
**problem → diagnosis → fix → verification output**, not as a tutorial.

| # | Case | Headline result |
|:--:|---|---|
| 1 | [Recovering a full system drive with zero reinstalls](docs/use-cases.md#1-recovering-a-full-system-drive-with-zero-reinstalls) | 9.9 GB → **33 GB free** via junctions, no reinstall, no path changes |
| 2 | [Self-hosted multi-provider AI gateway with cost control](docs/use-cases.md#2-self-hosted-multi-provider-ai-gateway-with-cost-control) | Loopback-only routing across free/paid tiers, machine-scoped admin token, no stored passwords |
| 3 | [GitHub CLI auth on a Windows/MSYS host](docs/use-cases.md#3-github-cli-auth-on-a-windowsmsys-host) | Non-PTY device flow that actually completes; scope drift caught before a failed push |
| 4 | [Provisioning an agent skill library under rate limits](docs/use-cases.md#4-provisioning-an-agent-skill-library-under-rate-limits) | 100+ skills installed with **zero** authenticated API calls (60 req/h cap bypassed) |
| 5 | [n8n + Telegram bots: laptop → AWS EC2](docs/use-cases.md#5-n8n--telegram-bots-laptop--aws-ec2) | Zero open app ports, state restored, survives reboot, ≈ $13/month |
| 6 | [Delegation that reported success and shipped nothing](docs/use-cases.md#6-delegation-that-reported-success-and-shipped-nothing) | A "completed" sub-agent with 0 artifacts → verification gates on every delegation |

```text
Storage & Host Ops   ·  junction-based app-data migration, read-only attribute pitfalls, disk budgeting
AI / Gateway Infra   ·  multi-provider routing chains, tiered model combos, measured token savings
Cloud Delivery       ·  EC2 sizing, Cloudflare Tunnel ingress, idempotent scripts, ADR trail
Agent Operations     ·  skill library provisioning, credential handling, delegation verification
```

📄 Full write-up → [`docs/use-cases.md`](docs/use-cases.md)

---

## Other repositories

| Repository | What it is |
|---|---|
| [`osint-tools`](https://github.com/r3conxploit/osint-tools) | Python utilities for open-source intelligence: username enumeration and data gathering, for authorized research. |
| [`redteam-lab`](https://github.com/r3conxploit/redteam-lab) | Adversary-simulation notes and lab exercises — scoped to my own environment, written for detection and hardening. |
| [`stego-lab`](https://github.com/r3conxploit/stego-lab) | Steganography research: concealment techniques, tooling, and how to detect them. |
| [`darkwiki`](https://github.com/r3conxploit/darkwiki) | Markdown knowledge base — security notes, field guides, and operational references. |
| [`r3x-terminal`](https://github.com/r3conxploit/r3x-terminal) | Terminal configuration and CLI tooling: shell ergonomics, prompts, dotfiles. |

> Security work here is **defensive and lab-scoped**: hardening, detection, and documentation.
> Nothing is intended for use against systems without written authorization.

---

## Toolkit

![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat-square&logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat-square&logo=linux&logoColor=black)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=flat-square&logo=ubuntu&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=flat-square&logo=gnubash&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![n8n](https://img.shields.io/badge/n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white)
![Cloudflare](https://img.shields.io/badge/Cloudflare%20Tunnel-F38020?style=flat-square&logo=cloudflare&logoColor=white)
![Git](https://img.shields.io/badge/Git-F05032?style=flat-square&logo=git&logoColor=white)
![systemd](https://img.shields.io/badge/systemd-000000?style=flat-square&logo=systemd&logoColor=white)

---

## Open to

Cloud Engineering · DevOps / Platform · Linux Systems Administration · Automation Engineering —
roles where reliability, cost awareness, and clean documentation actually matter.

📬 **r3conxploit@protonmail.com**

---

<!-- PORTFOLIO:START -->

## 🗂️ Portfolio

**7 repo** · **1 ⭐** · top stack: `Python`, `Shell`

| # | Repo | Fokus | Stack | Aktivitas |
|:--:|---|---|---|---|
| 1 | [`r3conxploit`](https://github.com/r3conxploit/r3conxploit) — Profile repository — Cloud & Automation Engineer: Linux systems, AWS infrastruct… | Cloud & DevOps | `—` | 🟢 aktif |
| 2 | [`n8n-telegram-aws-migration`](https://github.com/r3conxploit/n8n-telegram-aws-migration) — Cloud engineering case study: migrate self-hosted n8n + Telegram bots from a lap… | Cloud & DevOps | `Shell` | 🟢 aktif |
| 3 | [`osint-tools`](https://github.com/r3conxploit/osint-tools) — Python utilities for open-source intelligence: username enumeration and authoriz… | OSINT & Recon | `Python` | 🟢 aktif |
| 4 | [`darkwiki`](https://github.com/r3conxploit/darkwiki) — Markdown knowledge base — security notes, field guides, and operational referenc… | OSINT & Recon | `—` | 🟢 aktif |
| 5 | [`redteam-lab`](https://github.com/r3conxploit/redteam-lab) — Adversary-simulation lab notes and exercises — detection, hardening, and defensi… | Red Team | `—` | 🟢 aktif |
| 6 | [`stego-lab`](https://github.com/r3conxploit/stego-lab) — Steganography research: concealment techniques, tooling, and detection methods. | Steganography | `—` | 🟢 aktif |
| 7 | [`r3x-terminal`](https://github.com/r3conxploit/r3x-terminal) — Terminal configuration and CLI tooling — shell ergonomics, prompts, and dotfiles… | Terminal & Tooling | `—` | 🟢 aktif |

<sub>Generated 2026-11-11 23:44 by asisten-github</sub>

<!-- PORTFOLIO:END -->

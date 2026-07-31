# sleipnir

**Sleipnir** — Odin's eight-legged steed, the swift mount across the nine realms — is a fast, opinionated
`doctl` wrapper for navigating your **DigitalOcean** estate from the terminal. It turns long, flag-heavy
doctl invocations into short verbs with sane defaults. The DigitalOcean-side companion to the GitHub
"pack" (huginn · muninn · geri · freki · grimnir).

> **Status:** scaffold. The command harness (dispatch, preflight, config, help) is in place; the
> read-first commands are being built. Build brief for the agent: [`CLAUDE.md`](CLAUDE.md).

## What it does (v1 — read + navigate)

| Command | What it does |
| ------- | ------------ |
| `sleipnir ls [--tag X]` | Droplet table — name · IP · region · size · tags · est. $/mo |
| `sleipnir ip <name>` | Just the public IPv4 of a droplet (for ssh / ansible) |
| `sleipnir ssh <name>` | SSH into a droplet by name |
| `sleipnir survey` | Whole-estate view — droplets, apps, Spaces, DBs, firewalls + a cost tally |
| `sleipnir apps` / `deploys <app>` / `logs <app>` | App Platform |
| `sleipnir config` / `install` | Print or write the local config |

## How it works

Bash + `doctl` + `jq`. Sleipnir **shells out to doctl** — which owns authentication — and formats the
result. It never talks to the DO API directly and **never handles your DO token.** Defaults (region, size,
image, SSH keys, tags, droplet profiles) live in local config; see
[`sleipnir.config.example`](sleipnir.config.example).

**Scope boundary:** Sleipnir is for *interactive ops and inspection* — quick fetches, throwaway droplets,
checking what's running and what it costs, tailing a deploy. Standing, reproducible infrastructure stays in
Terraform + Ansible (heimdall, vor, asgard), not here.

## Install

_TBD — will follow the pack's install path (and Homebrew tap) once v1 lands._

## Conventions

Branch → PR → review → merge; no direct commits to `main`. Signed commits. AgentGate on every PR. See
[`CONTRIBUTING.md`](CONTRIBUTING.md) and, for building in the repo, [`CLAUDE.md`](CLAUDE.md).

## License

[MIT](LICENSE) © Brett Buskirk.

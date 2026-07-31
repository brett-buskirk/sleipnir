# sleipnir

**Sleipnir** — Odin's eight-legged steed, the swift mount across the nine realms — is a fast, opinionated
`doctl` wrapper for navigating your **DigitalOcean** estate from the terminal. It turns long, flag-heavy
doctl invocations into short verbs with sane defaults. The DigitalOcean-side companion to the GitHub
"pack" (huginn · muninn · geri · freki · grimnir).

> **Status:** v1 in progress — the droplet verbs (`ls`, `ip`) and `config` are live; the rest of the
> read-first surface is being built. See the [ROADMAP](ROADMAP.md), or
> [`CHEATSHEET.md`](CHEATSHEET.md) for full reference on what ships today.

## What it does (v1 — read + navigate)

| Command | What it does | |
| ------- | ------------ |-|
| `sleipnir ls [--tag X] [--region Y]` | Droplet table — name · IP · region · size · tags · est. $/mo | ✅ |
| `sleipnir ip <name>` | Just the public IPv4 of a droplet (for ssh / ansible) | ✅ |
| `sleipnir ssh <name> [-- <args>]` | SSH into a droplet by name | ✅ |
| `sleipnir config` / `install` | Show the resolved config, or write a starter one | ✅ |
| `sleipnir apps` / `deploys <app>` / `logs <app>` | App Platform — apps, deploy history, logs | ✅ |
| `sleipnir survey` | Whole-estate view — droplets, apps, Spaces, DBs, firewalls + a cost tally | |

```console
$ sleipnir ls

  droplets  · Brett Buskirk LLC · context brett

  rootroute-staging  64.225.22.218    nyc3  s-2vcpu-2gb  $18/mo  active  staging,web,rootroute
  vor-analytics      165.227.123.156  nyc3  s-2vcpu-4gb  $24/mo  active  plausible,analytics,vor

  2 droplets  · ~$42/mo

$ sleipnir ssh vor-analytics
  → root@165.227.123.156 (vor-analytics · Brett Buskirk LLC · context brett)
```

Run `sleipnir install` once to write a starter config, then `sleipnir config` to see what's in force.

## How it works

Bash + `doctl` + `jq`. Sleipnir **shells out to doctl** — which owns authentication — and formats the
result. It never talks to the DO API directly and **never handles your DO token.** Defaults (region, size,
image, SSH keys, tags, droplet profiles) live in local config; see
[`sleipnir.config.example`](sleipnir.config.example).

**The active context is always visible.** doctl has named contexts the way `gh` has accounts, and hitting
the wrong one is the same class of mistake as pushing as the wrong GitHub user. Estate-wide views carry a
header naming the account and context the numbers came from, so "which account am I about to hit?" is
never a guess.

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

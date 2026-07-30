# CLAUDE.md — Sleipnir

## What Sleipnir is
**Sleipnir** — Odin's eight-legged steed, the swiftest mount across the nine realms — is a fast,
opinionated **`doctl` wrapper** for navigating Brett's **DigitalOcean** estate from the terminal. It turns
the long, flag-heavy `doctl` invocations (fetch a droplet's IP, list by tag, spin up a throwaway box,
glance at cost) into short, memorable verbs with sane defaults. It is the **DigitalOcean-side companion to
the GitHub "pack"** — huginn/muninn/geri/freki/grimnir manage the GitHub estate; Sleipnir manages the DO
estate.

You are the agent building and maintaining Sleipnir. This file is your standing brief — read it before any
non-trivial change.

## You operate under the estate's floors
Sleipnir lives in Brett's estate (`~/github-repos`), so two higher-level files auto-load and **govern
everything here** — don't restate or override them:
- `/etc/claude-code/CLAUDE.md` — the machine-wide **managed policy** (safety, review, governance floors).
- `~/github-repos/CLAUDE.md` — the **Estate Steward manual**; its *Issue & PR conventions* apply to you
  directly (assignee `brett-buskirk`, labels, the Estate board **#17**).

The non-negotiables in short: **branch → PR → stop and let Brett merge** (never self-merge, never commit to
`main`); signed commits ending `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`; never commit
secrets; `brett-buskirk` is the active `gh` account.

## Design north stars (if a change conflicts with one of these, the change is wrong)
1. **Thin wrapper, never a reimplementation.** Sleipnir shells out to `doctl` (and `ssh`) and formats the
   result. It does **not** talk to the DO API directly and does **not** re-do what doctl already handles
   (auth, pagination, retries). When doctl changes, Sleipnir keeps working. Parse `doctl … --output json`
   with `jq` — never scrape doctl's human tables.
2. **Never handle the DO token.** `doctl` owns authentication, completely. Sleipnir never reads, stores,
   prompts for, echoes, or logs a DO API token. No secrets in the repo, and none in the committed config
   example.
3. **Interactive ops, not infrastructure-as-code.** Sleipnir is for *ad-hoc* work — fetching, inspecting,
   throwaway droplets, tailing a deploy, a cost glance. **Standing / reproducible infrastructure stays
   declarative in Terraform + Ansible** (heimdall, vor, asgard). If a resource should persist and be
   reproducible, it is not Sleipnir's job. Don't grow Sleipnir into a Terraform competitor — that's the
   same call Brett already made ("Ansible role, not a Terraform `remote-exec` provisioner").
4. **Guard the active context.** `doctl` has named contexts the way `gh` has accounts; acting on the wrong
   one is the DO analog of the `itguysocal`-vs-`brett-buskirk` gotcha. Show the active context/account on
   any state-changing action, and make "which account am I about to hit?" trivial to see.
5. **Read-first, safety-gated.** v1 is **read + navigate only** — no destructive verbs. When create/resize/
   delete arrive (v2+), they follow the **freki** model: dry-run by default or an explicit confirm, never a
   bare destroy, with the active context shown loudly.
6. **Config-driven, pack-consistent.** Defaults (region, image, size, SSH-key fingerprints, tags, droplet
   profiles) live in **config, not code**. Follow pack conventions: XDG config dir, `$SLEIPNIR_*` env
   overrides, an `install`/`config` verb, `--help` on every command, a `CHEATSHEET.md` mirroring vegtam's,
   and born-compliant repo hygiene.

## Stack & shape
- **Bash + `doctl` + `jq`** — the pack's idiom (huginn/geri/freki). Preflight-check that `doctl` and `jq`
  exist and doctl is authenticated; fail with a clear, actionable message otherwise (a skeleton
  `require_doctl` is already in `bin/sleipnir`).
- **Entry point:** `bin/sleipnir` — a single dispatch script to start. If it grows, split subcommands into
  a `lib/` the way the pack does. The command harness (dispatch, preflight, config load, `help`/`version`/
  `config`) is scaffolded; **fill in the subcommands.**
- **Config:** `${SLEIPNIR_CONFIG:-${XDG_CONFIG_HOME:-$HOME/.config}/sleipnir/config}`, sourced if present.
  `sleipnir.config.example` documents the shape.

## v1 surface — build these first (read + navigate)
| Command | What it does |
| ------- | ------------ |
| `sleipnir ls [--tag X] [--region Y]` | Droplet table: name · public IP · region · size · tags · status · est. $/mo |
| `sleipnir ip <name>` | Print just the public IPv4 of a droplet (non-zero exit if not found) — the everyday "stop typing `--format`" win |
| `sleipnir ssh <name> [-- <args>]` | Resolve name→IP and `ssh <user>@<ip>` with the configured default user/key; pass extra args through |
| `sleipnir survey` | The flagship view — droplets, App Platform apps, Spaces, databases, firewalls, reserved IPs, plus a rough monthly-cost tally |
| `sleipnir apps` / `deploys <app>` / `logs <app>` | App Platform (day-one, brett-buskirk-dev, vor-when-live): list apps, recent deployments, tail logs |
| `sleipnir help [cmd]` / `version` / `config` / `install` | Meta: help, version, show resolved config, write a starter config |

**v1 acceptance:** each command works against a real doctl-authed account; handles *not found* and *doctl
not authenticated / wrong context* cleanly (no stack traces); `--help` documents it; `shellcheck` is clean.

### The `ls` idiom — follow this pattern
```bash
doctl compute droplet list --output json \
  | jq -r '.[] | [.name,
                  (.networks.v4[] | select(.type=="public").ip_address),
                  .region.slug, .size_slug, (.tags | join(","))] | @tsv' \
  | column -t
```
Estimate $/mo from the droplet's `.size.price_monthly` (or map `.size_slug` via `doctl compute size list
--output json`).

## Brett's DO footprint (tune defaults/profiles to this — but keep specifics in *config*, not code)
- **Droplets:** heimdall (observability), vor (analytics, when live), and ad-hoc `ansible-baseline` test
  boxes. Region typically **nyc3**, image **debian-13-x64** (trixie), smallest test size **s-1vcpu-1gb**.
- **App Platform:** **day-one** (dayone-sim.app), **brett-buskirk-dev** (the site), possibly vor.
- A natural first **droplet profile** ("test"/"hardened") — debian-13-x64, s-1vcpu-1gb, nyc3, Brett's SSH
  key, a tag — mirrors the `ansible-baseline` harness create step, so v2's `sleipnir summon` becomes that
  step in one word.
- **Never hardcode SSH-key fingerprints or account specifics** — they live in local config; this repo may
  go public.

## Roadmap (phases)
- **v1 — read + navigate** (this scope): `ls` / `ip` / `ssh` / `survey` / `apps` / `deploys` / `logs` +
  `config` / `install`, CHEATSHEET, shellcheck CI.
- **v2 — safe composites:** `summon <name> --profile <p>` (create from a profile, `--wait`, print IP),
  `open <app>`, etc. Still no bare deletes.
- **v3 — destructive, gated:** `reap` / resize / rebuild — freki-style dry-run + confirm, loud active-context
  display.

## Testing / verification
- **`shellcheck bin/sleipnir`** (and any `lib/`) must pass — wire it into CI (a shellcheck workflow if the
  pack doesn't already provide a template).
- **Smoke:** `bin/sleipnir --help`, `version`, `config` run fine with *no* doctl account — the
  doctl-dependent paths sit behind the preflight and fail cleanly.
- **Manual:** run each read command against the real account; confirm output + the not-found + not-authed
  cases.
- Keep it green locally before opening a PR; branch → PR → **stop for Brett to merge**.

## Visibility
Created **private** (like every `huginn new` repo). It's a natural fit to go **public** alongside the rest
of the pack once v1 is real — Brett's call, and a `geri`-hardening pass (secret scanning + push protection)
goes with the flip. Config with real fingerprints/defaults always stays **local**, never committed.

## Where to look
- **The pack** — huginn / geri / freki / grimnir / vegtam — for structure, config conventions, the
  `CHEATSHEET.md` format, CI, and the **freki safety model** for the eventual destructive verbs.
- `~/sandboxes/ansible-baseline/` (harness README) — the exact `doctl compute droplet create … --wait`
  invocation that v2's `summon` profile should encapsulate.
- `doctl` docs for the `--output json` shapes you'll `jq` over.

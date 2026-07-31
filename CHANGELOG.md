# Changelog

All notable changes to sleipnir are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0] — 2026-07-30

First release. The **read + navigate** surface is complete: every command works against a real
doctl-authenticated account, handles *not found* and *not authenticated* cleanly, documents itself with
`--help`, and passes `shellcheck`.

### Added — droplets
- `ls [--tag X] [--region Y] [--json]` — droplet table (name · public IPv4 · region · size · estimated
  $/mo · status · tags) with a monthly total.
- `ip <name>` — print just a droplet's public IPv4 on stdout, so it embeds in other commands. Matching
  is exact, then case-insensitive; an ambiguous name is an error rather than a guess.
- `ssh <name> [-u user] [-i key] [-- <ssh args>]` — resolve a name to its IP and connect, passing
  everything after `--` to `ssh` untouched. Prints the target and the owning account to stderr before
  handing over the terminal.

### Added — App Platform
- `apps [--json]` — app table (name · phase · time since last deploy · components · region · tier ·
  live URL).
- `deploys <app> [-n N] [--json]` — recent deployments, newest first. Takes an app *name*; doctl's own
  `list-deployments` requires a UUID.
- `logs <app> [component] [-f] [-n N] [--type build|deploy|run]` — read an app's logs. Defaults to
  **build** logs for an app with no runtime component, because doctl's `run` default hangs on a
  websocket that never delivers for a static site.

### Added — the estate
- `survey [--json]` — whole-estate view: droplets, apps, volumes, and firewalls in detail, plus counts
  of reserved IPs, databases, load balancers, and Kubernetes clusters, with an estimated monthly cost.
  Distinguishes **none** (doctl answered; nothing there) from **unknown** (the lookup failed), and says
  plainly when a failed *priced* lookup has left the total an undercount.

### Added — meta & configuration
- `config` — show the resolved configuration and the active doctl context + account.
- `install [--force]` — write a starter config `0600`, refusing to overwrite an existing one without
  `--force`, prefilling SSH-key fingerprints from the account when doctl is authenticated.
- Two-level help (`sleipnir help`, `sleipnir <command> help`), `NO_COLOR` and non-TTY plain output, and
  an active-context header on estate-wide views — because acting on the wrong DigitalOcean context is
  the analog of pushing as the wrong GitHub account.
- `SLEIPNIR_SSH_IDENTITY` — local private key for `ssh -i`, distinct from the `SLEIPNIR_SSH_KEYS`
  fingerprints doctl attaches at create time.
- `SLEIPNIR_VOLUME_PRICE_GIB` (default `0.10`) — block-storage price per GiB/month. doctl reports a
  droplet's price but not a volume's, so this is the one figure sleipnir must be told; it lives in
  config so it can be corrected when pricing moves.
- `CHEATSHEET.md` and a shellcheck CI workflow.

### Notes
- Sleipnir never reads, stores, prompts for, or logs a DigitalOcean API token — `doctl` owns
  authentication entirely.
- Cost figures cover droplet list price and block storage only. Apps, databases, load balancers,
  snapshots, backups, and bandwidth are excluded. A glance, not an invoice.
- Spaces buckets are not listed: doctl exposes only `spaces keys`, and sleipnir does not call the
  DigitalOcean API directly to fill gaps in doctl's coverage.

[Unreleased]: https://github.com/brett-buskirk/sleipnir/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/brett-buskirk/sleipnir/releases/tag/v1.0.0

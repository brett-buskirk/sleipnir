# Changelog

All notable changes to sleipnir are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

_The v1 surface (read + navigate) is now complete._

### Added
- `survey` — the whole-estate view: droplets, apps, volumes, and firewalls in detail, plus counts of
  reserved IPs, databases, load balancers, and Kubernetes clusters, with an estimated monthly cost.
  Supports `--json`.
- `SLEIPNIR_VOLUME_PRICE_GIB` (default `0.10`) — block-storage price per GiB/month. doctl reports a
  droplet's price but not a volume's, so this is the one figure sleipnir must be told; it lives in
  config so it can be corrected when pricing moves.
- `survey` distinguishes **none** (doctl answered; nothing there) from **unknown** (the lookup failed),
  and states plainly when a failed *priced* lookup has left the total an undercount.
- `apps` — App Platform table (name · phase · time since last deploy · components · region · tier ·
  live URL), with `--json`.
- `deploys <app>` — recent deployments, newest first, with `-n` and `--json`. Takes an app *name*;
  doctl's own `list-deployments` requires a UUID.
- `logs <app> [component]` — read an app's logs with `-f` / `-n` / `--type`. Defaults to **build** logs
  for apps with no runtime component, because doctl's `run` default hangs on a websocket that never
  delivers for a static site.
- `ssh <name>` — resolve a droplet name to its IP and connect as the configured user, with `-u`/`-i`
  overrides and everything after `--` passed to `ssh` untouched. Prints the target (and the account it
  belongs to) on stderr before handing over the terminal.
- `install` — write a starter config `0600`, refusing to overwrite an existing one without `--force`,
  prefilling SSH-key fingerprints from the account when doctl is authenticated.
- `SLEIPNIR_SSH_IDENTITY` — local private key for `ssh -i`, distinct from the `SLEIPNIR_SSH_KEYS`
  fingerprints that doctl attaches at create time.
- `ls` — droplet table (name · public IPv4 · region · size · estimated $/mo · status · tags) with a
  monthly total, `--tag` / `--region` filters, and `--json`.
- `ip <name>` — print just a droplet's public IPv4 on stdout, for embedding in other commands.
- `config` — show the resolved configuration and the active doctl context + account.
- Two-level help (`sleipnir help`, `sleipnir <command> help`), `NO_COLOR` / non-TTY plain output, and
  an active-context header on estate-wide views.
- `CHEATSHEET.md` and a shellcheck CI workflow.
- Initial scaffold.

### Changed
- Command harness reworked to the pack idiom: `set -uo pipefail` (commands return non-zero rather than
  aborting mid-render), memoized doctl lookups so a run costs one droplet call, and preflight failures
  that report the active context instead of a bare error.

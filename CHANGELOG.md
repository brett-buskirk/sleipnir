# Changelog

All notable changes to sleipnir are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
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

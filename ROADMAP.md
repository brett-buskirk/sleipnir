# Roadmap

_What's planned for sleipnir — check items off as they ship._

## v1 — read + navigate (current)
- [ ] `ls` — droplet table (name · IP · region · size · tags · $/mo)
- [ ] `ip <name>` — print a droplet's public IPv4
- [ ] `ssh <name>` — ssh into a droplet by name
- [ ] `survey` — whole-estate view + rough monthly cost tally
- [ ] `apps` / `deploys <app>` / `logs <app>` — App Platform
- [ ] `config` / `install` — local config
- [ ] `CHEATSHEET.md` + shellcheck CI

## v2 — safe composites
- [ ] `summon <name> --profile <p>` — create from a profile, `--wait`, print IP (the `ansible-baseline` step in one word)
- [ ] `open <app>` and other read/create conveniences (no bare deletes)

## v3 — destructive, gated
- [ ] `reap` / resize / rebuild — freki-style dry-run + confirm, with a loud active-context display

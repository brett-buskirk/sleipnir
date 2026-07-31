# Roadmap

_What's planned for sleipnir — check items off as they ship._

## v1 — read + navigate (current)
- [x] `ls` — droplet table (name · IP · region · size · tags · $/mo)
- [x] `ip <name>` — print a droplet's public IPv4
- [x] `ssh <name>` — ssh into a droplet by name
- [x] `survey` — whole-estate view + rough monthly cost tally
- [x] `apps` / `deploys <app>` / `logs <app>` — App Platform
- [x] `config` — show the resolved config + active doctl context
- [x] `install` — write a starter config
- [x] shellcheck CI
- [x] `CHEATSHEET.md` — kept current with each command as it lands

## v2 — safe composites
- [ ] `summon <name> --profile <p>` — create from a profile, `--wait`, print IP (the `ansible-baseline` step in one word)
- [ ] `open <app>` and other read/create conveniences (no bare deletes)

## v3 — destructive, gated
- [ ] `reap` / resize / rebuild — freki-style dry-run + confirm, with a loud active-context display

# sleipnir cheat sheet

Quick reference for every command, option, and behavior. `sleipnir` is a thin wrapper over
[`doctl`](https://docs.digitalocean.com/reference/doctl/) — it shells out, parses the JSON, and formats
the result. **doctl owns authentication; sleipnir never reads, stores, or logs your DO token.**

For the narrative version see the [README](README.md); for per-command detail in the terminal, run
`sleipnir <command> help`.

---

## At a glance

| Command | What it does | Options |
|---------|--------------|---------|
| [`ls`](#ls) | Droplet table — name · IP · region · size · $/mo · status · tags | `--tag`, `--region`, `--json` |
| [`ip`](#ip-name) | Print just a droplet's public IPv4 | |
| [`config`](#config) | Resolved config + the active doctl context | |
| [`help`](#help--version) | The menu, or detail for one command | |
| `version` | Print the version | |

Not built yet — see the [ROADMAP](ROADMAP.md): `ssh` · `survey` · `apps` · `deploys` · `logs` · `install`.

---

## Requirements & global behavior

- **Requires** `bash` + [`doctl`](https://docs.digitalocean.com/reference/doctl/) + `jq`. The
  doctl-backed commands check all three up front and fail with an actionable message; the meta commands
  (`help`, `version`, `config`) work on a machine with no doctl account at all.
- **Authentication is doctl's job.** Run `doctl auth init` once. If the active context is wrong,
  `doctl auth switch --context <name>`.
- **The active context is always visible.** Estate-wide views carry a header naming the account and
  doctl context the numbers came from — acting on the wrong DigitalOcean account is the analog of
  pushing as the wrong GitHub user, so sleipnir makes "which account am I about to hit?" impossible to
  miss.
- **`NO_COLOR`** — set it (`NO_COLOR=1 sleipnir ls`) to disable color. Output is also automatically
  plain when piped or redirected (not a TTY).
- **Two-level help** — `sleipnir help` for the menu, `sleipnir <command> help` (or `-h`/`--help`) for
  one command.
- **Exit codes** — `0` on success; `1` on error (missing dependency, doctl not authenticated, a name
  that doesn't resolve, an unknown command).

### Configuration

Read from `$SLEIPNIR_CONFIG`, defaulting to `${XDG_CONFIG_HOME:-~/.config}/sleipnir/config`. It is a
plain shell file, sourced at startup — see [`sleipnir.config.example`](sleipnir.config.example).
Precedence is **`SLEIPNIR_*` environment variables > the config file > built-in defaults.**

| Key | Default | Used by |
|-----|---------|---------|
| `SLEIPNIR_REGION` | `nyc3` | default region for creates (v2) |
| `SLEIPNIR_SSH_USER` | `root` | `ssh` (not built yet) |

**Keep it secret-free.** doctl holds the API token; this file holds only defaults and profiles.

---

## `ls`

Every droplet on the active account, one row each: name · public IPv4 · region · size · estimated
$/mo · status · tags, with a monthly total. *(network — one API call)*

```sh
sleipnir ls
sleipnir ls --tag vor            # only droplets carrying a tag
sleipnir ls --region nyc3        # only droplets in a region
sleipnir ls --tag web --region nyc3
sleipnir ls --json               # machine-readable rows
```

| Option | Effect |
|--------|--------|
| `--tag <tag>` | Only droplets carrying this tag |
| `--region <slug>` | Only droplets in this region (`nyc3`, `sfo3`, …) |
| `--json` | Emit JSON rows instead of the table |

**`--json` fields:** `name, ip, region, size, price, tags[], status`

```console
$ sleipnir ls

  droplets  · Brett Buskirk LLC · context brett

  rootroute-staging  64.225.22.218    nyc3  s-2vcpu-2gb  $18/mo  active  staging,web,rootroute
  vor-analytics      165.227.123.156  nyc3  s-2vcpu-4gb  $24/mo  active  plausible,analytics,vor

  2 droplets  · ~$42/mo
```

> **On the cost figure:** it is doctl's list price for the droplet's *size*, summed. It ignores
> volumes, snapshots, backups, and bandwidth overage — a glance, not an invoice. A droplet with no
> public interface shows `—` in the IP column rather than dropping out of the table.

---

## `ip <name>`

Print **just** the public IPv4 of a droplet — nothing else on stdout, so it drops straight into other
commands. Exits non-zero if the name doesn't resolve.

```sh
sleipnir ip vor-analytics
ssh root@$(sleipnir ip vor-analytics)
ansible-playbook -i "$(sleipnir ip web)," site.yml
```

Matching is **exact first, then case-insensitive**. Anything ambiguous is an error, never a guess:

| Situation | Behavior |
|-----------|----------|
| One match | Prints the IPv4, exit `0` |
| No match | Error on **stderr** + a "did you mean" list of substring matches, exit `1` |
| More than one match | Error naming the count, exit `1` (DigitalOcean permits duplicate droplet names) |
| Match with no public IPv4 | Error — private-only droplet, exit `1` |

Errors go to stderr and diagnostics never touch stdout, so `IP=$(sleipnir ip web)` is safe to embed.

---

## `config`

Show the resolved configuration: which file was loaded (if any), the defaults in force, and the doctl
context + account sleipnir would act on. Works without doctl installed.

```sh
sleipnir config
```

```console
$ sleipnir config

  sleipnir config

    file        /home/you/.config/sleipnir/config (loaded)
    region      nyc3
    ssh user    root
    doctl       brett
    account     Brett Buskirk LLC
```

---

## `help` / `version`

```sh
sleipnir help          # the command menu
sleipnir help ls       # detail for one command
sleipnir ls help       # same thing
sleipnir ls --help     # and so is this
sleipnir version
```

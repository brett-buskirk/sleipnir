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
| [`ssh`](#ssh-name) | SSH into a droplet by name | `-u`, `-i`, `-- <ssh args>` |
| [`apps`](#apps) | App Platform apps — phase, age, components, live URL | `--json` |
| [`deploys`](#deploys-app) | Recent deployments for an app | `-n`, `--json` |
| [`logs`](#logs-app-component) | Read an app's logs | `-f`, `-n`, `--type` |
| [`config`](#config) | Resolved config + the active doctl context | |
| [`install`](#install) | Write a starter config | `--force` |
| [`help`](#help--version) | The menu, or detail for one command | |
| `version` | Print the version | |

Not built yet — see the [ROADMAP](ROADMAP.md): `survey`.

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
| `SLEIPNIR_SSH_USER` | `root` | `ssh` |
| `SLEIPNIR_SSH_IDENTITY` | *(unset)* | `ssh` — local private key, passed to `ssh -i` |
| `SLEIPNIR_SSH_KEYS` | *(unset)* | public-key **fingerprints** for doctl to attach on create (v2) |
| `SLEIPNIR_TAGS` | `sleipnir` | default tags on created resources (v2) |

> `SLEIPNIR_SSH_IDENTITY` and `SLEIPNIR_SSH_KEYS` are **different things**: the first is a private key
> file on this machine that `ssh` uses to log in; the second is a list of public-key fingerprints
> identifying keys stored at DigitalOcean, for attaching to droplets at create time.

**Keep it secret-free.** doctl holds the API token; this file holds only defaults and profiles.
`sleipnir install` writes it `0600`.

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

## `ssh <name>`

Resolve a droplet name to its public IPv4 and connect. Sleipnir prints the target on **stderr** before
handing the terminal to `ssh`, so you always see which host — and which account — you landed on.

```sh
sleipnir ssh vor-analytics
sleipnir ssh vor-analytics -u deploy                    # different user
sleipnir ssh vor-analytics -i ~/.ssh/do_ed25519         # explicit key
sleipnir ssh vor-analytics -- -L 8080:localhost:80      # port-forward
sleipnir ssh vor-analytics -- systemctl status caddy    # run and return
```

| Option | Effect |
|--------|--------|
| `-u`, `--user <user>` | Connect as this user (default: `SLEIPNIR_SSH_USER`, i.e. `root`) |
| `-i`, `--identity <path>` | Private key to use, passed to `ssh -i` (default: `SLEIPNIR_SSH_IDENTITY`) |
| `-- <ssh args>` | Everything after `--` is handed to `ssh` untouched |

Name resolution is identical to [`ip`](#ip-name) — exact, then case-insensitive, and an ambiguous name
is an error rather than a guess.

> **Why pass-through args land after the host:** OpenSSH re-enters its option loop once it has consumed
> the hostname, so `ssh host -L 8080:localhost:80` really does set up the forward — while a *remote
> command* only works in that position. Putting everything after the host is what lets one `--` slot
> serve both.

---

## `apps`

Every App Platform app on the active account: name · deployment phase · time since the last deploy ·
components · region · tier · live URL. *(network — one API call)*

```sh
sleipnir apps
sleipnir apps --json
```

```console
$ sleipnir apps

  App Platform  · Brett Buskirk LLC · context brett

  brett-buskirk-dev  ACTIVE    14m  static:1  nyc  starter  https://brett-buskirk.dev
  day-one            ACTIVE     1d  static:1  nyc  starter  https://dayone-sim.app
  rc-journey         ACTIVE     1d  static:1  nyc  starter  https://rcjourney.cloud

  3 apps  · age is time since the last deployment · sleipnir deploys <app> for history
```

**`--json` fields:** `name, id, region, tier, url, phase, deployed` (epoch), `runtime` (count of
executing components), `components` (compact summary)

Components are counted by kind — `svc` · `static` · `worker` · `job` · `fn`. An app with no `svc`,
`worker`, or `job` has **no runtime**, which is what [`logs`](#logs-app-component) keys off.

---

## `deploys <app>`

Recent deployments, newest first — short id · phase · age · cause (usually the triggering commit).

```sh
sleipnir deploys day-one
sleipnir deploys day-one -n 25
sleipnir deploys day-one --json
```

| Option | Effect |
|--------|--------|
| `-n`, `--limit <count>` | How many to show (default: `10`) |
| `--json` | Machine-readable rows |

```console
$ sleipnir deploys day-one -n 3

  deployments · day-one · Brett Buskirk LLC · context brett

  d86e5c57  ACTIVE         1d  commit 70595d7 pushed to github.com/brett-buskirk/day-one/tree/main
  6e30b8ce  SUPERSEDED     1w  commit 06e9b75 pushed to github.com/brett-buskirk/day-one/tree/main
  58f148ee  SUPERSEDED     1w  commit ccfa00b pushed to github.com/brett-buskirk/day-one/tree/main
```

> Takes an app **name**. `doctl apps list-deployments` demands a UUID and rejects a name outright
> (`invalid uuid`) — sleipnir resolves it for you, using the same matching rules as `ip`.

---

## `logs <app> [component]`

Read an app's logs. Hands off to doctl, so `-f` streams until you interrupt it.

```sh
sleipnir logs day-one
sleipnir logs day-one -f                  # follow
sleipnir logs day-one -n 100              # last 100 lines
sleipnir logs day-one web --type build    # one component, explicit type
```

| Option | Effect |
|--------|--------|
| `-f`, `--follow` | Stream new lines as they arrive |
| `-n`, `--tail <lines>` | Show only the last N lines |
| `--type <type>` | `build` · `deploy` · `run` (default: chosen for you — see below) |

### The log-type default

doctl asks for **run** logs unless told otherwise. But an app with no service, worker, or job has no
runtime, and that request **hangs on a websocket that never delivers** — no error, no output, just a
stall until you kill it. For those apps sleipnir asks for **build** logs instead, and says so:

```console
$ sleipnir logs day-one
  → day-one build logs · Brett Buskirk LLC · context brett (no runtime component — showing build logs)
```

Pass `--type` to override. Note that **starter-tier apps have no deploy logs at all** — DigitalOcean
returns a clear `400` saying so, which sleipnir passes straight through.

---

## `install`

Write a starter config to the resolved config path, with your current defaults filled in. **Refuses to
overwrite an existing config** unless you pass `--force` — that file is hand-tuned.

```sh
sleipnir install
sleipnir install --force    # overwrite an existing config
```

| Option | Effect |
|--------|--------|
| `-f`, `--force` | Overwrite an existing config file |

If doctl is authenticated, your SSH-key **fingerprints** are prefilled from the account — public
identifiers, not secrets, and they never leave the machine. The file is written `0600` and contains no
credentials: doctl owns your DO token.

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
    ssh key     (ssh decides)
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

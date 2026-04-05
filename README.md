# devports

See which dev servers are running on which ports.

When you're juggling multiple projects, it's easy to lose track of what's running where. `devports` scans your listening ports and tells you the **project name**, **framework**, and **directory** for each one — no more guessing which app is on `:3000`.

```
$ devports

  Dev Servers
  ──────────────────────────────────────────────────────────────────────
  PORT    PROJECT                FRAMEWORK    PID      DIRECTORY
  ──────────────────────────────────────────────────────────────────────
  3000    my-api                 Static       12345    ~/projects/my-api
  4321    my-website             Astro        12400    ~/projects/my-website
  5173    dashboard              Vite         12512    ~/projects/dashboard
  8080    backend                Django       12600    ~/projects/backend
  ──────────────────────────────────────────────────────────────────────
  4 server(s) found
```

## Features

- **Project detection** — reads `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, `Gemfile` to find the project name
- **Framework detection** — identifies Next.js, Vite, Astro, Nuxt, Angular, Django, Flask, Rails, Laravel, Hugo, and 15+ more
- **Smart filtering** — hides system processes (Bluetooth, AirDrop, Spotlight, etc.) by default, shows only dev servers
- **Port color coding** — green (3000s), cyan (4000s), blue (5000s), yellow (8000–9999)
- **PostgreSQL integration** — list databases, sizes, active connections, and table details
- **Watch mode** — live dashboard that auto-refreshes every 2 seconds
- **Zero dependencies** — single shell script, uses only `lsof`, `ps`, and `awk`

## Install

### Quick install

```bash
curl -fsSL https://raw.githubusercontent.com/azej11/devports/main/devports -o ~/.local/bin/devports
chmod +x ~/.local/bin/devports
```

Make sure `~/.local/bin` is in your `PATH`. Add this to your `~/.zshrc` if needed:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

### Clone and symlink

```bash
git clone https://github.com/azej11/devports.git
cd devports
chmod +x devports
ln -s "$(pwd)/devports" ~/.local/bin/devports
```

### Homebrew (coming soon)

```bash
brew install devports
```

## Usage

```bash
devports                # List dev servers (filtered, smart defaults)
devports all            # List ALL listening ports (no filtering)
devports 3000           # Show what's on a specific port
devports kill 3000      # Kill whatever is on port 3000
devports db             # List PostgreSQL databases on port 5432
devports db 5433        # List PostgreSQL databases on a custom port
devports watch          # Live-refresh dashboard (2s interval)
devports --version      # Show version
devports help           # Show help
```

### List dev servers

```bash
$ devports
```

The default view auto-filters system processes and shows only likely dev servers. It detects servers by:

1. Matching the process command against known dev runtimes (`node`, `python`, `ruby`, `go`, etc.)
2. Including anything on common dev ports (3000–9999) even if the command isn't recognized
3. Excluding known macOS/Linux system daemons

### Inspect a port

```bash
$ devports 5173

  Port 5173
  ──────────────────────────────────────────────────────────────────────
  PORT    PROJECT                FRAMEWORK    PID      DIRECTORY
  ──────────────────────────────────────────────────────────────────────
  5173    my-app                 Vite         13392    ~/projects/my-app
  ──────────────────────────────────────────────────────────────────────
  1 server(s) found
```

### Kill a port

```bash
$ devports kill 3000

Killing process on port 3000
  Project: my-api
  PID:     50605
  Command: node server.js
Done.
```

Sends `SIGTERM` first. If the process doesn't exit within 1 second, sends `SIGKILL`.

### PostgreSQL databases

```bash
$ devports db

  PostgreSQL Databases (port 5432)
  ──────────────────────────────────────────────────────
  DATABASE             OWNER            SIZE       CONNECTIONS
  ──────────────────────────────────────────────────────
  myapp                john             8606 kB    4
  postgres             john             7606 kB    1
  ──────────────────────────────────────────────────────
  2 database(s)

  myapp tables:
    users                          64 kB
    sessions                       48 kB
    posts                          32 kB
```

Requires `psql` to be installed. The tool auto-discovers it from common Homebrew and system locations.

### Watch mode

```bash
$ devports watch
```

Clears the screen and refreshes every 2 seconds. Press `Ctrl+C` to exit.

## Detected frameworks

| Framework | Detection method |
|-----------|-----------------|
| Next.js | `next` in command, or `next.config.*` in project |
| Vite | `vite` in command, or `vite.config.*` in project |
| Astro | `astro` in command |
| Nuxt | `nuxt` in command, or `nuxt.config.*` in project |
| Angular | `angular`/`ng` in command, or `angular.json` in project |
| SvelteKit | `svelte` in command |
| Remix | `remix` in command |
| Gatsby | `gatsby` in command |
| CRA | `react-scripts` in command |
| Webpack | `webpack-dev` in command |
| Django | `django`/`manage.py` in command |
| Flask | `flask` in command |
| Uvicorn | `uvicorn` in command |
| Rails | `rails`/`puma` in command |
| Laravel | `artisan` in command |
| Hugo | `hugo` in command |
| Turbo | `turbo` in command |
| Bun | `bun` in command |
| Deno | `deno` in command |
| Static | `live-server`/`http-server`/`serve` in command |

## Requirements

- **macOS** or **Linux**
- **zsh** (default shell on macOS since Catalina)
- **lsof** (pre-installed on macOS and most Linux distros)
- **psql** (optional, for the `db` command)

## How it works

1. Runs `lsof -iTCP -sTCP:LISTEN` to get all processes listening on TCP ports
2. For each process, resolves the **working directory** via `lsof -d cwd`
3. Walks up the directory tree to find `package.json`, `Cargo.toml`, `go.mod`, etc. and extracts the **project name**
4. Matches the running command against known framework signatures to detect the **framework**
5. Deduplicates IPv4/IPv6 entries and filters out system processes

## License

MIT

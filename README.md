# wp-cli-local (Antigravity Edition)

An AI agent skill that runs [WP-CLI](https://wp-cli.org/) commands against [Local](https://localwp.com/) (by Flywheel) sites on macOS and Linux.

- **Auto-detects** the Local site from your current directory (supporting tilde `~/` path expansions on Linux)
- Falls back to `--site=<name>` when outside a site folder
- Uses Local's own PHP and MySQL binaries with the correct `php.ini` and shared libraries path (`LD_LIBRARY_PATH`)

<div align="center">
<img width="739" height="656" alt="Screenshot 2026-03-12 at 00 54 40" src="https://github.com/user-attachments/assets/f975a023-9b17-421c-b88a-576f03b4513a" />
<p><i>More <a href="#example-prompts">example prompts</a> below</i></p>
</div>

## Installation

```bash
npx skills add https://github.com/soderlind/wp-cli-local --skill wp-cli-local -g
```

The `--all` flag installs the skill for all detected AI agents (GitHub Copilot, Claude Code, Cursor, Cline, etc.) without prompting:

```bash
npx skills add https://github.com/soderlind/wp-cli-local --skill wp-cli-local -g --all
```

Without `--all`, you'll be prompted to select which agents to install for.

## Prerequisites

- **macOS** (Apple Silicon or Intel) or **Linux**
- **[Local](https://localwp.com/)** installed with at least one site
- **[WP-CLI](https://wp-cli.org/)** installed and in `PATH` (on Linux, it automatically falls back to Local's bundled WP-CLI phar at `/opt/Local/resources/extraResources/bin/wp-cli/wp-cli.phar` if not in `PATH`)
- The target site must be **running** in Local

## How It Works

The wrapper script at `scripts/wp`:

1. Reads the Local sites configuration:
   - macOS: `~/Library/Application Support/Local/sites.json`
   - Linux: `~/.config/Local/sites.json`
2. Matches the current working directory against site paths, automatically expanding `~/` prefixes for Linux compatibility.
3. Resolves the correct PHP and MySQL binaries from Local's `lightning-services/` using the system architecture/OS path (`linux` on Linux, `darwin*` on macOS).
4. Loads the site-specific `php.ini` (which contains the MySQL socket path).
5. Sets `MYSQL_UNIX_PORT` so `wp db` commands use the correct socket.
6. Configures `LD_LIBRARY_PATH` on Linux to include PHP's `shared-libs` directory, preventing dynamic link issues (e.g. missing `libtidy.so.5deb1`).
7. Executes WP-CLI with the correct environment.

## Usage

Once installed, just ask your AI agent to run WP-CLI commands in natural language. The skill is triggered automatically.

### Example prompts

- "List all plugins on my Local site"
- "Activate the woocommerce plugin"
- "What version of WordPress is running?"
- "Flush the cache"
- "Export the database"
- "Show all options matching `siteurl`"
- "List scheduled cron events"
- "Run `wp db query \"SELECT * FROM wp_options LIMIT 5;\"`"

If your working directory is inside a Local site folder, the site is detected automatically. Otherwise the agent will ask which site to target, or you can specify it: "List plugins on the **my-site** site".

### Manual / testing

You can also call the wrapper script directly in a terminal:

```bash
# Auto-detect site from CWD
bash scripts/wp plugin list

# Explicit site name
bash scripts/wp --site=my-site core version

# List all Local sites with running/halted status
bash scripts/wp --list
```

### Example output of `--list`

```
Local by Flywheel sites:

  ● plugins              running    /Users/you/Sites/plugins
  ○ my-site              halted     /Users/you/Sites/my-site
```

## Antigravity Edition

This fork is optimized to run as an **Agent Skill** inside the **Antigravity** coding environment on **Linux**. It introduces key fixes for Linux/LocalWP environments:
- **Tilde Expansion**: Auto-detection resolves Linux `~/Local Sites` paths by properly expanding the home directory in Python helpers.
- **Dynamic Linker Config**: Resolves PHP core binary shared library dependencies on Linux by dynamically exporting `LD_LIBRARY_PATH` to point to Local's PHP service `shared-libs`.
- **Bundled WP-CLI Fallback**: Gracefully falls back to `/opt/Local/resources/extraResources/bin/wp-cli/wp-cli.phar` if WP-CLI is not globally installed.

### Installation in Antigravity

To register this skill globally inside your **Antigravity CLI and GUI**, run the provided installation script:

```bash
# If installed via npx skills:
bash ~/.agents/skills/wp-cli-local/scripts/install-antigravity

# Or if you cloned this repository locally:
bash skills/wp-cli-local/scripts/install-antigravity
```

This script links the skill to Antigravity's global configuration directory (`~/.gemini/config/skills/wp-cli-local`), making it permanently available.

## Credit

Inspired by [local-wp-cli](https://github.com/aslamdoctor/local-wp-cli) skill, but rewritten from scratch with better site detection and MySQL socket support.

## License

MIT

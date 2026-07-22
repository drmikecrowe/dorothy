# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

### Testing, Linting, and Formatting
```bash
dorothy test      # Run tests
dorothy lint      # Format and check code
dorothy check     # Format check only
dorothy format    # Auto-format changes
dorothy dev       # Install development dependencies
```

### Dorothy Management
```bash
dorothy commands  # List all available Dorothy commands
dorothy install   # Install/configure Dorothy and shells
dorothy update    # Pull updates for Dorothy and user config
```

### Setup Commands
```bash
setup-system install    # Full system setup
setup-system update     # Update system packages
setup-util <name>       # Install a utility (cross-platform)
setup-util --cli=tool APK=tool APT=tool BREW=tool  # Manual utility installation
```

### Using npm scripts
```bash
npm run our:test        # Run tests
npm run our:verify     # Run linting and checks
```

## Project Architecture

Dorothy is a cross-platform dotfile ecosystem that provides automation and configuration management across multiple operating systems, architectures, and shells.

### Directory Structure

```
$DOROTHY (default: ~/.local/share/dorothy)
├── commands/              # Stable production-ready commands
├── commands.beta/         # Beta-quality commands
├── commands.deprecated/   # Deprecated commands
├── config/               # Default configuration files
├── sources/              # Shell initialization scripts
│   ├── bash.bash         # Bash strict mode, utilities, shims
│   ├── environment.sh    # Environment configuration (for login shells)
│   └── interactive.sh    # Interactive config (for login+interactive shells)
├── themes/               # Theme configurations
├── init.sh               # POSIX shell initialization (Bash, Zsh, Dash, KSH)
├── init.fish             # Fish shell initialization
├── init.nu               # Nushell initialization
├── init.xsh              # Xonsh initialization
└── user -> ~/.config/dorothy/  # User's personal configuration (symlink)
```

### Shell Initialization Flow

When a login shell starts:
1. Shell loads the appropriate `init.*` script (init.sh for POSIX shells)
2. If login shell: loads `sources/environment.sh`
   - Runs `setup-environment-commands`
   - Applies system environment configuration
   - Loads `user/config(.local)/environment.bash` if exists
3. If login+interactive shell: loads `sources/interactive.sh`
   - Loads `user/config(.local)/interactive.*` (shell-specific)
   - Loads common utilities, themes, SSH config, autocomplete

### Configuration Precedence (highest to lowest)

1. `user/config.local/` - Private configuration (git ignored)
2. `user/config/` - Public user configuration (git tracked)
3. `config/` - Dorothy defaults
4. System defaults

### The `setup-util` Ecosystem

The `setup-util` command is an intelligent wrapper around package managers. It:
- Detects available package managers (brew, apt, dnf, pacman, snap, npm, cargo, etc.)
- Falls back through alternatives based on platform availability
- Supports optional vs required dependencies
- Handles cross-platform installation differences

The `sources` array in `setup-util` defines package manager preference order. Modify this when changing installation behavior.

### Key Architectural Patterns

**Command Structure:**
- Filename uses dashes: `my-command`
- Function name uses underscores: `my_command` or `my_command_` (if single word)
- Commands are subshells: `function cmd_name () (` to prevent environment leakage
- Always source `$DOROTHY/sources/bash.bash` at start of bash commands

**Helper Functions in `sources/bash.bash`:**
- `eval_capture` - Capture exit status and output
- `__require_globstar` - Require bash globstar support
- `__require_array` - Require empty array support
- `__command_exists` - Check if command is available

**Source Files:**
- `sources/bash.bash` - Core bash utilities, strict mode, shims
- `sources/styles.bash` - Output styling utilities
- `sources/stdinargs.bash` - Standard input/argument handling for transformers

## Important Conventions

### Code Style
- Use **tabs** for leftmost indentation
- Use **spaces** for rightmost indentation (e.g., inside `<<-EOF` blocks)
- Commands/functions: `lower_case`
- Environment variables: `UPPER_CASE`
- Local variables: `lower_case`
- Always use `local` for non-global variables
- Wrap values in quotes: single quotes if no interpolation, double quotes if interpolation needed
- Always quote variable usage: `"$var"`
- Prefer `[[` for bash conditionals, avoid `test`
- Always use explicit `if ... then` statements, avoid `&&`/`||` magic
- Use specific exit codes (see `docs/bash/errors.md`), not just `1`

### Command Types
1. **Generic** - Process arguments, execute something
2. **Installer** - `setup-util-*` commands for cross-platform installation
3. **Transformer** - Transform input (args/stdin), like `echo-*` commands

### User Commands Override
User commands (`user/commands/`) take precedence over Dorothy's built-in commands. This allows users to extend or override built-in functionality.

### Package Manager Detection
Dorothy intelligently detects and uses available package managers. The `setup-util` command contains a `sources` array defining preference order across platforms.

## Key Commands Reference

- `dorothy` - Main entry point, manipulation of Dorothy ecosystem
- `setup-*` - System setup commands (git, dns, shell, etc.)
- `setup-util-*` - Cross-platform utility installation
- `config-*` - Configuration helpers
- `is-*` - System detection utilities
- `get-*` - Information retrieval utilities
- `echo-*` - Output transformation utilities
- `fs-*` - Filesystem utilities
- `ask`/`confirm`/`choose` - User input prompts
- `edit` - Open file in preferred editor
- `secret` - Secure secret management via 1Password

## Shell Support

Dorothy supports: Bash, Zsh, Fish, Nu (Nushell), Xonsh, Elvish, Dash, KSH

Each shell has its own initialization script that loads the appropriate sources.

## Documentation

- `docs/scripting/conventions.md` - Code style conventions
- `docs/scripting/commands.md` - Command structure guide
- `docs/bash/` - Bash-specific documentation
- `docs/dorothy/` - Dorothy-specific documentation

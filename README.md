# Helix + Zellij + AI + REPL Workflow

This repository provides a complete workflow integration for modern development
using Helix editor, Zellij terminal multiplexer, AI CLI apps, and REPL
environments. The workflow is built around three scripts with minimal
dependencies:

## Overview

- **`haico`** - AI-powered code completion for Helix editor supporting OpenAI,
  Gemini, Codestral and Claude providers
- **`zqantara`** - REPL bridge between Helix and Zellij with bracketed paste
  mode support for all Zellij window types
- **`symbol-search`** - Workspace-wide symbol completion using command line tools

This workflow enables seamless integration between:

- Code editing in Helix with AI-powered completions
- REPL interaction for Python, R, Shell, and other languages
- AI CLI applications like Aider and Claude Code
- Multiplexing: Manage multiple REPLs and processes efficiently using Zellij's tabs, panes, and floating windows.

All scripts are designed with minimal dependencies and focus on providing
efficient, keyboard-driven workflows.

> **💡 Tip**: Use `haico --help`, `zqantara --help`, and `symbol-search --help` to view the complete
> documentation of all supported command-line arguments and options.

## Installation

1. Clone this repository
2. Copy `haico`, `zqantara`, and `symbol-search` scripts to your PATH
3. For `symbol-search` to function without `fzf`, copy the `picker` script to
   your `PATH` as well.
4. Set up the required API keys for `haico`
5. Configure Helix with the provided key bindings
6. Ensure Zellij is installed and running

## Video Showcase

See the workflow in action:

### AI Code Completion

https://github.com/user-attachments/assets/f1d1c159-94da-4fd9-8ea7-986738e7ac33

### Claude Code Integration

https://github.com/user-attachments/assets/dc7085c1-6a74-4943-b417-879697996b09

### Zellij Tab REPL

https://github.com/user-attachments/assets/7a585708-d6b7-4eaf-83c2-5ea4c411b815

### Zellij Pane REPL

https://github.com/user-attachments/assets/cfbe4f99-937d-4130-b8ae-937d4ad58fda

### Zellij Floating REPL

https://github.com/user-attachments/assets/9f644b01-cbdc-4ac1-bdfd-079ab54d2592

## haico - Helix AI COmpletion

`haico` provides AI-powered code completion that integrates seamlessly with
Helix editor, offering intelligent code suggestions using various AI providers.

### Features

- Multi-provider support: Works with OpenAI, Gemini, Codestral, FIM, and Claude APIs
- Recommended model: `gemini-2.0-flash` or `codestral` for optimal performance and speed
- Minimal dependencies: Only requires `jq` and standard Unix tools
- Context-aware: Understands cursor position, file context, and programming language
- Two-step workflow: Designed to work around Helix's limitations with buffer content and cursor position handling

### Dependencies

- `jq` - JSON processor (only external dependency)
- Standard Unix tools (`curl`, `grep`, `awk`, `sed`)

### Setup

Set the appropriate API key based on your chosen provider:

```bash
# For Gemini (recommended)
export GEMINI_API_KEY="your-api-key"

# For OpenAI
export OPENAI_API_KEY="your-api-key"
export OPENAI_BASE_URL="https://openrouter.ai/api/v1"  # optional, for custom endpoints

# For Claude
export ANTHROPIC_API_KEY="your-api-key"
```

### Helix Integration

Add these key bindings to your Helix configuration:

```toml
[keys.insert]
# long completion
A-y = [
    "save_selection",
    "select_all",
    ":pipe-to haico --prepare --file ~/.cache/haico/data.txt",
    "jump_backward",
    """:insert-output \
            haico \
            --cursor-line %{cursor_line} \
            --cursor-column %{cursor_column} \
            --language %{language} \
            --buffer-name %{buffer_name} \
            --max-tokens 128 \
            --file ~/.cache/haico/data.txt
    """,
    "align_view_center"
]

# short (single-line) completion
"A-'" = [
    "save_selection",
    "select_all",
    ":pipe-to haico --prepare --file ~/.cache/haico/data.txt",
    "jump_backward",
    """:insert-output \
            haico \
            --provider codestral \
            --cursor-line %{cursor_line} \
            --cursor-column %{cursor_column} \
            --language %{language} \
            --buffer-name %{buffer_name} \
            --file ~/.cache/haico/data.txt \
            --stop '["\\n"]'
    """,
    "align_view_center"
]
```

**Note**: Claude models do not support newline/whitespace as stop sequences, so
single-line completion is not available with Claude.

## zqantara - Zellij REPL Bridge

Named after the Arabic "qantara" meaning bridge, `zqantara` bridges the gap
between your Helix editor and REPL environments in Zellij, enabling seamless
code execution and AI CLI tool integration.

### Features

- Bracketed paste mode support\*\*: **Essential feature** for REPL integration utility
- Complete Zellij mode support: Works with all Zellij window types:
  - Tab mode: Routes content to named tabs, creating them if needed
  - Floating window mode: Routes content to floating windows by position
  - Pane mode: Routes content to panes in specified positions
- Zero dependencies: Uses only standard Unix tools (`grep`, `bash`)
- Handle REPL creation: This script can also handle creating new
  tabs/panes/windows with specified programs
- CLI integration: Can also used for for Non REPL CLI tools like yazi, lazygit,
  Aider and Claude Code

### Dependencies

- Zero external dependencies - uses only standard Unix tools

### Helix Integration

#### Opening REPL Programs and AI Tools

```toml
# Open in tabs
[keys.normal.space.o]
g = ":sh zqantara --tab --name lazygit --program lazygit"
p = ":sh zqantara --tab --name ipython --program 'uv run ipython'"
r = ":sh zqantara --tab --name radian --program radian"
c = ":sh zqantara --tab --name aichat --program 'aichat -s'"
C = ":sh zqantara --tab --name claude --program claude"
a = ":sh zqantara --tab --name aider --program aider"

# Open in floating windows
[keys.normal.space.o.f]
p = ":sh zqantara --floating --program 'uv run ipython'"
t = ":sh zqantara --floating --program $SHELL"
c = ":sh zqantara --floating --program aichat"

# Open in panes (embedded windows)
[keys.normal.space.o.w]
p = ":sh zqantara --pane --pos down --program ipython"
t = ":sh zqantara --pane --pos right --program $SHELL"
c = ":sh zqantara --pane --pos stacked --program aichat"
```

#### Sending Content to REPLs and AI Tools

```toml
[keys.normal.space.space]
# Send to named tabs
p = ":pipe-to zqantara --tab --name ipython"
r = ":pipe-to zqantara --final-return false --tab --name radian"
c = ":pipe-to zqantara --tab --name aichat"
C = ":pipe-to zqantara --final-return 0.1 --tab --name claude"
t = ":pipe-to zqantara --tab --name shell"

# Send to floating windows by position
# 'current' means use current floating window without switching position
"1" = ":pipe-to zqantara --floating --pos current"
"2" = ":pipe-to zqantara --floating --pos up"
"3" = ":pipe-to zqantara --floating --pos down"

# Send to panes by position
"l" = ":pipe-to zqantara --pane --pos 'down right'"
"h" = ":pipe-to zqantara --pane --pos 'down left'"

[keys.select.space.space]
# Send to named tabs
p = ":pipe-to zqantara --tab --name ipython"
r = ":pipe-to zqantara --final-return false --tab --name radian"
c = ":pipe-to zqantara --tab --name aichat"
C = ":pipe-to zqantara --final-return 0.1 --tab --name claude"
t = ":pipe-to zqantara --tab --name shell"

# Send to floating windows by position
# 'current' means use current floating window without switching position
"1" = ":pipe-to zqantara --floating --pos current"
"2" = ":pipe-to zqantara --floating --pos up"
"3" = ":pipe-to zqantara --floating --pos down"

# Send to panes by position
"l" = ":pipe-to zqantara --pane --pos 'down right'"
"h" = ":pipe-to zqantara --pane --pos 'down left'"
```

### Recommended Zellij Configuration

If you're using a terminal emulator that supports the Kitty keyboard protocol
(such as Kitty or Ghostty), I recommend add the following line to your
`config.kdl`:

```kdl
support_kitty_keyboard_protocol false
```

Without this configuration, certain Helix keybindings that require `Alt+Shift`
combinations will not work properly. This includes keys like `Alt+C` (typed as
`Alt+Shift+c`) and `Alt+*` (typed as `Alt+Shift+8`). For technical details
about this issue, see Zellij Issue #4148.

### Current Limitations

Due to Zellij's API limitations, the script cannot retrieve pane names or floating window names:

- Content can only be sent to floating windows by **position** (not by name)
- Content can only be sent to panes by **position** (not by name)
- Cannot accurately target a specific pane/floating window by its desired name

### Why we need Bracketed Paste Mode?

<details>

Bracketed-paste mode wraps pasted text in escape sequences, letting terminals
distinguish it from typed input. Most modern REP Ls and command-line
applications support this feature, which offers several advantages:

- Ensures multi-line code blocks are pasted as a single unit
- Prevent REPLs from misinterpreting pasted newlines or special characters.
- Maintains proper indentation and formatting

</details>

## symbol-search - Workspace Symbol Completion

`symbol-search` provides workspace-wide symbol completion for Helix editor
through command line tools integration. While it can work with only standard
Unix tools, it's designed to leverage advanced command line utilities like
universal-ctags, ripgrep, and fzf for better user experience. It supports
multiple search backends: universal-ctags for tag-based symbol indexing,
`ripgrep` for fast text searching, and `grep` as a fallback. The script
gracefully falls back to standard Unix tools using the included `picker` script
when external dependencies are unavailable.

### Features

- **Adaptive toolchain**: Leverages ripgrep, fzf, and universal-ctags when
  available; falls back to grep and the dependency-free `picker` script
- **Workspace-wide completion**: Symbol search across the entire project, not
  limited to the current file
- **Zero external dependencies**: Complete functionality using only standard
  Unix tools when needed

### Comparison with simple-language-server

| Feature          | symbol-search                               | simple-language-server                               |
| ---------------- | ------------------------------------------- | ---------------------------------------------------- |
| **Dependencies** | Standard Unix tools or common CLI utilities | Requires Rust toolchain for compilation              |
| **Installation** | Ready-to-use shell script                   | Binary compilation required (Nix packages available) |
| **Scope**        | Workspace-wide symbol completion            | Files opened in editor session                       |
| **Integration**  | External picker via `:pipe` command         | Native LSP auto-completion                           |
| **Features**     | Symbol completion only                      | Symbol + snippet completion, more LSP features       |
| **Portability**  | Works anywhere with basic Unix tools        | Requires pre-built binary or compilation             |

### Helix Integration

Configure symbol completion with these polished key bindings:

```toml
[keys.insert.C-x]
# Macro ensures proper word selection before symbol search
"C-x" = "@<esc>hmiw<C-x>a"

[keys.normal]
"C-x" = [
    ":pipe symbol-search --rg --tags .tags",
]

[keys.select]
"C-x" = [
    ":pipe symbol-search --rg --tags .tags",
]
```

The insert mode configuration uses a macro (`@<esc>hmiw<C-x>a`) to ensure
complete word selection under the cursor before triggering symbol search. This
approach provides more reliable completion context than command sequences,
which cannot accurately select word boundaries (`miw` operation).

## Complete Workflow

This setup provides a comprehensive development environment:

1. Code in Helix with AI-powered completions via `haico`
2. Execute code in REPLs via `zqantara` with proper bracketed paste
3. Access AI tools like Aider and Claude Code through organized Zellij sessions
4. Manage multiple CLI Apps with tabs, panes, and floating windows
5. Maintain context across all tools with minimal context switching

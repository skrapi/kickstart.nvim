# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal Neovim configuration forked from [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim). There is no build step and no test suite — it is config code loaded by Neovim at startup. "Running" it means launching `nvim`; mistakes surface as Lua errors on startup or via `:checkhealth`.

**Requires Neovim 0.12+.** The config uses the built-in `vim.pack` plugin manager and native `vim.lsp.config`/`vim.lsp.enable`, neither of which exists on 0.11. `lua/kickstart/health.lua` enforces this.

## Commands

- Format Lua: `stylua .` (settings in `.stylua.toml`: 2-space indent, single quotes, no call parens, 160 col). `stylua` is installed via Mason inside Neovim, so it may not be on the shell `PATH` until after the first launch.
- Check formatting without writing: `stylua --check .`
- Verify config after a change: `nvim --headless +q` (exits non-zero / prints errors on a broken config). To syntax-check only without executing: `nvim --headless -c "lua assert(loadfile('init.lua'))" -c "qa"`.
- Plugins (inside Neovim): `:lua vim.pack.update()` to update, `:lua vim.pack.update(nil, { offline = true })` to inspect state offline. `:Mason` for LSP servers & tools. `:checkhealth` (`:checkhealth kickstart` for this config's own check).

The `.github/workflows/stylua.yml` action is inherited from upstream and gated to the `nvim-lua/kickstart.nvim` repo, so it does **not** run on this fork — run `stylua` locally before committing.

## Architecture

- **`init.lua`** is the entire config. It is split into nine `do ... end` blocks labelled `SECTION 1` … `SECTION 9` (Foundation, Plugin-manager intro, UI plugins, Search/nav, LSP, Formatting, Autocomplete, Treesitter, Optional examples). The `do`-blocks scope locals so sections stay independent. Almost every change lands in one of these sections.
- **Plugins use `vim.pack`, not `lazy.nvim`.** There is no central plugin table. To add a plugin: call `vim.pack.add { gh 'owner/repo' }` in the relevant section, then `require('plugin').setup { ... }` immediately after. `gh()` (defined after SECTION 2) just prepends `https://github.com/`. `vim.pack.add` is imperative and de-duplicates, so a shared dependency (e.g. `plenary.nvim`) can be listed by multiple sections safely. Build steps (fzf-native, LuaSnip, treesitter) are handled by the `PackChanged` autocommand in SECTION 2.
- Plugin versions are pinned in **`nvim-pack-lock.json`** (tracked in this fork; `.gitignore` deliberately un-ignores it).
- **`lua/kickstart/plugins/`** — optional upstream plugin modules, activated only when explicitly `require`d at the bottom of SECTION 9. Currently only `autopairs` is enabled; `debug`, `indent_line`, `lint`, `neo-tree`, `gitsigns` exist but are commented out.
- **`lua/custom/plugins/`** — `init.lua` there auto-`require`s every other `*.lua` file in the directory, but the `require 'custom.plugins'` line in SECTION 9 is commented out, so the directory is currently inert. New customizations are added inline to `init.lua` instead.
- **`lua/kickstart/health.lua`** — backs `:checkhealth kickstart`.

## LSP setup (SECTION 5)

- The `servers` table maps language-server names (`vim.lsp.Config` entries) to per-server overrides. `vim.tbl_keys(servers)` is fed to `mason-tool-installer` for auto-install, and the `for name, server` loop calls `vim.lsp.config(name, server)` + `vim.lsp.enable(name)`. **To add a server, add an entry to `servers`** — install and enable are automatic.
- Pure tools that are *not* language servers (formatters, etc.) go in the `vim.list_extend(ensure_installed, { ... })` list, **not** in `servers` — currently `prettier` lives there.
- **TypeScript is the exception**: handled by `typescript-tools.nvim` (set up at the end of SECTION 5), not by `ts_ls` in the `servers` table.
- `julials` swaps `cmd[1]` to a dedicated julia binary, if present, by patching the resolved `vim.lsp.config.julials.cmd` after the enable loop.
- Diagnostics render as **virtual lines** (`vim.diagnostic.config` in SECTION 1) — the built-in replacement for the old `lsp_lines.nvim` plugin.

## Formatting (SECTION 6)

`conform.nvim` formats on save for every filetype **except C/C++** (`disable_filetypes` in `format_on_save`). `formatters_by_ft` maps Lua → `stylua` and the web filetypes → `prettier`. A new formatter must also be installed by Mason — add it to the `ensure_installed` list in SECTION 5.

## Local customizations beyond stock kickstart

Easy to miss, and what distinguishes this fork:
- Wayland clipboard via `wl-clipboard` (`vim.g.clipboard` block in SECTION 1; assumes `wl-copy`/`wl-paste` exist).
- `mouse = 'n'` — mouse enabled only in normal mode.
- Keymaps: `jk` exits insert mode; `<C-d>`/`<C-u>` re-centre the view; `<leader>?` shows buffer-local keymaps.
- `gruvbox` colorscheme (replaces upstream's `tokyonight`).
- Extra plugins added inline: `neogit` (+ `diffview`), `vim-slime` (tmux target), `obsidian.nvim` (vault at `~/vaults/sylv_sync/`), `typescript-tools.nvim`.
- Web language servers (`html`, `cssls`, `emmet_ls`) and `prettier` added for frontend work.

## Conventions

- Match the existing kickstart comment style and density when editing `init.lua` — it is heavily annotated by design.
- Keep `stylua` formatting clean. Commit `nvim-pack-lock.json` changes deliberately (a lockfile bump is a real change, not noise).

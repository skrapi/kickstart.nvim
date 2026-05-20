# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal Neovim configuration forked from [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim). There is no build step and no test suite — it is config code loaded by Neovim at startup. "Running" it means launching `nvim`; mistakes surface as Lua errors on startup or via `:checkhealth`.

## Commands

- Format Lua: `stylua .` (settings in `.stylua.toml`: 2-space indent, single quotes, no call parens, 160 col)
- Check formatting without writing: `stylua --check .`
- Verify config after a change: `nvim --headless +q` (exits non-zero / prints errors on a broken config)
- Plugin management (inside Neovim): `:Lazy` (status/update/sync), `:Mason` (LSP servers & tools), `:checkhealth` (`:checkhealth kickstart` for this config's own check in `lua/kickstart/health.lua`)

The `.github/workflows/stylua.yml` action is inherited from upstream and is gated to `nvim-lua/kickstart.nvim`, so it does **not** run on this fork — run `stylua` locally before committing.

## Architecture

- **`init.lua`** is the entire config: vim options, keymaps, autocommands, and one large `require('lazy').setup({ ... })` plugin table. Almost every change lands here. Each plugin is a table entry; `opts` is passed to the plugin's `setup()`, `config = function()` runs custom setup. Plugin versions are pinned in `lazy-lock.json` (committed; `:Lazy update` regenerates it).
- **`lua/kickstart/plugins/`** — optional upstream plugin modules. They activate only when explicitly `require`d near the bottom of `init.lua`'s setup table. Currently only `autopairs` is enabled; `debug`, `indent_line`, `lint`, `neo-tree`, `gitsigns` exist but are commented out.
- **`lua/custom/plugins/`** — intended home for user plugins via `{ import = 'custom.plugins' }`, but that import line is commented out and `init.lua` is **not** loaded. New plugins are currently added inline to the `init.lua` setup table instead.
- **`lua/kickstart/health.lua`** — backs `:checkhealth kickstart`.

## LSP setup

LSP is configured in the `nvim-lspconfig` block of `init.lua`:
- `mason-tool-installer` installs servers/tools listed in `ensure_installed` (currently `stylua`, `html-lsp`, `css-lsp`, `emmet-ls`, `prettier`, plus the keys of the `servers` table).
- The `servers` table holds per-server overrides; explicitly configured servers are `julials`, `elmls`, `lua_ls`. To add a server, add an entry there — it is auto-installed and set up via the `mason-lspconfig` handler.
- **TypeScript is the exception**: it is handled by `typescript-tools.nvim`, not `nvim-lspconfig`. Do not add `ts_ls`/`tsserver` to the `servers` table.
- Formatting is done by `conform.nvim` (format-on-save, disabled for c/cpp). `formatters_by_ft` maps filetypes to formatters (`stylua` for Lua, `prettier` for web filetypes). New formatters must also be added to `ensure_installed` so Mason installs them.

## Local customizations beyond stock kickstart

These distinguish this fork from upstream and are easy to miss:
- Wayland clipboard via `wl-clipboard` (set in a `vim.schedule` block; assumes `wl-copy`/`wl-paste` exist).
- `mouse = 'n'` — mouse enabled only in normal mode.
- Keymaps: `jk` exits insert mode; `<C-d>`/`<C-u>` re-centre the view.
- Extra plugins added inline: `neogit` (+ `diffview`), `vim-slime` (tmux target), `obsidian.nvim` (vault at `~/vaults/sylv_sync/`), `lsp_lines.nvim`, `gruvbox` colorscheme.

## Conventions

- Match the existing kickstart comment style and density when editing `init.lua` — it is heavily annotated by design.
- Keep `stylua` formatting clean; commit `lazy-lock.json` changes deliberately (a lockfile bump is a real change, not noise).

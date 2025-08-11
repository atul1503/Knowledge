## Neovim: Editor Commands and Configuration

This note teaches basic Neovim editor commands and how to configure Neovim using Lua. The language is simple. Every new term is explained the first time it appears.

### What is Neovim?

- Neovim is a text editor. It is a modern version of Vim. It runs in your terminal.
- Neovim has different "modes". A mode changes what keys do.
  - Normal mode: for moving around and running commands. Press `Esc` to enter Normal mode.
  - Insert mode: for typing text. Press `i` in Normal mode to start Insert mode.
  - Visual mode: for selecting text. Press `v` in Normal mode to start Visual mode.
  - Command-line mode: for `:` commands (like save or quit). Press `:` in Normal mode to start Command-line mode.

When this note says "press a key", it always means: be in Normal mode unless it says otherwise.

## Moving the cursor (navigation)

In Neovim, you can put a number before a motion to repeat it. This number is called a "count". Example: `5k` means "go up 5 lines".

### Basic arrows

- `h`: move left by one character.
- `l`: move right by one character.
- `j`: move down by one line.
- `k`: move up by one line.

You can add a count. Example: `10h` moves left 10 characters.

### Words

Here a "word" is a group of letters, numbers, or underscore `\_` without spaces between them.

- `w`: go to the start of the next word.
- `b`: go to the start of the previous word. This is a way to go back by words.
- `e`: go to the end of the current or next word.
- `ge`: go to the end of the previous word. This is also going back by words.

You can add a count. Example: `3b` goes back 3 words.

### Line starts and ends

- `0` (zero): go to the very start of the line (column 1), even if there are spaces.
- `^` (caret): go to the first non-space character in the line.
- `$`: go to the end of the line.

These are useful to quickly move within the current line. For going back inside the same line, `0`, `^`, and motions like `b`, `ge`, `F`, and `T` (explained next) help a lot.

### Find a character on the same line

- `f{char}`: go forward to the next `{char}` on the same line.
- `F{char}`: go backward to the previous `{char}` on the same line. This is a way to go back inside a line.
- `t{char}`: go forward to just before `{char}` on the same line.
- `T{char}`: go backward to just after `{char}` on the same line. This is also going back inside a line.
- `;`: repeat the last `f`, `F`, `t`, or `T` in the same direction.
- `,`: repeat the last `f`, `F`, `t`, or `T` in the opposite direction.

In the above, `{char}` means "a single character you type". Example: `F,` moves backward to the previous comma on the line.

### Paragraphs

Here a "paragraph" is a block of text separated by an empty line.

- `}`: go to the start of the next paragraph.
- `{`: go to the start of the previous paragraph. This is going back by paragraphs.

You can add a count. Example: `2{` goes back two paragraphs.

### Screens and pages

- `Ctrl-d`: move down half a screen (half page).
- `Ctrl-u`: move up half a screen (half page). This is going back by a large step.
- `Ctrl-f`: move forward one full screen (page down).
- `Ctrl-b`: move backward one full screen (page up). This is going back by a page.

On some keyboards, `Ctrl` is written as `^`. For example, `^U` means hold the Control key and press `u`.

### Go to specific lines

- `gg`: go to the first line of the file.
- `G`: go to the last line of the file.
- `{count}G`: go to line number `{count}`. Example: `42G` jumps to line 42.

### Jump back to previous locations

Neovim keeps a "jump list". A jump is a big movement, like a search, a `G`, or opening a file. You can move back and forward through this list:

- `Ctrl-o`: jump back to the previous location (earlier in time). This is a general "go back".
- `Ctrl-i`: jump forward to the next location (later in time). This undoes the last `Ctrl-o`.

Neovim also remembers the position before the most recent jump:

- `''` (two single quotes): go back to the previous line before the last jump.
- `` `` `` (two backticks): go back to the exact previous cursor position (line and column) before the last jump.

Neovim stores a "change list". It is a list of places where you edited text:

- `g;`: jump to the previous change (go back through edits).
- `g,`: jump to the next change (go forward through edits).

### Summary: ways to go back

- Back in the same line: `h` (left), `b` (to start of previous word), `ge` (to end of previous word), `0` (line start), `^` (first non-space), `F{char}` (to previous char), `T{char}` (to just after previous char).
- Back one or more paragraphs: `{` (use a count like `3{` for three paragraphs).
- Back several lines: `{count}k` (example: `7k`), or `Ctrl-u` (half page up), or `Ctrl-b` (full page up).
- Back to a previous place in the file: `Ctrl-o` (jump back), `''` (previous line), `` `` `` (previous exact spot), `g;` (previous change).

## Editing basics

These commands change text. Most of them are used in Normal mode.

- `i`: switch to Insert mode just before the cursor (you can now type text). Press `Esc` to go back to Normal mode.
- `a`: switch to Insert mode just after the cursor.
- `o`: open a new line below and enter Insert mode.
- `O`: open a new line above and enter Insert mode.
- `x`: delete the character under the cursor.
- `dd`: delete the current line.
- `yy`: copy ("yank") the current line to the clipboard inside Neovim.
- `p`: paste the last yanked or deleted text after the cursor.
- `P`: paste the last yanked or deleted text before the cursor.
- `u`: undo the last change.
- `Ctrl-r`: redo the last undone change.

Operator + motion (powerful pattern):

- `d{motion}`: delete text moved over by `{motion}`. Example: `dw` deletes from the cursor to the start of the next word.
- `c{motion}`: change (delete then enter Insert mode). Example: `ciw` changes the "inner word" under the cursor. Here `iw` is a "text object" that means the word text without surrounding spaces.
- `y{motion}`: yank (copy). Example: `y$` yanks to the end of line.

In these, a "motion" is any move command like `w`, `b`, `0`, `$`, `}` etc. You can add a count. Example: `d3w` deletes three words forward. The idea is: first you say what to do (delete/change/yank), then you say where to move to (the motion), and Neovim applies the action over that area.

## Search

- `/text` then `Enter`: search forward for `text`.
- `?text` then `Enter`: search backward for `text`. This is also a way to go back to a previous match.
- `n`: move to the next match.
- `N`: move to the previous match (goes the opposite direction of the last search).

## Save and quit

- `:w` then `Enter`: save (write) the file.
- `:q` then `Enter`: quit the file if there are no unsaved changes.
- `:wq` then `Enter`: save and quit.
- `:q!` then `Enter`: quit and discard unsaved changes.
- `:qa` then `Enter`: quit all files (if no unsaved changes). Use `:qa!` to force quit all.

## How Neovim relates to LazyVim and others

- Neovim is the core editor. It is the engine.
- A "plugin manager" is a tool that installs and loads plugins (extra features). Examples: `lazy.nvim` and `packer.nvim`.
- A "starter" or "distribution" is a pre-built configuration for Neovim. It includes settings, keymaps, plugins, and looks. Examples: LazyVim, NvChad, AstroNvim, LunarVim.

Relationship:

- LazyVim is not Neovim. LazyVim is a starter that sits on top of Neovim. It uses the `lazy.nvim` plugin manager under the hood. It gives you many defaults and plugins already set up.
- NvChad, AstroNvim, and LunarVim are similar ideas. They are ready-made configs built on top of Neovim. They pick plugins, set keymaps, and set options for you.
- You can use Neovim without any starter. You can write your own config from scratch with Lua. You can also start with a starter and then edit it.

## Where Neovim loads configuration from

Neovim looks for a main config file named `init.lua` (Lua) or `init.vim` (Vimscript).

On macOS and Linux, Neovim reads these paths in this order:

- If the environment variable `XDG_CONFIG_HOME` is set, the main config is at: `"$XDG_CONFIG_HOME"/nvim/init.lua`.
- If `XDG_CONFIG_HOME` is NOT set, the main config is at: `~/.config/nvim/init.lua`.

Extra notes:

- If you set the environment variable `NVIM_APPNAME`, Neovim will use `~/.config/<NVIM_APPNAME>/init.lua`. This lets you keep multiple configs side-by-side.
- Neovim also loads Lua modules from a `lua` folder inside your config directory. Example: `~/.config/nvim/lua/my_module.lua` can be loaded with `require("my_module")` from your `init.lua`.
- Neovim loads plugins using the plugin manager you set up (for example, `lazy.nvim`). The plugin manager itself reads its own configuration files and then loads plugins.

## Minimal Lua config example (from scratch)

This is a very small but complete setup you can copy. It shows:

- How to set options in Lua
- How to set keymaps in Lua
- How to set up the `lazy.nvim` plugin manager

Folder layout to create:

```
~/.config/nvim/init.lua
~/.config/nvim/lua/plugins.lua
```

To create folders on macOS/Linux, you can run this in a terminal:

```
mkdir -p ~/.config/nvim/lua
```

### File: `~/.config/nvim/init.lua`

```lua
-- Set basic options.
-- `vim` is a global object that Neovim provides for Lua.
-- `vim.o` is the table for simple options (like 'number').
-- `true` means the option is turned on. `false` means off.
vim.o.number = true            -- show line numbers
vim.o.relativenumber = true    -- show relative line numbers
vim.o.expandtab = true         -- use spaces instead of tabs
vim.o.shiftwidth = 2           -- number of spaces used for each indent step
vim.o.tabstop = 2              -- number of spaces that a <Tab> counts for

-- Set a keymap (keyboard shortcut).
-- `vim.keymap.set(mode, lhs, rhs, opts?)`
-- - `mode` is a string. "n" means Normal mode.
-- - `lhs` is the keys you press.
-- - `rhs` is what Neovim will do.
-- - `opts` is an optional table for extra flags. Here we use `{ noremap = true, silent = true }`.
vim.keymap.set("n", "<leader>w", ":w<CR>", { noremap = true, silent = true })
-- In the above: `<leader>` is a special prefix key. By default it is `\` in Neovim.
-- `<CR>` means the Enter key (Carriage Return).

-- Bootstrap the lazy.nvim plugin manager if it is not installed yet.
-- This downloads lazy.nvim to a local path and adds it to the runtime path.
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable",
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

-- Load our plugin list from lua/plugins.lua
require("lazy").setup(require("plugins"))
```

Explanation of new terms in the snippet above:

- `vim.fn.stdpath("data")`: calls a Neovim function named `stdpath` from Lua. The argument `"data"` asks for the data directory. Neovim uses it to store downloaded plugins.
- `vim.loop.fs_stat(path)`: checks if a file or folder exists at `path`.
- `vim.fn.system({ ... })`: runs a shell command. Here we run `git clone` with arguments to download `lazy.nvim`.
- `vim.opt.rtp:prepend(lazypath)`: adds the path to the runtime path (a list of folders Neovim searches when loading files and plugins). `rtp` means runtime path.
- `require("lazy")`: loads the Lua module named `lazy` (this is the plugin manager).
- `require("plugins")`: loads our `lua/plugins.lua` file as a Lua module.

### File: `~/.config/nvim/lua/plugins.lua`

```lua
-- This file returns a Lua table (a list) of plugin specs for lazy.nvim.
-- Each item tells lazy.nvim what plugin to install and how to set it up.
return {
  {
    "nvim-treesitter/nvim-treesitter",
    build = ":TSUpdate",   -- run :TSUpdate after install/update
    config = function()
      require("nvim-treesitter.configs").setup({
        ensure_installed = { "lua", "vim", "vimdoc", "bash", "markdown" },
        highlight = { enable = true },
      })
    end,
  },

  {
    "nvim-lualine/lualine.nvim",
    dependencies = { "nvim-tree/nvim-web-devicons" },
    config = function()
      require("lualine").setup({ options = { theme = "auto" } })
    end,
  },
}
```

Explanation of new terms in the snippet above:

- `return { ... }`: in Lua this makes the file a module that returns a list of items.
- Each `{ ... }` inside the list is a plugin specification.
- The first string like `"nvim-treesitter/nvim-treesitter"` is the GitHub `owner/repo` of the plugin.
- `build = ":TSUpdate"`: after install/update, lazy.nvim runs the command `:TSUpdate` in Neovim.
- `config = function() ... end`: a Lua function that runs after the plugin loads. Here we call the plugin’s own setup function.
- `dependencies = { ... }`: tells lazy.nvim to install these plugins first because the main plugin needs them.

With the two files above in place, start Neovim by typing `nvim` in a terminal. The first start will download plugins. Wait until it finishes. Then restart Neovim.

## Using a starter like LazyVim instead

If you want a ready-made setup:

1. Backup or remove your current config folder `~/.config/nvim` if it exists.
2. Follow LazyVim’s quick start (on its GitHub). It usually means cloning a template into `~/.config/nvim`.
3. Start Neovim. LazyVim will install plugins and set many defaults.

Key idea: You can still add your own files in `~/.config/nvim/lua/` to change or extend LazyVim. Starters sit on top of Neovim; they do not replace it.

## Troubleshooting

- Neovim does not see my config: check that your file is at `~/.config/nvim/init.lua`. Check file spelling and folder names.
- A plugin does not load: check that `lazy.nvim` is installed (the `lazy` folder exists under the path shown by `:echo stdpath('data')`). Check for typos in plugin names like `owner/repo`.
- I want a separate config for experiments: set `NVIM_APPNAME=mytest` before starting Neovim. Then Neovim will use `~/.config/mytest/` as the config folder. Example: run `NVIM_APPNAME=mytest nvim` in your terminal.

## Quick reference: going back

- Back within a line: `h`, `b`, `ge`, `0`, `^`, `F{char}`, `T{char}`
- Back by words: `{count}b`, `{count}ge`
- Back by paragraphs: `{count}{`
- Back by lines: `{count}k`
- Back by pages: `Ctrl-u` (half page up), `Ctrl-b` (full page up)


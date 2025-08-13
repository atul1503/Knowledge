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

## Window management and navigation

A "window" shows one buffer (one file view). A "split" is just another window beside or above/below. A "tab" is a layout that can contain multiple windows.

### Move between windows

- Ctrl-w h: move to the window on the left
- Ctrl-w j: move to the window below
- Ctrl-w k: move to the window above
- Ctrl-w l: move to the window on the right
- Ctrl-w w: move to the next window (cycle)
- Ctrl-w p: move to the previous window (go back)

Jump to a specific window by number:

- Show how many windows exist: `:echo winnr('$')`
- Jump to window N: type `:N wincmd w` (example: `:2 wincmd w`)

Optional Lua shortcuts to jump to windows 1..9:

```lua
for i = 1, 9 do
  vim.keymap.set("n", "<leader>" .. i, i .. "<C-w>w", { remap = true, desc = "Go to window " .. i })
end
```

### Create and close splits

- Horizontal split: `:split` or Ctrl-w s
- Vertical split: `:vsplit` or Ctrl-w v
- Close current window: `:q` or Ctrl-w c
- Keep only current window (close others): Ctrl-w o

### Resize and rearrange

- Make all windows equal size: Ctrl-w =
- Grow/shrink height: Ctrl-w + (taller), Ctrl-w - (shorter)
- Grow/shrink width: Ctrl-w > (wider), Ctrl-w < (narrower)
- Move current window to a side: Ctrl-w H (left), Ctrl-w J (bottom), Ctrl-w K (top), Ctrl-w L (right)

### Tabs (optional)

- Next tab: `gt`
- Previous tab: `gT`
- Go to tab N: `{N}gt`

Tip: Tabs are collections of windows. Use them if you want separate layouts for different tasks.

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

## Copy and paste (inside Neovim and with the system clipboard)

Here the word "yank" means copy. A "register" is a small storage box inside Neovim that holds text. The default register holds your last yank or delete. The system clipboard is a special register named `+`.

### Copy (yank) inside Neovim

- Copy the current line: `yy`
- Copy N lines: `{N}yy` (example: `5yy`)
- Copy by motion: `y{motion}` (example: `y$` copies to end of line; `yaw` copies the word under the cursor)
- Visual select then copy: press `v`, move to select, then `y`

### Paste inside Neovim

- Paste after cursor: `p`
- Paste before cursor: `P`
- Paste over a selection: select with `v`, then press `p` (replaces the selection)

### Use the system clipboard

- Copy to system clipboard: `"+y` (example: select text with `v` then press `"+y`)
- Paste from system clipboard: `"+p`

On Linux there is also a selection clipboard named `*` (middle-click paste in X11): use `"*y` and `"*p`.

Make system clipboard the default so normal `y`/`p` use it:

- Temporary (command): `:set clipboard=unnamedplus`
- Permanent (Lua in `init.lua`):
```lua
vim.opt.clipboard = "unnamedplus"
```

### Paste text from your OS into Neovim

- Easiest: go to Insert mode (`i`) and paste from your terminal (for example Cmd+V on macOS, Ctrl+Shift+V on many terminals). Modern terminals send "bracketed paste" and Neovim handles it safely.
- Precise: stay in Normal mode and use `"+p` to paste from the system clipboard register.

### Copy text from Neovim to other apps

- Visual select the text, then `"+y`. Now you can paste into other apps with your OS paste (Cmd+V or Ctrl+V).
- Optional shortcut (Lua): map `<leader>y` to copy selection to clipboard.
```lua
vim.keymap.set("v", "<leader>y", '"+y', { desc = "Yank to system clipboard" })
```

### Troubleshooting clipboard

- Check support: run `:checkhealth` and read the Clipboard section.
- macOS: Neovim uses `pbcopy`/`pbpaste` automatically. If `"+` fails, ensure Neovim was installed with clipboard support (Homebrew `neovim` is fine).
- Linux (X11): install `xclip` or `xsel` so Neovim can talk to the clipboard. For Wayland, install `wl-clipboard` (`wl-copy`, `wl-paste`).
- Inside tmux: enable clipboard pass-through. In `~/.tmux.conf` add:
```
set -g set-clipboard on
```
Then restart tmux. Now Neovim’s `"+y` should reach the system clipboard.

## Search

- `/text` then `Enter`: search forward for `text`.
- `?text` then `Enter`: search backward for `text`. This is also a way to go back to a previous match.
- `n`: move to the next match.
- `N`: move to the previous match (goes the opposite direction of the last search).

## Search across files (project-wide)

This means: look for text in many files inside a folder and its sub-folders, and jump to the results.

### Using built-in `:vimgrep`

- Basic: type `:vimgrep /text/ **/*` then press `Enter`.
  - `:vimgrep` searches using Vim’s regular expressions.
  - `/text/` is the pattern. The slashes `/` are separators. Replace `text` with what you want to find.
  - `**/*` says: search in all files under the current directory (recursively). It uses shell-style globs.
- Show the result list (called the quickfix list): `:copen`
- Jump through results: `:cnext` (next), `:cprevious` (previous)
- Close the list: `:cclose`

Useful patterns:

- Case-insensitive match: add `\c` inside the pattern. Example: `:vimgrep /todo\c/ **/*`
- Match a whole word: use `\<` and `\>` around the word. Example: `:vimgrep /\<init\>/ **/*`
- Add more results to the same list: use `:vimgrepadd` instead of `:vimgrep`.

### With Telescope (if installed, common in LazyVim)

- Live search by typing: `:Telescope live_grep`
- Search for the word under cursor: `:Telescope grep_string`

These commands open a picker window where you can type the search term. Select a result to jump there.

### Terminal-only fallback (outside Neovim)

Run these in your terminal if you prefer, then open files in Neovim:

- Using ripgrep:
```sh
rg -n "text" .
```
- Using standard grep (slower):
```sh
grep -RIn "text" . 2>/dev/null
```

In both, `-n` or `-nI` prints line numbers. `-R` searches recursively. `-I` skips binary files.

### File glob patterns for `:vimgrep`

`:vimgrep /PATTERN/ {file-globs...}` searches with a Vim regex `PATTERN` across files that match the globs. The part after the second `/` uses file name patterns (globs).

- `*`: any characters except `/` (stays in one path segment)
- `**`: any characters, can cross `/` (any depth of subdirectories)
- `?`: exactly one character (not `/`)
- `[abc]`: one of `a`, `b`, or `c`; `[a-z]`: any lowercase letter; `[^a]`: any char except `a`
- `{alt1,alt2}`: alternation for file name parts or extensions

Common examples:

- Whole project: `:vimgrep /text/ **/*`
- One level deep only: `:vimgrep /text/ */*`
- Only Java: `:vimgrep /\<MyClass\>/ **/*.java`
- Java or Kotlin: `:vimgrep /pattern/ **/*.{java,kt}`
- Only under `src/`: `:vimgrep /pattern/ src/**/*`
- Test files: `:vimgrep /pattern/ **/Test*.*`
- Hidden files (dotfiles): `:vimgrep /pattern/ **/.*`
- Uppercase names: `:vimgrep /pattern/ **/[A-Z]*.java`

Notes:

- Use spaces to pass multiple globs: `:vimgrep /pat/ src/**/* test/**/*`
- Quote paths with spaces: `:vimgrep /pat/ "docs/My Notes"/**/*`
- Excluding directories is best done by narrowing positives (for example target `src/**` instead of `**/*`).
- Add to the same quickfix list with `:vimgrepadd`.

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

## Common Neovim Lua API use cases (simple recipes)

The Lua API lets you change Neovim behavior using Lua code. In Lua, `vim` is a global table that Neovim provides. It contains:

- `vim.o` and `vim.opt`: set options.
- `vim.g`: set global variables (used by some plugins).
- `vim.keymap.set`: set keymaps.
- `vim.api`: low-level functions (create autocommands, buffers, windows, etc.).
- `vim.cmd`: run a Vim command as a string.
- `vim.diagnostic`: work with diagnostics (errors, warnings).
- `vim.lsp`: talk to the Language Server Protocol features when an LSP is attached.
- `vim.fs`: helper functions to find files and walk directories.
- `vim.notify`: show messages.

Each recipe below explains new terms when they appear.

### Set options and globals

```lua
-- Options (editor settings)
vim.o.number = true            -- turn on absolute line numbers
vim.opt.relativenumber = true  -- turn on relative line numbers
vim.opt.shiftwidth = 2         -- spaces per indent step
vim.opt.tabstop = 2            -- how many spaces a <Tab> counts for

-- Global variables (seen by plugins)
vim.g.mapleader = " "         -- set <leader> to Space
```

Explanation:

- `vim.o` and `vim.opt` both change options. `vim.opt` accepts richer types (like lists); both are fine for simple values.
- `vim.g.mapleader` sets the leader key. `<leader>` is a prefix used in keymaps.

### Keymaps (keyboard shortcuts)

```lua
-- Map <leader>e to open the file explorer (example using :Ex)
vim.keymap.set("n", "<leader>e", ":Ex<CR>", { desc = "Open explorer", silent = true })

-- Buffer-local mapping: only for the current buffer (use 0 for current buffer id)
vim.keymap.set("n", "<leader>c", function()
  print("Hello from this buffer only")
end, { buffer = 0, desc = "Buffer hello" })
```

Explanation:

- `vim.keymap.set(mode, lhs, rhs, opts?)` sets a keymap.
- `mode` "n" means Normal mode. `lhs` is what you press. `rhs` is what runs.
- `desc` adds a human-friendly description (used by some plugins and help).
- `silent = true` hides the command from the command line while it runs.
- `buffer = 0` makes the map only for the current buffer (the current open file).

### Autocommands (run code on events)

```lua
-- Create an augroup to keep related autocommands together
local group = vim.api.nvim_create_augroup("ExampleGroup", { clear = true })

-- Auto-trim trailing spaces on save for Lua files
vim.api.nvim_create_autocmd("BufWritePre", {
  group = group,
  pattern = "*.lua",
  callback = function()
    local save = vim.fn.winsaveview()  -- remember cursor view
    vim.cmd([[keeppatterns %s/\s\+$//e]]) -- remove trailing spaces
    vim.fn.winrestview(save)            -- restore cursor view
  end,
})
```

Explanation:

- An "autocommand" runs when an event happens. `BufWritePre` fires before a buffer is written to disk.
- `nvim_create_augroup(name, { clear = true })` defines a group so you can manage/remove its autocommands as a set.
- `pattern` matches file names.
- `callback` is the Lua function to run.

### User commands (custom :commands)

```lua
-- Define :Hello that prints a message
vim.api.nvim_create_user_command("Hello", function(opts)
  -- opts.args contains any arguments typed after :Hello
  vim.notify("Hello " .. (opts.args ~= "" and opts.args or "Neovim"))
end, { nargs = "?", desc = "Say hello" })

-- Usage: :Hello or :Hello YourName
```

Explanation:

- `nvim_create_user_command(name, fn, opts)` creates a `:` command.
- `nargs = "?"` means 0 or 1 argument is allowed.
- `vim.notify(text)` shows a message.

### Work with buffers, windows, and tabs

```lua
-- Write a line at the end of the current buffer
local buf = vim.api.nvim_get_current_buf()      -- current buffer id (a number)
local last = vim.api.nvim_buf_line_count(buf)   -- how many lines are in this buffer
vim.api.nvim_buf_set_lines(buf, last, last, false, { "-- Added by Lua" })

-- Move cursor to line 1, column 1 in the current window
local win = vim.api.nvim_get_current_win()
vim.api.nvim_win_set_cursor(win, { 1, 0 })

-- Open a vertical split with a new scratch buffer
vim.cmd("vsplit")
local newbuf = vim.api.nvim_create_buf(true, true) -- listed=true, scratch=true
vim.api.nvim_win_set_buf(0, newbuf)                -- 0 = current window
```

Explanation:

- A "buffer" is the in-memory text of a file (or scratch text). A "window" shows a buffer. A "tabpage" is a layout of windows.
- `nvim_buf_set_lines(buf, start, end_, strict, lines)` replaces lines from `start` to `end_` with the given list `lines`. Here we append at the end by using `start == end == last`.
- `nvim_win_set_cursor(win, { line, col })` places the cursor. `col` is 0-based.
- `nvim_create_buf(listed, scratch)` creates a new empty buffer. A scratch buffer is not tied to a file.

### Diagnostics (errors and warnings)

```lua
-- Show diagnostics for the current line in a floating window
vim.keymap.set("n", "<leader>d", function()
  vim.diagnostic.open_float(nil, { focus = false, border = "rounded" })
end, { desc = "Line diagnostics" })

-- Navigate diagnostics
vim.keymap.set("n", "[d", vim.diagnostic.goto_prev, { desc = "Prev diagnostic" })
vim.keymap.set("n", "]d", vim.diagnostic.goto_next, { desc = "Next diagnostic" })
```

Explanation:

- `vim.diagnostic.open_float(bufnr?, opts?)` shows messages for a spot.
- `goto_prev` and `goto_next` jump between diagnostics.

### LSP (Language Server Protocol) helpers

These work when an LSP server is attached to the current buffer.

```lua
-- Simple LSP keymaps (when LSP is available)
vim.keymap.set("n", "K", vim.lsp.buf.hover, { desc = "LSP Hover" })
vim.keymap.set("n", "gd", vim.lsp.buf.definition, { desc = "Go to definition" })
vim.keymap.set("n", "gr", vim.lsp.buf.references, { desc = "List references" })
vim.keymap.set("n", "<leader>rn", vim.lsp.buf.rename, { desc = "Rename symbol" })
vim.keymap.set("n", "<leader>ca", vim.lsp.buf.code_action, { desc = "Code action" })
vim.keymap.set({ "n", "v" }, "<leader>lf", function()
  vim.lsp.buf.format({ async = true })
end, { desc = "Format" })
```

Explanation:

- `vim.lsp.buf` has common editor actions powered by the language server.
- `format({ async = true })` runs formatting without blocking the editor.

### Filesystem helpers

```lua
-- Find the nearest git root or `.git` directory upward from the current file
local git_dir = vim.fs.find(".git", { upward = true })[1]
if git_dir then
  vim.notify("Found git folder at: " .. git_dir)
end

-- Find files named README.md under the current working directory (one level deep)
local readmes = vim.fs.find(function(name)
  return name:lower() == "readme.md"
end, { limit = 10 })
```

Explanation:

- `vim.fs.find(target, opts)` searches for names starting from a directory. `upward = true` walks up parent folders.
- You can pass a function to decide matches.

### Notifications and UI prompts

```lua
-- Notify with levels: :help vim.log.levels
vim.notify("All good!", vim.log.levels.INFO)

-- Quick prompt selection (simple menu)
vim.ui.select({ "alpha", "beta", "gamma" }, { prompt = "Choose one:" }, function(choice)
  if choice then vim.notify("You picked: " .. choice) end
end)
```

Explanation:

- `vim.ui.select(items, opts, on_choice)` lets you show a simple list prompt. Plugins can override how it looks.

### Simple async helpers

```lua
-- Defer a function by 200 milliseconds
vim.defer_fn(function()
  vim.notify("This ran later")
end, 200)

-- Schedule a function to run in the main event loop (useful inside callbacks)
vim.schedule(function()
  print("Scheduled work")
end)
```

Explanation:

- `vim.defer_fn(fn, ms)` runs `fn` after `ms` milliseconds.
- `vim.schedule(fn)` runs `fn` soon on the main thread. Use it when you must touch the UI from a different callback.

## Quick reference: going back

- Back within a line: `h`, `b`, `ge`, `0`, `^`, `F{char}`, `T{char}`
- Back by words: `{count}b`, `{count}ge`
- Back by paragraphs: `{count}{`
- Back by lines: `{count}k`
- Back by pages: `Ctrl-u` (half page up), `Ctrl-b` (full page up)


```
                                 __
                                / /________  _________ _____ _____ _
                               / / ___/ __ \/ ___/ __ `/ __ `/ __ `/
                              / (__  ) /_/ (__  ) /_/ / /_/ / /_/ /
                             /_/____/ .___/____/\__,_/\__, /\__,_/
                                   /_/               /____/

                          ⚡ designed for convenience and efficiency ⚡
```

A light-weight lsp plugin based on neovim's built-in lsp with a highly performant UI.

[![](https://img.shields.io/badge/Element-0DBD8B?style=for-the-badge&logo=element&logoColor=white)](https://matrix.to/#/#lspsaga-nvim:matrix.org)

1. [Install](#install)
   - [Vim Plug](#vim-plug)
   - [Packer](#packer)
2. [Configuration](#configuration)
3. [Mappings](#mappings)
4. [Customize Appearance](#customize-appearance)
5. [Showcase](#showcase)
6. [Donate](#donate)
7. [License](#license)

## Install

### Vim Plug

```vim
Plug 'neovim/nvim-lspconfig'
Plug 'glepnir/lspsaga.nvim', { 'branch': 'main' }
```

### Packer

```lua
use({
    "glepnir/lspsaga.nvim",
    branch = "main",
    config = function()
        local saga = require("lspsaga")

        saga.init_lsp_saga({
            -- your configuration
        })
    end,
})
```

## Configuration

Lspsaga support use command `Lspsaga` with completion or use lua function

```lua
local saga = require 'lspsaga'

-- change the lsp symbol kind
local kind = require('lspsaga.lspkind')
kind[type_number][2] = icon -- see lua/lspsaga/lspkind.lua

-- use default config
saga.init_lsp_saga()

-- use custom config
saga.init_lsp_saga({
    -- put modified options in there
})

-- Options with default value
-- "single" | "double" | "rounded" | "bold" | "plus"
border_style = "single",
--the range of 0 for fully opaque window (disabled) to 100 for fully
--transparent background. Values between 0-30 are typically most useful.
saga_winblend = 0,
-- when cursor in saga window you config these to move
move_in_saga = { prev = '<C-p>',next = '<C-n>'},
-- Error, Warn, Info, Hint
-- use emoji like
-- { "🙀", "😿", "😾", "😺" }
-- or
-- { "😡", "😥", "😤", "😐" }
-- and diagnostic_header can be a function type
-- must return a string and when diagnostic_header
-- is function type it will have a param `entry`
-- entry is a table type has these filed
-- { bufnr, code, col, end_col, end_lnum, lnum, message, severity, source }
diagnostic_header = { " ", " ", " ", "ﴞ " },
-- show diagnostic source
show_diagnostic_source = true,
-- add bracket or something with diagnostic source, just have 2 elements
diagnostic_source_bracket = {},
-- preview lines of lsp_finder and definition preview
max_preview_lines = 10,
-- use emoji lightbulb in default
code_action_icon = "💡",
-- if true can press number to execute the codeaction in codeaction window
code_action_num_shortcut = true,
-- same as nvim-lightbulb but async
code_action_lightbulb = {
    enable = true,
    sign = true,
    enable_in_insert = true,
    sign_priority = 20,
    virtual_text = true,
},
-- finder icons
finder_icons = {
  def = '  ',
  ref = '諭 ',
  link = '  ',
},
-- finder do lsp request timeout
-- if your project big enough or your server very slow
-- you may need to increase this value
finder_request_timeout = 1500,
finder_action_keys = {
    open = "o",
    vsplit = "s",
    split = "i",
    tabe = "t",
    quit = "q",
    scroll_down = "<C-f>",
    scroll_up = "<C-b>", -- quit can be a table
},
code_action_keys = {
    quit = "q",
    exec = "<CR>",
},
rename_action_quit = "<C-c>",
rename_in_select = true,
definition_preview_icon = "  ",
-- show symbols in winbar must nightly
symbol_in_winbar = {
    in_custom = false,
    enable = false,
    separator = ' ',
    show_file = true,
    click_support = false,
},
-- show outline
show_outline = {
  win_position = 'right',
  --set special filetype win that outline window split.like NvimTree neotree
  -- defx, db_ui
  win_with = '',
  win_width = 30,
  auto_enter = true,
  auto_preview = true,
  virt_text = '┃',
  jump_key = 'o',
  -- auto refresh when change buffer
  auto_refresh = true,
},
-- if you don't use nvim-lspconfig you must pass your server name and
-- the related filetypes into this table
-- like server_filetype_map = { metals = { "sbt", "scala" } }
server_filetype_map = {},
```

## Show symbols in winbar need neovim 0.8+

<details>
<summary> work with custom winbar </summary>
  
```lua
saga.init_lsp_saga({
    symbol_in_winbar = {
        in_custom = true
    }
})
```

- use `require('lspsaga.symbolwinbar').get_symbol_node` this function in your custom winbar
to get symbols node and set `User LspsagaUpdateSymbol` event in your autocmds

```lua
-- Example:
local function get_file_name(include_path)
    local file_name = require('lspsaga.symbolwinbar').get_file_name()
    if vim.fn.bufname '%' == '' then return '' end
    if include_path == false then return file_name end
    -- Else if include path: ./lsp/saga.lua -> lsp > saga.lua
    local sep = vim.loop.os_uname().sysname == 'Windows' and '\\' or '/'
    local path_list = vim.split(string.gsub(vim.fn.expand '%:~:.:h', '%%', ''), sep)
    local file_path = ''
    for _, cur in ipairs(path_list) do
        file_path = (cur == '.' or cur == '~') and '' or
                    file_path .. cur .. ' ' .. '%#LspSagaWinbarSep#>%*' .. ' %*'
    end
    return file_path .. file_name
end

local function config_winbar()
    local exclude = {
        ['teminal'] = true,
        ['toggleterm'] = true,
        ['prompt'] = true,
        ['NvimTree'] = true,
        ['help'] = true,
    } -- Ignore float windows and exclude filetype
    if vim.api.nvim_win_get_config(0).zindex or exclude[vim.bo.filetype] then
        vim.wo.winbar = ''
    else
        local ok, lspsaga = pcall(require, 'lspsaga.symbolwinbar')
        local sym
        if ok then sym = lspsaga.get_symbol_node() end
        local win_val = ''
        win_val = get_file_name(true) -- set to true to include path
        if sym ~= nil then win_val = win_val .. sym end
        vim.wo.winbar = win_val
    end
end

local events = { 'BufEnter', 'BufWinEnter', 'CursorMoved' }

vim.api.nvim_create_autocmd(events, {
    pattern = '*',
    callback = function() config_winbar() end,
})

vim.api.nvim_create_autocmd('User', {
    pattern = 'LspsagaUpdateSymbol',
    callback = function() config_winbar() end,
})
```

</details>

<details>

<summary>Support Click in symbols winbar</summary>

To enable click support for winbar define a function similar to [statusline](https://neovim.io/doc/user/options.html#'statusline') (Search for "Start of execute function label")

minwid will be replaced with current node. For example:

```lua
symbol_in_winbar = {
    click_support = function(node, clicks, button, modifiers)
        -- To see all avaiable details: vim.pretty_print(node)
        local st = node.range.start
        local en = node.range['end']
        if button == "l" then
            if clicks == 2 then
                -- double left click to do nothing
            else -- jump to node's starting line+char
                vim.fn.cursor(st.line + 1, st.character + 1)
            end
        elseif button == "r" then
            if modifiers == "s" then
                print "lspsaga" -- shift right click to print "lspsaga"
            end -- jump to node's ending line+char
            vim.fn.cursor(en.line + 1, en.character + 1)
        elseif button == "m" then
            -- middle click to visual select node
            vim.fn.cursor(st.line + 1, st.character + 1)
            vim.cmd "normal v"
            vim.fn.cursor(en.line + 1, en.character + 1)
        end
    end
}
```
</details>

## Mappings

Plugin does not provide mappings by default. However, you can bind mappings yourself. You can find examples in the [showcase](#showcase) section.

## Customize Appearance

Colors can be simply changed by overwriting the default highlights groups LspSaga is using.

```vim
highlight link LspSagaFinderSelection Search
" or
highlight link LspSagaFinderSelection guifg='#ff0000' guibg='#00ff00' gui='bold'
```

The available highlight groups you can find in [here](./plugin/lspsaga.lua).

## Showcase

<details>
<summary>lsp finder</summary>

Finder Title work with neovim 0.8 +

**Lua**

```lua
-- or use command LspSagaFinder
vim.keymap.set("n", "gh", "<cmd>Lspsaga lsp_finder<CR>", { silent = true })
```

NOTE: This requires ```filetypes``` and ```root_dir``` set in the  LSP server ```config``` object. So for [nvim-jdtls](https://github.com/mfussenegger/nvim-jdtls) users, even if you are loading the plugin as ```ftplugin``` or with ```FileType java``` autocmd, set ```filetypes``` in the ```config``` object.
  
<div align='center'>
<img
src="https://user-images.githubusercontent.com/41671631/181253960-cef49f9d-db8b-4b04-92d8-cb6322749414.png" />
</div>

</details>

<details>
<summary>Code action</summary>

**Lua**

```lua
local action = require("lspsaga.codeaction")

-- code action
vim.keymap.set("n", "<leader>ca", action.code_action, { silent = true })
vim.keymap.set("v", "<leader>ca", function()
    vim.fn.feedkeys(vim.api.nvim_replace_termcodes("<C-U>", true, false, true))
    action.range_code_action()
end, { silent = true })
-- or use command
vim.keymap.set("n", "<leader>ca", "<cmd>Lspsaga code_action<CR>", { silent = true })
vim.keymap.set("v", "<leader>ca", "<cmd><C-U>Lspsaga range_code_action<CR>", { silent = true })
```

<div align='center'>
<img
src="https://user-images.githubusercontent.com/41671631/175305503-180e6b39-d162-4ef2-aa2b-9ffe309948e6.gif"/>
</div>

</details>

<details>
<summary>Async lightbulb</summary>

<div align='center'>
<img
src="https://user-images.githubusercontent.com/41671631/175752848-cef8218a-f8e4-42c2-96bd-06bb07cd42c6.gif"/>
</div>

</details>

<details id="hover-doc">
<summary>Hover doc</summary>

**Lua**

```lua
-- or use command
vim.keymap.set("n", "K", "<cmd>Lspsaga hover_doc<CR>", { silent = true })

local action = require("lspsaga.action")
-- scroll down hover doc or scroll in definition preview
vim.keymap.set("n", "<C-f>", function()
    action.smart_scroll_with_saga(1)
end, { silent = true })
-- scroll up hover doc
vim.keymap.set("n", "<C-b>", function()
    action.smart_scroll_with_saga(-1)
end, { silent = true })
```

<div align='center'>
<img
src="https://user-images.githubusercontent.com/41671631/175306592-f0540e35-561f-418c-a41e-7df167ba9b86.gif"/>
</div>

</details>

<details>
<summary>Signature help</summary>

You also can use `smart_scroll_with_saga` (see [hover doc](#hover-doc)) to scroll in signature help win.

**Lua**

```lua
-- or command
vim.keymap.set("n", "gs", "<Cmd>Lspsaga signature_help<CR>", { silent = true })
```

<div align='center'>
<img
src="https://user-images.githubusercontent.com/41671631/175306809-755c4624-a5d2-4c11-8b29-f41914f22411.gif"/>
</div>

</details>

<details>
<summary>Rename with preview and select</summary>

**Lua**

```lua
-- or command
vim.keymap.set("n", "gr", "<cmd>Lspsaga rename<CR>", { silent = true })
-- close rename win use <C-c> in insert mode or `q` in normal mode or `:q`
```

<div align="center">
<img
src="https://user-images.githubusercontent.com/41671631/175300080-6e72001c-78dd-4d86-8139-bba38befee15.gif" />
</div>

</details>

<details>
<summary>Preview definition</summary>

You also can use `smart_scroll_with_saga` (see [hover doc](#hover-doc)) to scroll in preview definition win.

**Lua**

```lua
-- or use command
vim.keymap.set("n", "gd", "<cmd>Lspsaga preview_definition<CR>", { silent = true })
```

<div align='center'>
<img
src="https://user-images.githubusercontent.com/41671631/105657900-5b387f00-5f00-11eb-8b39-4d3b1433cb75.gif"/>
</div>

</details>

<details>
<summary>Jump and show diagnostics</summary>

**Lua**

```lua
vim.keymap.set("n", "<leader>cd", "<cmd>Lspsaga show_line_diagnostics<CR>", { silent = true })
vim.keymap.set("n", "<leader>cd", "<cmd>Lspsaga show_cursor_diagnostics<CR>", { silent = true })

-- or jump to error
vim.keymap.set("n", "[E", function()
  require("lspsaga.diagnostic").goto_prev({ severity = vim.diagnostic.severity.ERROR })
end, { silent = true })
vim.keymap.set("n", "]E", function()
  require("lspsaga.diagnostic").goto_next({ severity = vim.diagnostic.severity.ERROR })
end, { silent = true })
-- or use command
vim.keymap.set("n", "[e", "<cmd>Lspsaga diagnostic_jump_next<CR>", { silent = true })
vim.keymap.set("n", "]e", "<cmd>Lspsaga diagnostic_jump_prev<CR>", { silent = true })
```

<div align='center'>
<img
src="https://user-images.githubusercontent.com/41671631/182015252-c2e8acc1-3833-473d-a375-8093e104dc47.gif"/>
</div>

</details>

<details>
<summary>Fastest show symbols in winbar by use cache </summary>

<div align="center">
<img
src="https://user-images.githubusercontent.com/41671631/176679585-9485676b-ddea-44ca-bc88-b0eb04d450b1.gif" />
</div>

</details>

<details>
<summary>Outline</summary>

work fast when lspsaga symbol winbar `in_custom = true` or `enable = true`,

**Lua**

```
:LSoutlineToggle
```

<img
src="https://user-images.githubusercontent.com/41671631/179864315-3ec84106-bcd4-43db-8590-2fb07f4055d9.gif"/>
</div>

</details>

<details>
<summary>Float terminal</summary>

**Lua**

```lua
-- float terminal also you can pass the cli command in open_float_terminal function
local term = require("lspsaga.floaterm")

-- float terminal also you can pass the cli command in open_float_terminal function
vim.keymap.set("n", "<A-d>", function()
    term.open_float_terminal("custom_cli_command")
end, { silent = true })
vim.keymap.set("t", "<A-d>", function()
    vim.fn.feedkeys(vim.api.nvim_replace_termcodes("<C-\\><C-n>", true, false, true))
    term.close_float_terminal()
end, { silent = true })

-- or use command
vim.keymap.set("n", "<A-d>", "<cmd>Lspsaga open_floaterm custom_cli_command<CR>", { silent = true })
vim.keymap.set("t", "<A-d>", "<C-\\><C-n><cmd>Lspsaga close_floaterm<CR>", { silent = true })
```

<div align='center'>
<img
src="https://user-images.githubusercontent.com/41671631/105658287-2c6ed880-5f01-11eb-8af6-daa6fd23576c.gif"/>
</div>

</details>

## Donate

[![](https://img.shields.io/badge/PayPal-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/bobbyhub)

If you'd like to support my work financially, buy me a drink through [paypal](https://paypal.me/bobbyhub).

# License

Licensed under the [MIT](./LICENSE) license.

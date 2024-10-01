---
title: Create Neovim Plugins
tags:
  - neovim
---
## Simple Neovim Function

```lua
function Todo() print("Hello, Neovim!") end
```

- Source the file using `%so`
- Run the command `:lua Todo()`

## Trigger
- `vim.api.nvim_create_user_command("Todo", Todo, {})` - Source the file and run `:Todo`
- `vim.api.nvim_create_auto_command("CursorHold", { callback = Todo })`  - Source the file and hold the cursor and it will call the function
- `vim.keymap.set("n", "<leader>u", Todo` - Source the file and press `space + u` on keyboard to trigger the function.

## Creating Plugin

1. make an empty lua directory (`mkdir lua`)
2. Add a file with your plugin name (`touch lua/<plugin-name>.lua`)
3. while opening the neovim, set the runtimepath `nvim --cmd "set rtp+=.`
4. require the plugin name (`require('NeovimConf').todo()`) 
5. [[#Important Note]]
## Module
```lua
-- lua/NeovimConf.lua
local M = {}
M.todo = function() print("Hello, Neovim!") end
return M
```

`nvim --cmd "set rtp+=.`

`require('NeovimConf').todo()`

## Important Note

> require caches the result. So, if we change the contents, it does not get reflected. So, we need to write a small script using `nvim_create_user_command` as below

```lua
vim.api.nvim_create_user_command("Test", function() 
 		package.loaded.<plugin-name> = nil
 		require("<plugin-name>").todo()
 		end, {}
```

we can type the command :Test, so that we make the cached result `nil`  and require again.




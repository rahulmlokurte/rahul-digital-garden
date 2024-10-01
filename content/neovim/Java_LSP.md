---
title: Java Lsp
tags:
  - neovim
---
# Setting Up Java LSP and Debugger in Neovim with Lazy.nvim

In this guide, we will cover the step-by-step process to set up Java Language Server Protocol (LSP) and debugging capabilities in Neovim using the `lazy.nvim` plugin manager. We'll use the `nvim-jdtls` plugin for LSP functionalities and `nvim-dap` for debugging support, ensuring an efficient Java development environment within Neovim.

## Prerequisites

1. **Neovim**: Ensure that you have Neovim installed (version 0.8 or later).
2. **Java Development Kit (JDK)**: Install a compatible version of the JDK.
3. **Maven/Gradle**: Preferred build tool for Java projects.
4. **`mason.nvim`**: For managing LSP servers, DAPs, and other tooling.
5. **`lazy.nvim`**: For plugin management.

If not already installed, you can install `lazy.nvim` as follows:

```git clone https://github.com/folke/lazy.nvim.git ~/.config/nvim/lazy.nvim```

## Step 1: Plugin Configuration with `lazy.nvim`
### Initial Plugin Setup
In your Neovim configuration, create a `lua/plugins.lua` file (if it doesn’t already exist) and add the following content for the Java LSP and Debugger setup:

```lua
return {
  -- Java Language Server
  'mfussenegger/nvim-jdtls',
  -- Debug Adapter Protocol (DAP)
  'mfussenegger/nvim-dap',
  dependencies = {
    'rcarriga/nvim-dap-ui',
    'williamboman/mason.nvim',
    'jay-babu/mason-nvim-dap.nvim',
    'leoluz/nvim-dap-go', -- Go debugger
    { 'mfussenegger/nvim-dap-python', ft = 'python' }, -- Python debugger
  },
}
```

### DAP Configuration
Create a file named `debug.lua` and add the following content:

```lua
return {
  'mfussenegger/nvim-dap',
  dependencies = {
    'rcarriga/nvim-dap-ui',
    'nvim-neotest/nvim-nio',
    'williamboman/mason.nvim',
    'jay-babu/mason-nvim-dap.nvim',
    'leoluz/nvim-dap-go',
    { 'mfussenegger/nvim-dap-python', ft = 'python' },
  },
  keys = function(_, keys)
    local dap = require 'dap'
    local dapui = require 'dapui'
    return {
      { '<F5>', dap.continue, desc = 'Debug: Start/Continue' },
      { '<F1>', dap.step_into, desc = 'Debug: Step Into' },
      { '<F2>', dap.step_over, desc = 'Debug: Step Over' },
      { '<F3>', dap.step_out, desc = 'Debug: Step Out' },
      { '<leader>b', dap.toggle_breakpoint, desc = 'Debug: Toggle Breakpoint' },
      { '<leader>B', function() dap.set_breakpoint(vim.fn.input 'Breakpoint condition: ') end, desc = 'Debug: Set Breakpoint' },
      { '<F7>', dapui.toggle, desc = 'Debug: See last session result.' },
      unpack(keys),
    }
  end,
  config = function()
    local dap = require 'dap'
    local dapui = require 'dapui'
    dap.configurations.java = {
      { type = 'java', name = 'Integration', request = 'launch', main = '', console = 'internalConsole', args = '${command:SpecifyProgramArgs}', vmArgs = '-Dspring.profiles.active=integration' },
      { type = 'java', name = 'local', request = 'launch', main = '', console = 'internalConsole', args = '${command:SpecifyProgramArgs}', vmArgs = '-Dspring.profiles.active=local' },
      { type = 'java', name = 'test', request = 'launch', main = '', console = 'internalConsole', args = '${command:SpecifyProgramArgs}', vmArgs = '-Dspring.profiles.active=test' },
    }

    require('mason-nvim-dap').setup {
      automatic_installation = true,
      ensure_installed = { 'delve', 'debugpy' },
    }

    dapui.setup { icons = { expanded = '▾', collapsed = '▸', current_frame = '*' } }
    dap.listeners.after.event_initialized['dapui_config'] = dapui.open
    dap.listeners.before.event_terminated['dapui_config'] = dapui.close
    dap.listeners.before.event_exited['dapui_config'] = dapui.close
    require('dap-go').setup { delve = { detached = vim.fn.has 'win32' == 0 } }
    require('dap-python').setup '~/.pyenv/versions/3.12.1/bin/python'
  end,
}
```

### JDTLS Configuration

Create a `java-lsp.lua` file to handle the LSP setup for Java:

```lua
local jdtls = require 'jdtls'
local project_name = vim.fn.fnamemodify(vim.fn.getcwd(), ':p:h:t')
local workspace_dir = '/home/rahul/neo-java/' .. project_name

local lsp_capabilities = require('cmp_nvim_lsp').default_capabilities()
local bundles = {
  vim.fn.glob('/home/rahul/.local/share/nvim/mason/packages/java-debug-adapter/extension/server/com.microsoft.java.debug.plugin-0.53.0.jar', 1),
}

vim.list_extend(bundles, vim.split(vim.fn.glob('/home/rahul/.local/share/nvim/mason/packages/java-test/extension/server/*.jar', 1), '\n'))

local config = {
  capabilities = lsp_capabilities,
  cmd = {
    '/home/rahul/.sdkman/candidates/java/current/bin/java',
    '-Declipse.application=org.eclipse.jdt.ls.core.id1',
    '-Dosgi.bundles.defaultStartLevel=4',
    '-Declipse.product=org.eclipse.jdt.ls.core.product',
    '-Dlog.protocol=true',
    '-Dlog.level=ALL',
    '-javaagent:/home/rahul/software/jdtls/lombok.jar',
    '-Xmx1g',
    '--add-modules=ALL-SYSTEM',
    '--add-opens',
    'java.base/java.util=ALL-UNNAMED',
    '--add-opens',
    'java.base/java.lang=ALL-UNNAMED',
    '-jar',
    '/home/rahul/software/jdtls/plugins/org.eclipse.equinox.launcher_1.6.900.v20240613-2009.jar',
    '-configuration',
    '/home/rahul/software/jdtls/config_linux/',
    '-data',
    workspace_dir,
  },
  root_dir = require('jdtls.setup').find_root { '.git', 'mvnw', 'gradlew', 'pom.xml', 'build.gradle' },
  settings = {
    java = {
      signatureHelp = { enabled = true },
      contentProvider = { preferred = 'fernflower' },
      completion = {
        favoriteStaticMembers = { 'org.hamcrest.MatcherAssert.assertThat', 'org.hamcrest.Matchers.*', 'org.junit.jupiter.api.Assertions.*', 'java.util.Objects.requireNonNull' },
        filteredTypes = { 'com.sun.*', 'io.micrometer.shaded.*', 'java.awt.*', 'jdk.*', 'sun.*' },
      },
      sources = { organizeImports = { starThreshold = 9999, staticStarThreshold = 9999 } },
      codeGeneration = { toString = { template = '${object.className}{${member.name()}=${member.value}, ${otherMembers}}' }, hashCodeEquals = { useJava7Objects = true }, useBlocks = true },
    },
  },
  init_options = { extendedClientCapabilities = jdtls.extendedClientCapabilities, bundles = bundles },
}

jdtls.start_or_attach(config)
```

## Step 2: Debugging Java Code
With the DAP setup in place, use the following keybindings to start debugging:

- **`<F5>`**: Start/Continue Debugging
- **`<F1>`**: Step Into
- **`<F2>`**: Step Over
- **`<F3>`**: Step Out
- **`<leader>b`**: Toggle Breakpoint
- **`<leader>B`**: Set Conditional Breakpoint

### Conclusion
This guide shows how to set up a comprehensive Java development environment in Neovim using `lazy.nvim` for plugin management. The setup includes LSP configuration through `nvim-jdtls` and debugging capabilities using `nvim-dap`. With this setup, you can seamlessly develop and debug Java applications within Neovim.

For further customization or advanced features, refer to the respective plugin documentation:

- [nvim-jdtls](https://github.com/mfussenegger/nvim-jdtls)
- [nvim-dap](https://github.com/mfussenegger/nvim-dap)
- [nvim-dap-ui](https://github.com/rcarriga/nvim-dap-ui)

Also refer troubleshooting [[Jdtls]]


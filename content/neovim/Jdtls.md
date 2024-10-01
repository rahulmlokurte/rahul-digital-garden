---
tags:
  - neovim
  - jdtls
---

Download the jdtls from the milestone https://download.eclipse.org/jdtls/milestones/?d

Extract and put it in a folder that needs to be referenced when configuring Java LSP. I have placed the folder at `/home/rahul/software/jdtls`

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
        favoriteStaticMembers = {
          'org.hamcrest.MatcherAssert.assertThat',
          'org.hamcrest.Matchers.*',
          'org.hamcrest.CoreMatchers.*',
          'org.junit.jupiter.api.Assertions.*',
          'java.util.Objects.requireNonNull',
          'java.util.Objects.requireNonNullElse',
          'org.mockito.Mockito.*',
        },
        filteredTypes = {
          'com.sun.*',
          'io.micrometer.shaded.*',
          'java.awt.*',
          'jdk.*',
          'sun.*',
        },
      },
      sources = {
        organizeImports = {
          starThreshold = 9999,
          staticStarThreshold = 9999,
        },
      },
      codeGeneration = {
        toString = {
          template = '${object.className}{${member.name()}=${member.value}, ${otherMembers}}',
        },
        hashCodeEquals = {
          useJava7Objects = true,
        },
        useBlocks = true,
      },
    },
  },
  handlers = {
    ['$/progress'] = function(_, result, ctx)
      -- disable progress updates.
    end,
  },
  init_options = {
    extendedClientCapabilities = jdtls.extendedClientCapabilities,
    bundles = bundles,
  },
}
jdtls.start_or_attach(config)
```


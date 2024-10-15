---
title: Understanding How Mason.nvim Works
tags:
  - neovim
  - mason
---

Due to network firewall restrictions at my company, I was unable to download the `jdtls` plugin for Java LSP directly using mason.nvim. This prompted me to explore how mason.nvim operates, as I was able to download `jdtls` locally. I now need to consider how to make manual setups function similarly to mason.nvim’s automatic process.

## How mason.nvim Works

Mason.nvim is a Neovim plugin for managing external tools like language servers, formatters, and linters. Here's a breakdown of how it handles package installation and executable access:

### 1. Installation Location
By default, mason.nvim installs packages in Neovim's data directory. To check where this directory is, you can run the following command in Neovim:

```vim
:echo stdpath('data')
```

For example, if the output is `/home/rahul/.local/share/nvim`, the structure inside this folder might look like this:
```sql
/home/rahul/.local/share/nvim
├── mason
│   ├── bin
│   ├── packages
│   │   ├── lua-language-server
│   │   ├── pyright
│   │   ├── eslint_d
│   │   └── prettier
│   └── registry
```

- **`mason/`**: The main folder where mason.nvim installs all tools.
- **`bin/`**: This contains symlinks or executables of installed tools. Neovim adds this folder to its `PATH`.
- **`packages/`**: Each tool gets installed in its own subfolder (e.g., `lua-language-server`, `pyright`, etc.).
- **`registry/`**: Stores metadata and versioning information for installed tools.

### 2. Executable Linking

When a package is installed, its executable is linked to the `bin/` directory. For example:
```bash
/home/rahul/.local/share/nvim/mason
├── bin
│   ├── lua-language-server -> ../packages/lua-language-server/bin/lua-language-server
│   ├── pyright -> ../packages/pyright/bin/pyright
│   ├── eslint_d -> ../packages/eslint_d/bin/eslint_d
│   └── prettier -> ../packages/prettier/bin/prettier
```

### 3.Adding to PATH

Mason.nvim automatically adds the `mason/bin` directory to Neovim's `PATH`. This means tools installed by mason.nvim are accessible in:

- Neovim’s command-line (`:!` commands),
- Neovim’s terminal emulator (`:term`),
- and third-party plugins requiring these tools.
You can check Neovim's `PATH` using:
```vim
:echo $PATH
```
You should see something like:
```bash
/home/rahul/.local/share/nvim/mason/bin:/usr/local/bin:/usr/bin:/bin
```

## Folder Structure Summary

```bash
/home/rahul/.local/share/nvim
└── mason
    ├── bin                      # Executables or symlinks to tools
    ├── packages                 # Tool directories
    └── registry                 # Metadata for packages
```

## Manual Installation of Language Server
If you've manually installed a language server (e.g., `my-server`) by downloading it into the `mason/packages/` folder, follow these steps to ensure it works similarly to a mason.nvim-managed tool.

Step 1: Ensure Executable Placement
Place the server's executable in the correct location within the `packages` folder. For example:
```perl
/home/rahul/.local/share/nvim/mason/packages/my-server/bin/my-server
```

Step 2: Create a Symlink
Manually create a symlink in the `mason/bin` directory so that Neovim can access it like other mason tools.
Run the following command in the terminal:
```bash
ln -s /home/rahul/.local/share/nvim/mason/packages/my-server/bin/my-server /home/rahul/.local/share/nvim/mason/bin/my-server
```

This will create a symbolic link in the `bin` folder.

Your `bin/` directory should now look like this:
```bash
/home/rahul/.local/share/nvim/mason/bin
├── lua-language-server -> ../packages/lua-language-server/bin/lua-language-server
├── pyright -> ../packages/pyright/bin/pyright
├── eslint_d -> ../packages/eslint_d/bin/eslint_d
└── my-server -> ../packages/my-server/bin/my-server
```

Step 3: Ensure PATH Includes mason/bin

Mason.nvim automatically adds the `mason/bin` directory to Neovim's `PATH`. You can verify this by running: `:echo $PATH`
Ensure that the output includes:
```bash
/home/rahul/.local/share/nvim/mason/bin:/usr/local/bin:/usr/bin:/bin
```

Now, Neovim should be able to recognize and use the manually installed `my-server` just like any tool installed via mason.nvim.

By following these steps, you can manually install and configure tools in a way that mimics mason.nvim’s automated process.



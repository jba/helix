# Helix (jba fork): prebuilt Linux binary

This archive contains:

- `hx`: the editor, for Linux x86_64. It needs glibc 2.39 or newer
  (Ubuntu 24.04+, Debian 13+, Fedora 40+).
- `runtime/`: themes, syntax queries and compiled tree-sitter grammars.
  Without it, `hx` still runs but has no syntax highlighting, no
  language-aware features and no built-in themes.
- `LICENSE`

## 1. Install the binary

Copy `hx` anywhere you like, preferably a directory on your `PATH`:

```sh
cp hx ~/.local/bin/        # or /usr/local/bin, ~/bin, ...
```

## 2. Let `hx` find the runtime

`hx` looks for the runtime in these places, highest priority first:

1. `~/.config/helix/runtime` (or `$XDG_CONFIG_HOME/helix/runtime`)
2. The directory named by the `HELIX_RUNTIME` environment variable
3. A `runtime` directory next to the `hx` executable (symlinks to `hx` are
   followed)

Pick **one** of the following.

**A. Put it in your Helix config directory (simplest).**

```sh
mkdir -p ~/.config/helix
rm -rf ~/.config/helix/runtime     # remove any older runtime first
cp -r runtime ~/.config/helix/runtime
```

**B. Put it anywhere and set `HELIX_RUNTIME`.**

```sh
mkdir -p ~/.local/share/helix
cp -r runtime ~/.local/share/helix/runtime
echo 'export HELIX_RUNTIME=~/.local/share/helix/runtime' >> ~/.bashrc
```

**C. Keep it next to the binary.** Copy `hx` and `runtime/` into the same
directory, for example `/opt/helix/hx` and `/opt/helix/runtime`. Then put
that directory on your `PATH`, or symlink `hx` into it:

```sh
ln -s /opt/helix/hx ~/.local/bin/hx
```

If `~/.config/helix/runtime` exists from an older Helix install, it takes
priority over B and C. Its old queries can break highlighting, so delete it
or replace it with this runtime.

## 3. Check

```sh
hx --version
hx --health          # shows the runtime directories it found
hx --health rust     # any language; highlight queries should show a check mark
```

## What's different from stock Helix

- **`auto-reload`**: reloads files that changed on disk when the terminal
  regains focus. Buffers with unsaved changes are left alone. Enable it in
  `~/.config/helix/config.toml`:

  ```toml
  [editor]
  auto-reload = true

  # Recommended companion: save when the terminal loses focus.
  [editor.auto-save]
  focus-lost = true
  ```

  Your terminal (and tmux, if you use it: `set -g focus-events on`) must
  report focus events. Stock Helix rejects `auto-reload` as an unknown
  option, so don't share this config with an unmodified Helix.

- **Bracket matching before the cursor**: if the cursor is just after a
  bracket, its partner is highlighted too.

Source and issues: https://github.com/jba/helix

# jba's Helix fork

This is [Helix](https://github.com/helix-editor/helix) with a few extra changes
on top of upstream `master`:

- **Reload on focus** (`auto-reload` option). When the terminal regains focus,
  every buffer whose file changed on disk is reloaded. Buffers with unsaved
  changes are left alone, and the status line reports how many were skipped.
- **Match bracket before the cursor.** If the cursor isn't on a bracket but the
  character just before it is, the matching bracket is highlighted
  (`ui.cursor.match`).
- **Startup log line.** `jba build of helix` is written to the log at startup,
  so you can confirm which build is running (`hx -v`, then check
  `~/.cache/helix/helix.log`).

## Install

You need a Rust toolchain (via [rustup](https://rustup.rs)) and a C compiler
(e.g. `build-essential`) for the tree-sitter grammars. The
`rust-toolchain.toml` in this repo selects the right Rust version.

```sh
git clone https://github.com/jba/helix ~/repos/github.com/jba/helix
cd ~/repos/github.com/jba/helix
cargo install --path helix-term --locked
```

This installs `hx` into `~/.cargo/bin` and builds the grammars into
`runtime/grammars`. Helix then needs to find the `runtime` directory. Link it
next to the binary:

```sh
ln -sfn ~/repos/github.com/jba/helix/runtime ~/.cargo/bin/runtime
```

(Alternatives: symlink it to `~/.config/helix/runtime`, or set `HELIX_RUNTIME`.)

Make sure `~/.cargo/bin` comes before any system `hx` on your `PATH`, then
check:

```sh
hx --version   # should show this repo's commit hash
hx --health    # runtime directory and grammars should be found
```

### Updating

```sh
cd ~/repos/github.com/jba/helix
git pull
cargo install --path helix-term --locked
```

The runtime symlink points into the repo, so it stays current.

### Syncing with upstream

```sh
git remote add upstream https://github.com/helix-editor/helix  # once
git fetch upstream
git rebase upstream/master
git push --force-with-lease origin master
```

## Configuration

Add the following to `~/.config/helix/config.toml`:

```toml
[editor]
# Reload files changed on disk when the terminal regains focus.
# (Fork-only option; stock Helix will reject it as an unknown field.)
auto-reload = true

# Recommended companion: save when the terminal loses focus.
[editor.auto-save]
focus-lost = true
```

Focus events only work if your terminal (and tmux, if used) reports them.
For tmux, add `set -g focus-events on` to `~/.tmux.conf`.

The bracket highlighting needs no configuration, but your theme must define
`ui.cursor.match`, as most themes do.

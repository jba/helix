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

## Install a prebuilt binary (Linux x86_64)

Download the latest `helix-*-x86_64-linux.tar.xz` from the
[releases page](https://github.com/jba/helix/releases). It needs glibc 2.39 or
newer (e.g. Ubuntu 24.04+).

```sh
V=25.07.1-jba.2
curl -LO https://github.com/jba/helix/releases/download/$V/helix-$V-x86_64-linux.tar.xz
tar -xJf helix-$V-x86_64-linux.tar.xz -C ~/.local/share
ln -sfn ~/.local/share/helix-$V-x86_64-linux/hx ~/.local/bin/hx
```

`hx` finds its bundled `runtime/` directory (themes, queries, compiled
grammars) next to the real executable, so keep them together. Then
`hx --health` should report the runtime and grammars. The archive's
`README-release.md` describes other ways to place the runtime.

Other platforms: build from source as below.

## Install from source

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

## Publishing a release (maintainer notes)

Releases are built by hand on Linux x86_64:

```sh
V=<upstream-version>-jba.<n>
N=helix-$V-x86_64-linux
cargo build --release --locked -p helix-term   # also builds runtime/grammars
mkdir -p /tmp/rel/$N && cp target/release/hx LICENSE README-release.md /tmp/rel/$N/
strip /tmp/rel/$N/hx
rsync -a --exclude grammars/sources runtime /tmp/rel/$N/
(cd /tmp/rel && tar -cJf $N.tar.xz $N && sha256sum $N.tar.xz > $N.tar.xz.sha256)
git tag -a $V -m "jba build $V" && git push origin master $V
gh release create $V --title "Helix $V (jba fork)" /tmp/rel/$N.tar.xz /tmp/rel/$N.tar.xz.sha256
```

The binary requires a glibc at least as new as the build machine's.

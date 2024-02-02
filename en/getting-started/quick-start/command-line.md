# Command Line

## Install

In a terminal, install `earendil` by typing:

```shell-session
rustup update # to make sure your Rust is up to date
```

```shell-session
cargo install --git https://github.com/mel-project/earendil.git earendil
```

Check that `earendil` is successfully installed by typing:

```shell-session
earendil
```

You should see:

```shell-session
Usage: earendil <COMMAND>

Commands:
  daemon         Runs an Earendil daemon
  control        Runs a control-protocol verb
  generate-seed
  help           Print this message or the help of the given subcommand(s)

Options:
  -h, --help     Print help
  -V, --version  Print version
```


# Installation

## System requirements

* An up-to-date [Rust](https://www.rust-lang.org/tools/install) installation, with tools like `cargo` and `rustup` in your $PATH. Earendil currently has no official binary distribution, so we'll be compiling it from source.
* For client nodes:
  * At least 1 GB of free RAM and disk space, to compile the program
  * Windows 10, macOS, or Linux
* For relay nodes:
  * A public IP address to serve clients. Generally, you'll find this on cloud servers, VPSes, dedicated servers, etc.
  * At least 1 GB of free RAM and disk space.
  * Only Linux is tested, though any platform that runs Rust is likely to work


## Command line

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

## GUI
## Windows and Mac

To install the Earendil GUI, first download the latest executable for your platform from our [releases page](https://github.com/mel-project/earendil/releases).

Once your download is complete, head to your computer's download folder to locate the file. You'll need to unzip or decompress it first. On Windows, you can do this by right-clicking the file and selecting "Extract All...", while on macOS, you usually just need to double-click the file.

With the file decompressed, you're almost ready to run your program. Locate the executable file in your download folder or within the decompressed files. You can now start the program simply by double-clicking on the executable file.

Please keep in mind that the binary you download must be compatible with your operating system. If the file doesn't run as expected, it's possible you downloaded the wrong version.

## Linux

You can use `cargo` to download, build, and launch the GUI:

1. `$ git clone https://github.com/mel-project/earendil` to get the latest source code on the `master` branch
2. In the `earendil` folder, use `$ cd utilities/earendil-gui` to move into the `earendil-gui`directory
3. Install the GUI using `$ cargo install --path .`
4. Finally, launch it with `$ earendil-gui` !

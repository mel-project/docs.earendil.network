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

## GUI

### Windows and Mac

1. Download the latest executable for *your platform* from our [releases page](https://github.com/mel-project/earendil/releases).

2. Decompress the executable file you just downloaded. 
  - On Windows, you can do this by right-clicking the file and selecting "Extract All...", while on macOS, you can simply double-click the file.

3. Start the program by double-clicking on the executable file!

{% hint style="info" %}
If your executable isn't working, first make sure that you downloaded the correct file for your platform! If it still doesn't work, come to our [Discord](https://discord.gg/AVsGbhzTzx) to ask for help.
{% endhint %}

### Linux

Use `cargo` to download, build, and launch the GUI:

1. Clone the latest source code from the `master` branch: 
  ```
  $ git clone https://github.com/mel-project/earendil
  ```
2. Move into the `earendil-gui`directory: 
  ```
  $ cd earendil/utilities/earendil-gui
  ```
3. Install the GUI: 
  ```
  $ cargo install --path .
  ```
4. Launch the GUI:
  ```
  $ earendil-gui
  ```


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
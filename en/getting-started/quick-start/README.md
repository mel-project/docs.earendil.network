# Installation

`earendil` is the reference implementation of Earendil, running as a background daemon similar to how `tor` runs as Tor's daemon.

This tutorial will teach you how to install `earendil`, run both **client** and a **relay** nodes, as well as create a basic `earendil` config file. It will give you the background needed to learn about hosting [havens](../using-havens.md) and [proxying normal Internet traffic](../browsing-web.md).

## System requirements

* An up-to-date [Rust](https://www.rust-lang.org/tools/install) installation, with tools like `cargo` and `rustup` in your $PATH. Earendil currently has no official binary distribution, so we'll be compiling it from source.
* For client nodes:
  * At least 1 GB of free RAM and disk space, to compile the program
  * Windows 10, macOS, or Linux
* For relay nodes:
  * A public IP address to serve clients. Generally, you'll find this on cloud servers, VPSes, dedicated servers, etc.
  * At least 1 GB of free RAM and disk space.
  * Only Linux is tested, though any platform that runs Rust is likely to work

# GUI

## Windows and Mac

To get started with the Earendil GUI app, you'll first want to download the latest executable for your platform from our [Releases page](https://github.com/mel-project/earendil/releases).

Once your download is complete, head to your computer's download folder to locate the file. You'll need to unzip or decompress it first. On Windows, you can do this by right-clicking the file and selecting "Extract All...", while on macOS, you usually just need to double-click the file.

With the file decompressed, you're almost ready to run your program. Locate the executable file in your download folder or within the decompressed files. You can now start the program simply by double-clicking on the executable file.

Please keep in mind that the binary you download must be compatible with your operating system. If the file doesn't run as expected, it's possible you downloaded the wrong version.

## Linux

You can use `cargo` to download, build, and launch the GUI:

1. `$ git clone https://github.com/mel-project/earendil` to get the latest source code on the `master` branch
2. Use `$ cd utilities/earendil-gui` to move into the `earendil-gui`directory
3. Install it with `$ cargo install --path .`
4. Finally, launch it with `$ earendil-gui` !

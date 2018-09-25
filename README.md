tergent - termux ssh agent
--------------------------

An [ssh-agent implementation](https://tools.ietf.org/id/draft-miller-ssh-agent-02.html) for [Termux](https://termux.com/) that uses [Android Keystore](https://developer.android.com/training/articles/keystore) as its backend.

This application enables the use of keys securely stored in termux-api with ssh-agent protocol capable clients. These clients include the applications provided by openssh, such as `ssh`, `scp`, `ssh-add` and `ssh-copy-id`.

Tergent does not (and cannot) access your private keys as they are stored inside the secure hardware. In fact, they can never leave the chip even with root privileges, thanks to [extraction preventation](https://developer.android.com/training/articles/keystore#ExtractionPrevention).  
Cryptographic actions are performed by the hardware itself.

Compiling
---------
Install [Rust](https://www.rust-lang.org/en-US/install.html) and [Android NDK](https://developer.android.com/ndk/).  
You will need to configure cargo with the correct locations for "ar" and "linker", you can follow this page up to and including the `rustup target add ...` command:  
[https://mozilla.github.io/firefox-browser-architecture/experiments/2017-09-21-rust-on-android.html](https://mozilla.github.io/firefox-browser-architecture/experiments/2017-09-21-rust-on-android.html)  
Then this project can be compiled with the command `cargo build --target=aarch64-linux-android` (or any other Android target).

Alternatively, you can download a debug build for ARM64 Android from the releases page.

Usage
-----
1. (This step will be updated if the patches to termux are merged)  
Make sure you have termux-api app with [this patch](https://github.com/aeolwyr/termux-api/commit/ef3a0af0f6cebf667385034be2c5698c7d3946f2) installed. Inside Termux, install the latest version of the termux-api package using `pkg install termux-api`. Finally, grab a copy of `termux-keystore` script from [this fork](https://github.com/aeolwyr/termux-api-package/blob/master/scripts/termux-keystore).

2. Generate a key using the command `./termux-keystore generate myAlias`, where myAlias is the name you want to give to the key.
 - This command creates an RSA key - to create an ECDSA key, use the argument "-a EC": `./termux-keystore generate myAlias -a EC`.
 - To specify a key size, use the argument "-s", e.g. `./termux-keystore generate myAlias -s 4096` or `./termux-keystore generate myAlias -a EC -s 521`.

3. Verify the key is generated by running the command `./termux-keystore list`.

4. Run tergent with this command: `eval $(./tergent)`.

5. Again list the keys to verify, but now using the standard ssh tool: `ssh-add -l`.

6. Copy the public key to your server. For this, you have two options:
  - ssh-copy-id is the standard tool for this - invoke it by running `ssh-copy-id example.com`.
  - You can also use `ssh-add -L` (notice the uppercase L), and manually copy and append the output to the `.ssh/authorized_keys` file on your server.

7. Connect to your server using the usual command `ssh example.com`.

You will need to run the command `eval $(./tergent)` every time you start up the terminal. This command can be included in .bash_profile (or a similar script) for convenience.

How do I...
-----------
* **list keys**: run either `ssh-add -l` or `termux-keystore list`
* **create a new key**: use `termux-keystore generate`
* **use a key**: just run `ssh`
* **delete a key**: use `termux-keystore delete`
* **import a key**: not supported, generate a new key instead


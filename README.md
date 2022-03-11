# How I installed older Rubies on my Macbook Pro with the M1 chip

As I was setting up my new MPB with the new Apple Silicon M1 chip, I ran into issues installing Rubies older than 2.7.x.  After much Googling, here's what worked for me.

## Assumptions
1. You already have Homebrew installed for Apple Silicon ARM64 architecture (in `/opt/Homebrew`)
2. You already have RVM installed
3. You're using `zsh` as your shell
4. You're comfortable executing commands in your Terminal

## Install Homebrew for Intel architectures

It sounds like a bad idea to have multiple installations of Homebrew on your machine at once, but Homebrew installs itself in completely different locations depending on the architecture it's running on, so it's not too difficult to manage.  I also imagine it will be ok to remove 'old' Homebrew once the Rubies you want are installed.

1. (If you haven't already,) install Rosetta: 
    ```bash
    /usr/sbin/softwareupdate --install-rosetta --agree-to-license
    ```
2. Install Intel-emulated Homebrew to the default `/usr/local`:
    ```bash
    arch --x86_64 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    ```
3. Create an alias for your Intel-Homebrew installation (I called mine 'obrew' for 'old brew'). In `~/.zshrc` add:
    ```bash
    alias obrew='arch --x86_64 /usr/local/Homebrew/bin/brew'
    ```
4. Add ARM Homebrew to your PATH. In `~/.zshrc` add:
    ```bash
    # Homebrew on Apple Silicon
    path=('/opt/homebrew/bin' $path)
    export PATH
    ```
5. Confirm
    `which brew` should return `/opt/homebrew/bin/brew`

    `brew --prefix` should return `/opt/homebrew`

    `which obrew` should return `obrew: aliased to arch --x86_64 /usr/local/Homebrew/bin/brew`

    `obrew --prefix` should return `/usr/local`

## Install Packages Required for Ruby installation

1. `obrew install gmp`
2. `obrew install gnupg`
3. `obrew install openssl@1.1`

## Install Rubies < 2.7.x

`rvm install 2.6.5 -C --with-openssl-dir=/usr/local/opt/openssl@1.1`
or other version

If you get any errors at this point, you may have to set some flags as follows:
```bash
export RUBY_CFLAGS=-DUSE_FFI_CLOSURE_ALLOC
export optflags="-Wno-error=implicit-function-declaration"
```

Then try the `rvm install` again.

After installation, confirm with: `rvm list` which should return something like:
```bash
   ruby-2.6.1 [ x86_64 ]
   ruby-2.6.3 [ x86_64 ]
   ruby-2.6.5 [ x86_64 ]
   ruby-2.7.2 [ arm64 ]
=* ruby-2.7.4 [ arm64 ]
   ruby-3.0.0 [ arm64 ]

# => - current
# =* - current && default
#  * - default
```

Hope this works for you!

### Resources
- [How can I run two isolated installations of Homebrew?](https://stackoverflow.com/questions/64951024/how-can-i-run-two-isolated-installations-of-homebrew)
- [MAC M1 Big Sur ruby compile issue for all rubies. Error running '__rvm_make -j8',](https://github.com/rvm/rvm/issues/5033)

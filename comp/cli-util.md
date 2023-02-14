---
title: Awesome CLI Toys
date: 2022-09-14T15:21:00+08:00
genre: sheet
category: comp
tags:
  - softwares
keywords:
  - cli
  - awesome
lang: en
---

Some nice CLI utilities that can cure your internal mental consumption.

<!-- more -->

Note that the mental consumption introduced by *installing* or *building from source* are not considered.

Also note that this article **DOES NOT** intend to deliberately stress the fact that ~~the Rust language specializes solely on rewriting CLI tools with fancy ANSI escape codes~~.

## Zsh

[`zsh` <img src="https://www.zsh.org/color_vertical_icon.png" alt="zsh" style="display: inline; height: 1em; vertical-align: sub;">](http://www.zsh.org/) is quite a powerful and configurable yet old-schooled shell(comparing to fancy ~~REAL *GEN-Z*~~ stuff like [`warp`](https://www.warp.dev/) or [`nushell`](https://www.nushell.sh/)). Traditionally you would want to install the famous ~~bloatware~~ [`oh-my-zsh`](https://ohmyz.sh/) as a drop-in configuration framework. ~~Wait a minute, this is a shell, not a CLI🤪.~~

```shell
# Install `oh-my-zsh`
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

# Install `oh-my-zsh` with Gitee mirror
sh -c "$(curl -fsSL https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh)"
```

 Then, it would be delightful to have [syntax highlighting](https://github.com/zsh-users/zsh-syntax-highlighting), [completions](https://github.com/zsh-users/zsh-completions) and [auto suggestions](https://github.com/zsh-users/zsh-autosuggestions).

```shell
# Cloning
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# Install via `omz`
omz plugin enable zsh-syntax-highlighting zsh-completions zsh-autosuggestions
```

## Zoxide

[`zoxide`](https://github.com/ajeetdsouza/zoxide) is a Rust implementation of smart working directory changing tools like [`autojump`](https://github.com/wting/autojump) and [`z`](https://github.com/rupa/z). It remembers your previous browsing records, so a tiny fraction of path suffices your next navigation to this directory. 

:::details[Demo]
![zoxide demo](https://github.com/ajeetdsouza/zoxide/raw/main/contrib/tutorial.webp)
:::

```shell
z ~/a/very/long/path/with/a-long-name

# The next time you want to jump to that directory:
z a-lo
```

Read the full installation instructions [here](https://github.com/ajeetdsouza/zoxide#installation).

```shell
# Install on Linux
# This script is COMPATIBLE with local installation
curl -sS https://raw.githubusercontent.com/ajeetdsouza/zoxide/main/install.sh | bash

# ADD the init script to your `.zshrc` 
eval "$(zoxide init zsh)"

# OR use `omz` plugin for `zoxide`
omz plugin enable zoxide

# ALIAS `cd` to `z`
alias cd="z"
```

## Starship 

Although themes like [`powerlevel10k`](https://github.com/romkatv/powerlevel10k) or [`rubbyrussell`](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes#robbyrussell) ~~or `random`~~ looks pretty nice, sometimes you might have to cope with ~~the recognized best Linux desktop environment~~, namely Windows. It would have been a nightmare trying to sync your `pwsh` theme with your `zsh` counterpart. [`starship` <img src="https://starship.rs/icon.png" alt="starship" style="display: inline; height: 1em; vertical-align: sub;">](https://starship.rs/) provides a unified and configurable cross-shell prompting environment, which suits this need perfectly.

:::details[Demo]
<video controls>
  <source src="https://starship.rs/demo.webm" type="video/webm">
</video>
:::

```shell
# Install on Linux
# This root is INCOMPATIBLE with local installation!
#   manually set the env var to install it locally.
export BIN_DIR="/usr/local/bin"
curl -sS https://starship.rs/install.sh | sh

# ADD the init script to your `.zshrc`
eval "$(starship init zsh)"
```

```powershell
# Install on Windows with `scoop`
scoop install starship

# ADD this line to your Powershell profile ($PROFILE).
Invoke-Expression (&starship init powershell)
```

Then nicely disable your `omz` theme and place a configuration in your `~/.config/starship.toml`. Detailed instructions of `starship` can be found [here](https://starship.rs/config/). It is plausible that you want to change a bunch of default prompt icons from weird emojis into pretty [Nerd Fonts](https://www.nerdfonts.com), but as Material Design Icons regularly clash with CJK codepoints, personally it is not recommended. Checkout [official presets](https://starship.rs/presets/#nerd-font-symbols) for more configuration ideas.

```toml
# Starship config schema
"$schema" = 'https://starship.rs/config-schema.json'

# Example: Inject some prompt of the current operating system
format = "[{windows|linux|vps}](cyan) $all"
```

## Fzf

[`fzf`](https://github.com/junegunn/fzf) is an **interactive** fuzzy finder implemented in Go. By default it will walk the file system as a file finder, but it is capable of finding a whole bunch of other stuff, like searching inside text files, managing Docker environments and picking references from your bibliography. Check the [community examples wiki page](https://github.com/junegunn/fzf/wiki/examples) for some handy inspiration. 

:::details[Demo]
![fzf demo](https://raw.githubusercontent.com/junegunn/i/master/fzf-preview.png)
:::

```shell
# Use `git` to install
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

## Fd

[`fd`](https://github.com/sharkdp/fd) is a Rust implementation of GNU `find` from [`findutils`](https://www.gnu.org/software/findutils/). It uses regular expressions by default, which is more intuitive than `find`'s verbose parameters. It self proclaims to be even faster than `find` in its [benchmark](https://github.com/sharkdp/fd#benchmark), as Rust regex automata are fast and the directory traversal part is always carried out in parallel. **Beware that `fd` skips hidden files and gitignore'd files**, you need to use the `-HI` flag to explicitly include them.

:::details[Demo]

![fd demo](https://github.com/sharkdp/fd/raw/master/doc/screencast.svg)

:::

`fd` doesn't come with a install script, so you have to grab them by hand from [the release page](https://github.com/sharkdp/fd/releases). Because of name clash, it is called `fdfind` on `apt` repo.

Refer to [this page](https://github.com/sharkdp/fd#how-to-use) for documentation. If you want to use `fd` to boost your `fzf` search experience, stuff these env vars into your `.zshrc` or so.

```shell
export FZF_DEFAULT_COMMAND='fd --type file'
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"
```

## Ripgrep

This GNU [`grep`](https://www.gnu.org/software/grep/) alternative is handcrafted by the very author of the Rust regex crate. In fact, [the Rust CLI book](https://rust-cli.github.io/book/tutorial/index.html) actually teach you to implement your own `grep`, but this one, often abbreviated as `rg`, is technically way more usable. You may want to check the author's [blog post](https://blog.burntsushi.net/ripgrep/) for a benchmark. Besides, it provides support for Unicode and basic regex right out of box. If you want to use powerful Perl-flavoured regexes(namely PC2E, with fancy stuff like the `(?=foo)` lookahead), you can enable the `-P` flag. 

There is also no install script provided, just grab a prebuilt binary from [its release page](https://github.com/BurntSushi/ripgrep/releases). **Beware that `rg` also skips hidden files and gitignore'd files**.

:::details[Demo]
![](https://burntsushi.net/stuff/ripgrep1.png)
:::

 ## Bat

Yet another Rust-ified CLI tool from the same author of `fd`, `bat` is a human readable version of `cat`. 

![cat on a bin](https://images.unsplash.com/photo-1613341027066-a522e1a04c27?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=774&q=80)

This time, ~~I prefer a cat in my bin than a bat, anyway.~~ But the syntax highlighting and `git` intergration provided by `bat` is kind of tempting.

:::details[Demo]

![bat demo: syntax highlighting](https://camo.githubusercontent.com/7b7c397acc5b91b4c4cf7756015185fe3c5f700f70d256a212de51294a0cf673/68747470733a2f2f696d6775722e636f6d2f724773646e44652e706e67)

![bat demo: git](https://camo.githubusercontent.com/c436c206f2c86605ab2f9fb632dd485afc05fccbf14af472770b0c59d876c9cc/68747470733a2f2f692e696d6775722e636f6d2f326c53573452452e706e67)

:::

Check [this section](https://camo.githubusercontent.com/c436c206f2c86605ab2f9fb632dd485afc05fccbf14af472770b0c59d876c9cc/68747470733a2f2f692e696d6775722e636f6d2f326c53573452452e706e67) for its intergration with other fancy CLI apps. Same as `fd`, grab a binary from the [release page](https://github.com/sharkdp/bat/releases), or check `batcat`(yet another name clash) on `apt` repo.

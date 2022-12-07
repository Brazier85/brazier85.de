---
title: "My Dotfile Setup"
date: 2022-12-07T13:42:04+01:00
draft: false

categories: ["code"]
tags: ["dotfiles", "dotbot"]
toc: false
author: "Ferdinand Berger"
---

## Why I use dotfiles
On regular bases I use the following workstations:
- work windows notebook
- work macbook
- linux notebook
- gaming pc

Therefore my goal with this setup is to provide the same toolset on all of them. Even on Windows with WSL2 I use this setup so I can easily switch between different linux distributions and do not have to configure them all the time. When I update something on one of my workstations I just commit it to my dotfile repository and I can use it on all my other workstations.

## My setup
The backbone for my setup is [dotbot](https://github.com/anishathalye/dotbot). The idea behind this tool ist to use a single command to setup your console or even pc with all the required software you have defined before. In my case the command will look like this when I'm @HOME: `git clone --recurse-submodules $url ~/.dotfiles && cd ~/.dotfiles && ./install` and like so `git clone --recurse-submodules $url && cd dotfiles && ./install -c work` when I'm @WORK.

{{< code type="bash" title="install" >}}
#!/usr/bin/env bash

set -e

# Colors for Messages
GREEN=`tput setaf 2`
RED=`tput setaf 1`
MAGENTA=`tput setaf 5`
RST=`tput sgr0`

# Print some info to stdout
function info {
    echo "${GREEN}$@${RST}"
}

DEFAULT_CONFIG_PREFIX="default"
CONFIG_SUFFIX=".conf.yaml"
#CONFIG="install.conf.yaml"
DOTBOT_DIR="dotbot"
DOTBOT_PLUGIN_DIR="dotbot_plugins"

DOTBOT_BIN="bin/dotbot"
BASEDIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

cd "${BASEDIR}"
git -C "${DOTBOT_DIR}" submodule sync --quiet --recursive
git submodule update --init --recursive "${DOTBOT_DIR}"

#"${BASEDIR}/${DOTBOT_DIR}/${DOTBOT_BIN}" -d "${BASEDIR}" -c "${CONFIG}" "${@}"

for conf in ${DEFAULT_CONFIG_PREFIX} ${@}; do
    info "!!! Starting configuration: ${conf}"
    "${BASEDIR}/${DOTBOT_DIR}/${DOTBOT_BIN}" -d "${BASEDIR}" --plugin-dir "${BASEDIR}/${DOTBOT_PLUGIN_DIR}/dotbot-git" -c "${conf}${CONFIG_SUFFIX}"
done
{{< /code >}}

My basic config will install all the tools I need for my daily work. I use the following folder setup:
{{< code type="yaml" title="folder setup" >}}
[root]
    [config]
        [apt]
            packages.txt
        [fonts] # Some fot files
        [git]
            gitconfig
            gitconfig.work
        [mac]
            Brewfile
        [python]
            pip3-requirements.txt
        [ssh]
            config
            config.work
        [tmux]
            oh-my-tmux # Submodule
            tmux.local.conf
        [vim]
            vimrc
        [zsh]
            alias
            p10k.zsh
            zshrc
            zshrc.work
    [dotbot]
        # Here is a link to the dotbot repo
    [dotbot_plugins]
        # Folder for dotbot plugins
    default.conf.yaml
    install
    work.conf.yaml
{{< /code >}}
**Note:** I can not show all files because they contain some work related stuff :)

The related config file is this here:
{{< code type="yaml" title="default.conf.yaml" >}}
- defaults:
    link:
      relink: true
      create: true
      force: true

- clean: ['~', '~/bin']


- shell:
  - [git submodule update --init --recursive, Installing submodules]


# Default Links for all
- link:
    # ~/.dotfiles: ''

    # SSH
    ~/.ssh/config:
      path: config/ssh/config
      force: true

    # VIM
    ~/.vimrc: config/vim/vimrc

    # GIT
    ~/.gitconfig:
      path: config/git/gitconfig
      force: true

    # ZSH
    ~/.zshrc:
      path: config/zsh/zshrc
      force: true
    ~/.alias: config/zsh/alias
    ~/.p10k.zsh: config/zsh/p10k.zsh

- shell:
  -
    description: install homebrew
    command: |
      if ! type brew > /dev/null 2>&1; then
        /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
        echo '# Set PATH, MANPATH, etc., for Homebrew.' >> ~/.zprofile
        echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.zprofile
        eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
      fi
    stdout: true
    stdin: true
  -
    description: apt install
    command: |
      if [ "$(uname -s)" = "Linux" ]; then
        sudo apt update
        sudo apt install -y $(cat config/apt/packages.txt)
      fi
    stdout: true
    stdin: true
  -
    description: install homebrew formulas
    command: |
      if [ "$(uname -s)" = "Darwin" ]; then
        bash -c "cd config/mac && brew bundle --no-upgrade --no-lock"
      fi
    stdout: true
    stdin: true
  -
    description: install pip3 modules
    command: bash -c "python3 -m pip install -r config/python/pip3-requirements.txt"
    stdout: true
    stdin: true
  -
    description: oh-my-zsh
    command: |
      if [ -d ~/.oh-my-zsh ]; then
        if [ ! -d ~/.oh-my-zsh/.git ]; then
          rm -rf ~/.oh-my-zsh
          sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
        fi
      fi
    stdout: true
    stdin: true

- git:
    '~/.oh-my-zsh/custom/plugins/zsh-autosuggestions':
        url: 'https://github.com/zsh-users/zsh-autosuggestions'
        description: 'oh my zsh - autosuggestions'
    '~/.oh-my-zsh/custom/themes/powerlevel10k':
        url: 'https://github.com/romkatv/powerlevel10k.git'
        description: 'oh my zsh - powerlevel10k'

- shell:
  -
    description: powerlevel10k fonts
    command: |
      if [ "$(uname -s)" = "Darwin" ]; then
        cp "config/fonts/MesloLGS NF Regular.ttf" "~/Library/Fonts/MesloLGS NF Regular.ttf"
      fi
      if [ "$(uname -s)" = "Linux" ]; then
        sudo cp "config/fonts/MesloLGS NF Regular.ttf" "/usr/local/share/fonts/MesloLGS NF Regular.ttf"
      fi
    stdout: true
    stdin: true


- link:
    # TMUX
    ~/.tmux:
      path: config/tmux/oh-my-tmux
      force: true
    ~/.tmux.conf: ~/.tmux/.tmux.conf
    ~/.tmux.conf.local: config/tmux/tmux.conf.local


- shell:
  -
    description: Making zsh the default shell
    command: |
      export user=$(whoami)
      sudo chsh -s "$(which zsh)" "$user"
    stdout: true
    stdin: true
{{< /code >}}

## Can I copy your setup
Yes and no... I am not able to share my whole setup with you but parts of it. So I created a separate repository for this. You can find it here: [dotfiles_public](https://github.com/Brazier85/dotfiles_public)

If you have any questions or ideas about this setup please let me know.
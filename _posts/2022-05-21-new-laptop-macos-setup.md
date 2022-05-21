---
layout: post
title: "Guide: Automate Setting Up New Laptop"
tags:
  - setup
  - automation
description: >
  Guide to automate installing apps and setting up tools for Data Science work.
published: true
---

[Source Code](https://github.com/nastjamakh/macos-setup-data-science)

This guide describes how to install your apps and development tools, as well as configure your coding environment with one script.

Getting a new laptop is, although very exciting, also takes some time to install all your favourote apps and packages. Years ago I used to go through this process manually, by writing down everything installed on my last laptop, and googling links to get it on my shiny new one. Now that command line is an essential part of flow, I much rather spend a few hours writing a few setup scripts than clicking around in web browser.

This post assumes that you work with *brew* and *Visual Studio Code*.

## I. Migrating from Another Laptop
If you do not have a functional laptop with settings you wish to transfer, skip this step.

### Installation script 1: apps

There is a wonderfull brew package `mas` that allows to download *App Store* apps directly from the command line.

List your installed apps with 
~~~bash
brew install mas    # if do not have it installed
mas list

# prefix each app ID with install command and save to a file.
echo '#!/bin/sh'  >> 'apps.sh'
mas list | xargs -L 1 echo | grep -o "^\w*\b" | xargs -L 1 echo mas installed >> 'apps.sh'
brew list --cask | xargs -L 1 echo brew install --cask >> 'apps.sh'
~~~

Unfortunately we cannot make `mas` print just a list of applications IDs. Hence we do a bit of hacking on the command line with `xargs` and `grep`.

* We are adding `#!/bin/sh` at the top of each script to make them executable.
* `xargs` allows us to operate on each line separately (pipe echo and grep commands to each line)
* We select the first string of each line with `grep` (to only select the application ID without name and version)
* each line is then prefixed with installation command
* save results to a script

We append installation commands for apps installed with brew cask in the same script.

### Installation script 2: brew packages

Now we are doing similar things with brew formulae.
~~~bash
echo '#!/bin/sh'  >> 'brew.sh'
brew list --formulae | xargs -L 1 echo brew install >> 'brew.sh'
~~~
### Installation script 3: VS Code extensions

~~~bash
echo '#!/bin/sh'  >> 'vscode.sh'
code --list-extensions | xargs -L 1 echo code --install-extension >> 'vscode.sh'
~~~

Now we have two installation scripts ready, one for brew formulae, one for apps (from brew cask and app store). We just have to append `#!/bin/sh` at the top of each to maek them executable, and run `chmod -R 755 <script.sh>` to get access permissions to run them.
{: .notice-success}


Here is how my `brew.sh` scripts looks like:
~~~bash
#!/bin/sh

############
### Taps ###
############

brew tap homebrew/core
brew tap homebrew/bundle
brew tap homebrew/services
brew tap jondot/tap

# Install Java as dependency
brew install --cask java

##########################
###### System Tools ######
##########################

brew install ack                        # Search tool like grep, but optimized for programmers
brew install bash                       # Bourne-Again SHell, a UNIX command interpreter
brew install bat                        # Clone of cat(1) with syntax highlighting and Git integration
brew install cloc                       # Statistics utility to count lines of code
brew install cmake                      # Cross-platform make
brew install coreutils                  # GNU File, Shell, and Text utilities
brew install curl                       # Get a file from an HTTP, HTTPS or FTP server
brew install denisidoro/tools/navi      # An interactive cheatsheet tool for the command-line ➜  navi
brew install direnv                     # Load/unload environment variables based on $PWD
brew install editorconfig               # Maintain consistent coding style between multiple editors
brew install exa                        # modern replacement for ls
brew install fd                         # Simple, fast and user-friendly alternative to find
brew install gcc                        # GNU compiler collection
brew install glow                       # Render markdown on the CLI
brew install grex                       # Command-line tool for generating regular expressions
brew install httpie                     # User-friendly cURL replacement (command-line HTTP client)
brew install jless                      # Command-line pager for JSON data
brew install mas                        # Mac App Store command-line interface
brew install neovim --HEAD              # NeoVim
brew install pkg-config                 # Manage compile and link flags for libraries
brew install readline                   # Library for command-line editing
brew install ruby                       # Powerful, clean, object-oriented scripting language
brew install rbenv                      # Ruby version manager
brew install slugify                    # Convert filenames and directories to a web friendly format
brew install tmate                      # Instant terminal sharing
brew install tmux                       # Terminal multiplexer
brew install tmuxinator                 # Create and manage tmux sessions easily
brew install tree                       # Display directories as trees (with optional color/HTML output)
brew install urlview                    # URL extractor/launcher (needed for tmux-urlview)
brew install watchman                   # Watch files and take action when they change
brew install wget                       # Internet file retriever
brew install yarn                       # JavaScript package manager
brew install z                          # Tracks most-used directories to make cd smarter

#############
### Git ###
#############
brew install git                        # Distributed revision control system
brew install git-lfs                    # Git extension for versioning large files
brew install gh                         # GitHub command-line tool
brew install bfg                        # Remove large files or passwords from Git history like git-filter-branch
brew install gource                     # Version Control Visualization Tool
brew install pre-commit                 # 

################################
### Data Science Development ###
################################
brew install cookicutter                # Create new projects from boilerplates
brew install six                        # Python 2 and 3 compatibility library
brew install circleci                   # CircleCI command-line tools for CI/CD pipelines
brew install awscli                     # AWS command-line tools
brew install openjdk                    # Open-Source JDK
brew install pyenv                      # Separate python environments
brew install python                     # Interpreted, interactive, object-oriented programming language
brew install docker                     # Pack, ship and run any application as a lightweight container
brew install postgresql                 # Open source relational database management system
brew install sqlite                     # Command-line interface for SQLite
brew install pyenv-virtualenv           # 
brew install libomp                     # LLVM's OpenMP runtime library
~~~

### Setup script: shell

I use zsh, but you can do similar for bash.

Note that installing `iterm2` should be done in a previous step, but I am adding it here as well to have everything relate to shell setup in one place.


~~~bash
#!/bin/sh

brew install --cask iterm2
brew install zsh                        # UNIX shell (command interpreter)

# zsh extensions
brew install zsh-autosuggestions        # 
brew install zsh-syntax-highlighting    # Fish shell like syntax highlighting for zsh

# Copy ZSH config
cp -R .zshrc ~/.zshrc
~~~


### Setup script: git

~~~bash
#!/bin/sh

echo Enter your git username:
read gitusername
echo Enter your git email:
read gitemail
git config --global user.name gitusername
git config --global user.email gitemail
git config --global credential.helper osxkeychain

# Generate new SSH key pair
ssh-keygen -t ed25519 -C gitemail
eval "$(ssh-agent -s)"
echo 'Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519' >> ~/.ssh/config
ssh-add -K ~/.ssh/id_ed25519

# add SSH key to GitHub (you must be logged in)
pbcopy < ~/.ssh/id_ed25519.pub
open https://github.com/settings/ssh/new
~~~

### Putting it all together
~~~bash
#!/bin/sh

# Install Xcode Developer Tools
xcode-select --install

# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Get permissions to acees the scripts
chmod -R 755 .

# Installing Homebrew packages
source ./brew.sh

# Installing Homebrew Cask and App Store apps
source ./apps.sh

# Setting up shell with zsh
source ./shell.sh

# Setting up git
source ./git.sh

# Installing Node.js
source ./nvm.sh

# Installing Python version manager
source ./python.sh

# Install current version of Ruby
source ./ruby.sh

# Install Visual Studio Code Extensions
source ./vscode.sh

# System configuration
source ./config.sh
~~~

## II. New Laptop

Now all we need to do is clone our setup repo on the new laptop, navigate to a folder with our scripts and run

~~~bash
./init.sh
~~~

It will install all your apps and brew packages, configure git on the comamnd line, install python and create virtual environmnets with pyenv.


References:

[1] https://gist.github.com/jbelke/4496b2b1d7900d7971802332234bd781

[2] https://github.com/kogakure/dotfiles/tree/master/setup
---
title: '初始化服务器(Initializing Server)'
description: 'Initialize the server.'
keywords: '初始化,服务器'

date: 2023-01-03T21:46:25+08:00

categories:
  - configuration
tags:
  - server
  - configuration
  - centos 7
---

After reinstalling the system, initialize the centos 7 system.

The following sections are included

1. IP address, static IP, host name, host file
2. Firewall Service
3. epel source and yum source
4. user and assign permissions
5. Common tool.(the_silver_searcher, tmux, axel, wget, curl, zsh, oh my zsh, p10k, vim)
<!--more-->

## IP address, static IP, host name, host file

### Step 1: Modify the IP address configuration file

```shell
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

The content of the file is:

```txt
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
- BOOTPROTO=dhcp
+ BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=a33798c6-2dfc-4b33-b67d-5a8544f9a2c5
DEVICE=ens33
- ONBOOT=no
+ ONBOOT=yes
+ IPADDR=192.168.100.100
+ GATEWAY=192.168.100.1
+ DNS1=192.168.100.1
+ DNS2=223.5.5.5
```

### Step 2: Modify the host name and host file

- change the host name to the name you want:

```shell
vi /etc/hostname
```

- modifying the host file:

```shell
vi /etc/hosts
```

```txt
ip-address hostname
such as:
192.168.100.100 centos100
```

## Disabling the Firewall Service

```shell
systemctl stop firewalld
systemctl disable firewalld.service
```

## Centos7 configuration epel source and yum source

### Step 1: Backup the source

```shell
cd /etc/yum.repos.d/
mkdir bak
mv *.repo bak/
```

### Step 2: Download centos-base.Repo

```shell
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

### Step 3: Yum cache clean and makecache

```shell
yum clean all
yum makecache
```

### Step 4: Install epel source

```shell
yum install -y epel-release.noarch
```

### Step 5: Repeat step 3

### Step 6: View yum enabled

view the enabled repository

```shell
yum repolist enabled
```

view the enabled repository

```shell
yum repolist all
```

### Special: Install using script execution

```shell
vi installrepos.sh
```

```txt
cd /etc/yum.repos.d/
mkdir bak
mv *.repo bak/
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache
yum install -y epel-release.noarch
yum clean all
yum makecache
yum repolist all
```

```shell
chmod 755 installrepos.sh
./installrepos.sh
```

## Install the Development Tools

```shell
yum groupinstall -y "Development Tools"
```

## Create a user and assign permissions

### Step 1: Create a user and change its password

```shell
useradd lucas
passwd ******
```

### Step 2: Configure root permission for the user, so that sudo can execute root permission commands later

```shell
vi /etc/sudoers
```

```txt
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL

## Allows members of the 'sys' group to run networking, software,
## service management apps and more.
# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS

## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL
+ lucas   ALL=(ALL)       ALL
```

## Install the_silver_searcher

```shell
yum install the_silver_searcher
```

## Install tmux

```shell
yum install -y tmux
vi .tmux.conf
```

```conf
# -- general -------------------------------------------------------------------
set -g default-terminal "screen-256color" # colors!
setw -g xterm-keys on
set -s escape-time 10 # faster command sequences
set -sg repeat-time 600 # increase repeat timeout
set-option -g prefix C-a #prefix
unbind-key C-a
bind-key C-a send-prefix
set -q -g status-utf8 on # expect UTF-8 (tmux < 2.2)
setw -q -g utf8 on
set -g history-limit 5000 # boost history
# -- display -------------------------------------------------------------------
set -g base-index 1 # start windows numbering at 1
setw -g pane-base-index 1 # make pane numbering consistent with windows
setw -g automatic-rename on # rename window to reflect current program
set -g renumber-windows on # renumber windows when a window is closed
set -g set-titles on # set terminal title
set -g display-panes-time 800 # slightly longer pane indicators display time
set -g display-time 1000 # slightly longer status messages display time
set -g status-interval 10 # redraw status line every 10 seconds
# clear both screen and history
bind -n C-l send-keys C-l \; run 'sleep 0.1' \; clear-history
# activity
set -g monitor-activity on
set -g visual-activity off
# -- nvigation ----------------------------------------------------------------
# create session
bind C-c new-session
# find session
bind C-f command-prompt -p find-session 'switch-client -t %%'
# split current window horizontally
bind - split-window -v
# split current window vertically
bind _ split-window -h
# pane navigation
bind -r h select-pane -L # move left
bind -r j select-pane -D # move down
bind -r k select-pane -U # move up
bind -r l select-pane -R # move right
bind > swap-pane -D # swap current pane with the next one
bind < swap-pane -U # swap current pane with the previous one
#pane resizing
bind -r H resize-pane -L 2
bind -r J resize-pane -D 2
bind -r K resize-pane -U 2
bind -r L resize-pane -R 2
# window navigation
unbind n
unbind p
bind -r C-h previous-window # select previous window
bind -r C-l next-window # select next window
bind Tab last-window # move to last active window
```

## Install axel

```shell
# https://github.com/axel-download-accelerator/axel/releases/
curl -o axel-2.17.11.tar.gz  https://github.com/axel-download-accelerator/axel/releases/download/v2.17.11/axel-2.17.11.tar.gz

tar zxvf axel-2.17.11.tar.gz
cd axel-2.17.11
./configure
make && make install
```

## Compile and install wget

```shell
# https://ftp.gnu.org/gnu/wget/wget-latest.tar.gz
yum install -y readline-devel
yum install -y gnutls-devel
./configure
make && make install
```

## Compile and install curl

```shell
# https://curl.se/download.html
yum install -y openssl openssl-devel
./configure --with-openssl
make && make install
```

## Compile and install zsh, oh my zsh, p10k

### Step 1: Install zsh and set it as the default shell

```shell
# https://zsh.sourceforge.io/Arc/source.html
./configure
make && make install
vi /etc/shells
chsh -s /usr/local/bin/zsh
```

### Step 2: Install oh my zsh

```shell
# https://gitee.com/mirrors/oh-my-zsh/blob/master/tools/install.sh
vi install.sh
chmod 777 install.sh
./install.sh
```

modifying the remote address

```txt
:59
- REMOTE=${REMOTE:-https://github.com/${REPO}.git}
+ REMOTE=${REMOTE:-https://gitee.com/mirrors/oh-my-zsh.git}
```

```shell
./install.sh
```

### Step 3: Install p10k

```shell
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
vi .zshrc
```

```txt
- ZSH_THEME="robbyrussell"
+ ZSH_THEME="powerlevel10k/powerlevel10k"
+ source /etc/profile
```

```shell
source .zshrc
vi .p10k.zsh
```

```txt
- context                 # user@hostname
+ # context                 # user@hostname
```

```shell
source .p10k.zsh
```

## Install and configure vim

```shell
yum install -y vim
vim /etc/vimrc
```

```txt
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
" Open file type detection
filetype on
" Load the corresponding plugin according to the detected different types
filetype plugin on
" Always show the status bar
set laststatus=2
" Display the current position of the cursor
set ruler
" Turn on line number display
set number
" Highlight current row/column
set cursorline
" set cursorcolumn
" Highlight search results
set hlsearch
" Prohibition of folding
set nowrap
" Turn on syntax highlighting
syntax enable
" Allows replacement of default schemes with the specified syntax highlighting scheme
syntax on
" Adaptive smart indentation in different languages
filetype indent on
" Extend tabs to spaces
set expandtab
" Set tabs to occupy spaces when editing
set tabstop=2
" Set the number of spaces occupied by tabs when formatting
set shiftwidth=2
" Let vim treat a continuous number of spaces as a tab
set softtabstop=2
" Code folding based on indentation or grammar
"set foldmethod=indent
set foldmethod=syntax
" Turn off folding code when starting vim
set nofoldenable
```

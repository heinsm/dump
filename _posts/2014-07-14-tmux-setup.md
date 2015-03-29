---
layout: post
title: tmux setup - my favorite terminal manager
categories: [sysadmin]
tags: [console]
# date: timestamp

---

Ref:

* http://en.wikipedia.org/wiki/Tmux
* http://forums.gentoo.org/viewtopic-t-836006-start-0.html


# Tmux

A linux terminal / virtual console manager similar to Screen.  Screen has many good uses, but tmux is like screen on drugs.

The following describes my favorite bindings, and status line settings.  Also, how to make it part of your default bashrc in such a way that it isn't annoying (or minimizes the annoyance).  If you are like me, this'll help organize your screen life.

## Installing

What does this all do?  well, it:

* Sets up your login to have persistent console/terminal(s).
* Every new terminal session adds a new virtual console to the set.
* The set is common across any terminal session you have open for the current user.
* and the set is persistent until reboot (or you manually exit them..)

Whats this good for? well it:

* allows say compiling in one terminal
    - switching "screens" to a new terminal session
    - and editing a file, or observing cpu consumption, etc...
    - tailing an ongoing file 
    - leaving IRC, telnet, serial sessions open in background...
* allows splitting screen to have both shown at same time, etc...
* basically good anytime you tend to be jumping between different paths, comparing, editing, compiling, installing, etc..
* so basically any real terminal work :] tends to get messy with multiple terminals real quick


Steps for setup:

* install tmux
    - apt-get install tmux
* install tmux.conf into /etc
    - see the [Bindings Section](#bindings) below
* install tmx script: /usr/local/bin/tmx
* chmod for executable, not writable by general public
    - chmod 755 /usr/local/bin/tmx
* add to user's .bashrc(near bottom):

    ```text
         ...
         ..
         .
    ############
    # Customizations
    tmx 0
    ```

## <a name="bindings"></a>Bindings

Save this as file

* in your home directory as ~/.tmux.conf;
* or place into /etc/tmux.conf for global tmux settings

```text
# Make it use C-a, similar to screen..
unbind C-b
unbind l
set -g prefix C-a
bind-key C-a last-window
 
# Reload key
bind r source-file ~/.tmux.conf
 
set -g default-terminal "screen-256color"
set -g history-limit 1000
set -g set-titles on
set -g set-titles-string "#T"
 
# THEME
set -g status-bg black
set -g status-fg white
set -g status-interval 60
set -g status-left-length 30
set -g status-left '#[fg=green](#S) #(whoami)@#H#[default]'
set -g status-right '#[fg=yellow]#(cut -d " " -f 1-3 /proc/loadavg)#[default] #[fg=blue]%H:%M#[default]'
set-window-option -g window-status-current-bg red
set-option -g mouse-select-pane on
setw -g mode-mouse on
#set -g terminal-overrides 'xterm*:smcup@:rmcup@'
set -g terminal-overrides "xterm*:XT:smcup@:rmcup@"
setw -g aggressive-resize on
 
# set correct term
set -g default-terminal screen-256color

#bind changes
#splitscreen
unbind % # Remove vertical
bind | split-window -h
bind _ split-window -v

#pane movement
bind-key ! command-prompt -p "swap pane with:"  "swap-pane -U -t '%%'"
```


## Installing as part of your login profile

To have Tmux automatically open up new virtual console, and attach to your on-going vconsole set - this script can be added to your bashrc to make this simple.

```bash
#!/bin/bash

# Author: Heinsm
# Modified TMUX start script from:
#     http://forums.gentoo.org/viewtopic-t-836006-start-0.html
#

# Works because bash automatically trims by assigning to variables and by 
# passing arguments
trim() { echo $1; }

if [[ -z "$1" ]]; then
    echo "Specify session name as the first argument"
    exit
fi

# Only because I often issue `ls` to this script by accident
if [[ "$1" == "ls" ]]; then
    tmux ls
    exit
fi

base_session="$1"
# This actually works without the trim() on all systems except OSX
tmux_nb=$(trim `tmux ls | grep "^$base_session" | wc -l`)
if [[ "$tmux_nb" == "0" ]]; then
    echo "Launching tmux base session $base_session ..."
    tmux new-session -s $base_session
else
    # Make sure we are not already in a tmux session
    if [[ -z "$TMUX" ]]; then
        # Kill defunct sessions first
        old_sessions=$(tmux ls 2>/dev/null | egrep "^[0-9]{14}.*[0-9]+\)$" | cut -f 1 -d:)
        for old_session_id in $old_sessions; do
            tmux kill-session -t $old_session_id
        done

        echo "Launching copy of base session $base_session ..."
        # Session is is date and time to prevent conflict
        session_id=`date +%Y%m%d%H%M%S`
        # Create a new session (without attaching it) and link to base session 
        # to share windows
        tmux new-session -d -t $base_session -s $session_id
        # Create a new window in that session
        tmux new-window
        # Attach to the new session
        tmux attach-session -t $session_id
        # When we detach from it, kill the session
        tmux kill-session -t $session_id
    fi
fi 
```

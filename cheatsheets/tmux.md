# tmux cheatsheet

tmux 3.4. Prefix is `<C-b>` by default. `prefix + key` means press Ctrl+b, then
release, then the key.

## Sessions

```bash
tmux                             # new session
tmux new -s work                 # named session
tmux ls                          # list sessions
tmux attach -t work              # attach
tmux a -t work                   # same, short
tmux kill-session -t work
tmux kill-server                 # kill everything (nuclear)
tmux new -s work -d              # detached, for scripts
```

In tmux:

```
prefix d          detach
prefix $          rename session
prefix s          list/switch sessions
prefix ( )        prev / next session
```

## Windows (tabs)

```
prefix c          create window
prefix ,          rename window
prefix n / p      next / previous window
prefix 0..9       switch to window by number
prefix w          list windows
prefix &          kill window (confirm)
prefix f          find window by name
```

## Panes (splits)

```
prefix %          split vertically (left/right)
prefix "          split horizontally (top/bottom)
prefix o          cycle panes
prefix arrow      move to pane in that direction
prefix q          show pane numbers, press one to jump
prefix x          kill pane (confirm)
prefix z          zoom pane to full window (toggle)
prefix space      cycle pane layouts
prefix { }        move pane left / right
prefix !          break pane into its own window
```

Resize panes (repeatable):

```
prefix <C-arrow>  resize by 1
prefix <M-arrow>  resize by 5    (M = Alt)
prefix <C-b>      if you rebind prefix; otherwise use the arrow forms
```

## Copy mode / scrollback

```
prefix [          enter copy mode
  q               exit
  arrows / hjkl   move
  / ?             search forward / back
  n N             next / prev match
  space           start selection
  enter           copy selection and exit
  g / G           top / bottom of history
prefix ]          paste
```

Scrolling with the mouse is easier if you enable it (see config below).

## Command mode / misc

```
prefix :          command prompt
prefix ?          list all key bindings
prefix t          big clock
prefix r          reload config (if bound)
prefix :set -g mouse on
```

## Config: ~/.tmux.conf

```tmux
# better prefix: C-a is easier than C-b
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# sane defaults
set -g mouse on
set -g history-limit 50000
setw -g mode-keys vi
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on

# reload config with prefix r
bind r source-file ~/.tmux.conf \; display "reloaded"

# split with | and -
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"

# move between panes with vim keys
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# start copy-mode with vi keys
setw -g mode-keys vi
```

Reload without restarting:

```bash
tmux source-file ~/.tmux.conf
```

## Scripting / automation

```bash
# start a session with a couple of windows for a project
tmux new -s dev -d
tmux send-keys -t dev 'cd ~/proj' C-m
tmux new-window -t dev -n logs
tmux send-keys -t dev:logs 'tail -f app.log' C-m
tmux attach -t dev
```

`C-m` is Enter. Without it, the command is typed but not run. I forget this every
time.

## Gotchas

- `tmux ls` shows sessions; if you're inside tmux, `prefix s` is faster.
- Detaching (`prefix d`) keeps everything running. Closing the terminal without
  detaching also leaves it running; `tmux a` brings it back.
- Scrollback is lost on reboot unless you configured it; tmux is not persistent
  storage.
- Mouse selection can conflict with your terminal's own selection. Hold Shift
  (or Option on macOS) to use the terminal's native copy.
- `set -g mouse on` makes scrolling with the wheel work, but also means you
  can't select text with the mouse unless you bypass with Shift.

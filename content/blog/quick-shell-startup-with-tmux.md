+++
title = "Faster terminal startup with tmux"
date = 2024-03-16
draft = false
[taxonomies]
tags = ["mac", "tmux"]
+++

I recently started a new job. The company cares about security; the
computer I've been assigned has some program-startup-latency which I'm
confident has something to do with various security features (it's a
super powerful M2 MacBook Pro that, most of the time, is very fast).

One slow action is simply starting a new shell (I use Bash). Starting
Bash requires waiting nearly a second (my `.bashrc` isn't doing much;
the equivalent `.bashrc` on my personal Linux box provides an
unnoticeable startup time). If the shell is slow to start, then
obviously a fresh terminal which spins up a shell will be slow to
start. Independent of terminal emulator use (I've tried multiple), it
is a little frustrating to type `Cmd+N` and wait a second, or click on
the icon in the dock and wait a second. Both of those actions frequent
and simple muscle memory, so it adds up throughout the day.

I've been using [tmux](https://github.com/tmux/tmux/wiki) for years,
but mostly just as a way to keep shells running on remote machines
(and never really finding it useful for local work). With the slow
shell startup I've been experiencing I decided to see if it could
help, and it sure does. Instead of new sessions of my terminal
emulator running `bash`, they run `tmux` and attach (or create, if
necessary) to a `tmux` session called `scratch` (taking inspiration
from the best:
[Emacs](https://www.gnu.org/software/emacs/manual/html_node/emacs/Lisp-Interaction.html)).

With [Alacritty](https://alacritty.org/) this is enabled by adding to
my `alacritty.toml` file:

```toml
[shell]
program = "/opt/homebrew/bin/tmux"
args = ["new", "-A", "-s", "scratch"]
```

It can also be accomplished with Terminal.app by modifying the
Profile's shell tab to run the command (same as above):

```
/opt/homebrew/bin/tmux new -A -s scratch
```

Now my terminals start without the 1 second drag, and I get to keep my
scratch shell going all the time. It's also helping me get more out of
tmux locally; I'm always in it jumping around sessions, windows, and
panes. It's really a great tool.

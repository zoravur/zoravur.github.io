+++
title = 'A Nice Shell Alias'
date = 2025-09-08T01:12:56-04:00
draft = false
tags = ['Mnemonics', 'Tricks']
+++

You know how those installations always expect you to restart your shell for changes to take effect? closing / reopening and then `cd`ing to your current directory can feel tedious. Luckily, there's a bash one-liner that restarts your shell, in window:
```bash
exec "$SHELL" -l
```
Dissection:
- `exec` -- run command, replacing current process
- `"$SHELL"` -- run the default shell.
- `-l` -- Run it as a login shell.

I don't know what the implications are of reloading a shell, but this does what I want right after I'm done installing something.

I've created an alias for this:
```bash
alias reload='exec "$SHELL" -l'
```

Pretty nifty.



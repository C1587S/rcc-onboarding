# Using `tmux` to Manage Long Sessions

Learn how to keep your work running even when you disconnect from the cluster.

## Why use [`tmux`](https://github.com/tmux/tmux)?

`tmux` (terminal multiplexer) is a tool that lets you:
- Start long-running processes and return to them later
- Disconnect from Midway without interrupting your session
- Split your terminal into multiple panes or windows
- Monitor multiple tasks at once (e.g. logs, jobs, data inspection)

It is especially useful for:
- Interactive Slurm sessions (`sinteractive`)
- Launching notebooks or data pipelines
- Debugging code interactively without losing progress

## Basic commands

### Starting a new session

```bash
tmux new -s mysession
```

This opens a new terminal inside `tmux` named `mysession`.

### Detaching from a session

To safely leave a `tmux` session without stopping it:

<kbd>Ctrl</kbd> + <kbd>b</kbd>, then <kbd>d</kbd>

(Press and release Ctrl+b, then press d)

### Reattaching to a session

List your existing sessions:
```bash
tmux ls
```

Reattach:
```bash
tmux attach -t mysession
```

### Splitting your terminal

Once inside a `tmux` session:

- Horizontal split: <kbd>Ctrl</kbd> + <kbd>b</kbd>, then <kbd>"</kbd>
- Vertical split: <kbd>Ctrl</kbd> + <kbd>b</kbd>, then <kbd>%</kbd>
- Switch between panes: <kbd>Ctrl</kbd> + <kbd>b</kbd>, then arrow keys

### Ending a session

Inside each pane, type `exit` to close it. Once all panes are closed, the session ends.

## Common workflow: interactive session + tmux

1. Connect to Midway:
   ```bash
   ssh YOURUSER@midway3.rcc.uchicago.edu
   ```

2. Start a `tmux` session:
   ```bash
   tmux new -s test_session
   ```

3. Launch your interactive Slurm job:
   ```bash
   sinteractive
   ```

4. Run your work (e.g. notebook, script, model)

5. Detach safely:
   - <kbd>Ctrl</kbd> + <kbd>b</kbd>, then <kbd>d</kbd>

6. Reconnect later and reattach:
   ```bash
   ssh YOURUSER@midway3.rcc.uchicago.edu
   tmux attach -t test_session
   ```

## What if I lose my session?

!!! warning
    If you log out of the cluster and reconnect later, you may end up on a different login node. `tmux` sessions are tied to the specific login node where they were started.
    So if you don't see your session with `tmux ls`, it's likely you're not on the same node.

### Example: what can go wrong

You connected to `midway3-login3` and started a tmux session there.  
Later, you reconnect using:

```bash
ssh YOURUSER@midway3.rcc.uchicago.edu
```

But this puts you on `midway3-login2`, and now:

```bash
tmux ls
```

returns:
```
no server running on /tmp/tmux-...
```

This doesn't mean your session is gone — just that you're on the wrong node.

### How to fix it

Try logging in directly to other login nodes to find where your session is:

```bash
ssh YOURUSER@midway3-login3.rcc.uchicago.edu
tmux ls
```

Or from within any Midway session:

```bash
for i in {2..4}; do
  ssh midway3-login$i "hostname && tmux ls"
done
```

Once you find it:

```bash
ssh midway3-login3
tmux attach -t test_session
```

## Shortcut summary

| Action               | Mac/Linux Shortcut         | Windows (WSL) |
|----------------------|----------------------------|---------------|
| Detach               | `Ctrl + b`, then `d`       | Same          |
| Split horizontal     | `Ctrl + b`, then `"`       | Same          |
| Split vertical       | `Ctrl + b`, then `%`       | Same          |
| Switch panes         | `Ctrl + b`, then arrow key | Same          |
| Kill pane            | `exit` or `Ctrl + d`       | Same          |
| List sessions        | `tmux ls`                  | Same          |
| Attach to session    | `tmux attach -t NAME`      | Same          |
| Kill session         | `tmux kill-session -t NAME`| Same          |

## Next steps

- Learn how to set up [interactive sessions](./interactive-sessions.md)
- Run your first [Slurm batch job](./first-slurm-job.md)
- Set up [Conda environments](../environment-setup/conda-basics.md)

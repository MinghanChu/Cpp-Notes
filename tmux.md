tmux cheat sheet

1. get started `tmux`

2. prefix key `C-b`, which means **press `Ctrl` and `b` at the same time, release both, and then type `commands`** 

3. **Commands**

   1. **splitting panes horizontally:** `C-b` +`%`(shift+key 5) 

   2. **splitting panes vertically:** `C-b`+`"`

   3. **Navigating panes:**`C-b`+`<arrow key>`

   4. **closing panes:** `exit` or `Ctrl-d`

   5. **Creating Windows** (note a single window can contain many panes)

      `C-b`+`c`(repeat 1-4 if you want to play with panes in the new window)

      `C-b`+`<number>` (in case many windows have created)

   6. **Session Handling**

      `C-b`+`d` to detach current session

      `C-b`+`D` to let `tmux` give you a choice which of your sessions you want to detach

      `tmux ls` list of all running sessions

      `tmux attach -t 0` to attach the running session with the parameter`0`

      `tmux rename-session -t 0 newname` give a new name to an existing session

      next time you can just attach the renamed session by `tmux rename-session -t 0 newname`


# tmux

## Session Management

Start a new session named "bla"
* `tmux new-session bla`

Attach to a running session. If no name is given, the last session will be attached to.
* `tmux attach [bla]`

List active sessions
* `tmux ls`

Kill all running sessions.
* `tmux kill-server`

## Pane Management

Select the pane in the respective direction.
* `Ctrl-b Left,Right,Up,Down`

Create pane to the right (horizontally)
* `Ctrl-b %`

Create pane below (vertical)
* `Ctrl-b "`

Current pane is maximized or unmaximized.
* `Ctrl-b z`

Enter scrollback mode in a pane.
* `Ctrl-b [`

Switch to last pane (the last pane is the one with the minus after the title)
* `Ctrl-b l`

## Window Management
Create new Window.
* `Ctrl-b c`

Kill current window (will ask for permission)
* `Ctrl-b &`

Select window at index 0.
* `Ctrl-b 0`

Rename the current window.
* `Ctrl-b ,`

## Copy and Pasting

If mouse mode is on, the easiest way to copy is to just select the text. Tmux has its own clipboard.

Pasting is done with: `Ctrl-b ]`



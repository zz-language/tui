# tui

Retained-mode terminal UI kit for ZZ: own the screen, redraw frames,
dispatch single keys. Where `ask` answers one question and restores
the terminal, `tui` runs the whole show — board games, dashboards,
editors.

Requires a toolchain with `std.term.poll` (post-0.2.2 dev line; the
released 0.2.2 `d_zz` does not have it yet).

```zz
import tui.screen
import tui.keys

screen.enter()
defer screen.exit()
k := keys.read()
if k == keys.up() { println("up") }
```

## Modules

| Module | Contents |
|--------|----------|
| `tui.screen` | `enter`/`exit`, `goto`, `clear`, `clear_line`, hide/show cursor, `size`, `flush`, `write`, `is_tty` |
| `tui.keys` | Key codes, pure `decode` over byte buffers, live `read` with Esc disambiguation |

## Key codes

Singles pass through as their byte (`13` Enter, `32` Space, `127`
Backspace, `3` Ctrl+C). Arrows decode to `1000`-`1003` (same numbering
as `ask`), Home/End/Delete to `1004`-`1006`. `-1` is EOF/closed stdin
(returned, never spun on), `-3` from `decode` means "need more bytes".

## Rules the package lives by

- Every input path returns: closed stdin yields EOF on the first
  failed read. No retry loops against no input.
- Non-TTY callers branch on `screen.is_tty()` — nothing here assumes
  a terminal.
- `defer screen.exit()` restores cooked mode on every exit path.

## Status

0.1.0: screen + keys + hermetic decode tests. Widgets (menu, textline,
canvas) land in 0.2.0.

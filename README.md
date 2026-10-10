```
┌────────────────────────────────────────┐
│  tui — own the terminal                │
│  screens, keys, widgets. pure ZZ.      │
└────────────────────────────────────────┘
```

Where `ask` answers one question and restores the terminal, `tui`
runs the whole show — board games, dashboards, editors, file browsers.

## Install

```bash
zz add tui
```

> Requires a dev toolchain with `std.term.poll` (the released
> toolchain does not have it yet — build `zz` from source).

## Hello, screen

```zz
import tui.screen
import tui.keys

func main() {
    screen.enter()
    defer screen.exit()
    screen.clear()
    println("press q to quit")
    live := 1
    while live == 1 {
        if keys.read() == 113 {
            live = 0
        }
    }
}
```

## One function per need

```zz
import tui.menu
import tui.textline
import tui.dialog
import tui.canvas

// Fullscreen picker. Arrows/hjkl + Enter. -1 on Esc.
idx := menu.choose("Pick your hero", ["mage", "rogue", "bard"])

// Editable line. Returns (text, confirmed).
(name, ok) := textline.edit("Name: ", "")

// Centered yes/no over your screen.
if dialog.confirm_box("Overwrite save?") {
    println("overwritten")
}

// Retained grid: draw anywhere, blit once.
c := canvas.blank(80, 24, " ")
c = canvas.box(c, 0, 0, 20, 7, 1)
c = canvas.text(c, 2, 1, "quest log")
c = canvas.text(c, 2, 3, canvas.bar(0.625, 8))
canvas.present(canvas.render(c))
```

## Modules

| Module | You get |
|--------|---------|
| `tui.screen` | `enter`/`exit`, `goto`, `clear`, cursor hide/show, `size`, `flush`, `write`, `is_tty`, color `mode` detect |
| `tui.keys` | Key codes (arrows, Home/End/Delete, PgUp/PgDn), pure `decode`, live `read` with Esc disambiguation |
| `tui.menu` | Wrap-around `step`, scrolling fullscreen `choose`, numbered fallback |
| `tui.multi` | Fullscreen checkbox multi-pick with `toggle` core |
| `tui.textline` | Pure `insert`/`erase`, live `edit` with cursor |
| `tui.tabs` | Tab strip `render` + `step` (bring your own loop) |
| `tui.layout` | `visible`/`pad`/`center`/`trunc` width-aware string ops |
| `tui.status` | One-line `bar(left, center, right, w)` |
| `tui.canvas` | Retained grid, `box`/`hline`/`vline`, `text`, styled overlays, `render`/`present`, `bar` |
| `tui.dialog` | Centered `note` + `confirm_box` with non-TTY fallbacks |
| `tui.log` | Capped message buffer + viewport `view` |
| `tui.mouse` | SGR mouse enable/decode/live `next` |
| `tui.clip` | OSC 52 `copy` + timed `paste` query |
| `tui.width` | Display widths (CJK/emoji) for column math |

## Keys at a glance

Singles pass through as their byte (`13` Enter, `32` Space, `127`
Backspace, `3` Ctrl+C). Arrows decode to `1000`–`1003`, Home/End/Delete
to `1004`–`1006` — the same numbering as `ask`. `-1` is EOF, returned,
never spun on. Full table: [`docs/keys.md`](docs/keys.md).

## Rules this package lives by

- **Every input path returns.** Closed stdin yields EOF on the first
  failed read. Piped stdin degrades to line mode. Nothing spins.
- **No TTY assumed.** Branch on `screen.is_tty()`; everything works
  (uglier) over a pipe, so examples stay testable.
- **Pure cores, thin live shells.** Navigation, buffer ops, rendering
  and decoding are pure functions with `zz test` coverage; terminal
  I/O lives in small wrappers around them.
- **Not a framework.** No app object, no prescribed loop — composable
  pieces you wire yourself. (A wrapper framework over `table`, `ask`
  and `tui` is a separate future package.)

## Docs

- [`docs/getting-started.md`](docs/getting-started.md) — first app in ten minutes
- [`docs/api.md`](docs/api.md) — every function, every signature
- [`docs/keys.md`](docs/keys.md) — keycode table + decoding guide
- [`docs/cookbook.md`](docs/cookbook.md) — game loop, dialogs, progress, quit-confirm

## Status

Screen + keys + menu + textline + canvas + dialog, all tested. See
[`docs/getting-started.md`](docs/getting-started.md) for what's next.

MIT — see [LICENSE](LICENSE).

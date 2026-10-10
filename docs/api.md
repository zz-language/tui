# API reference

Conventions: `-1` means cancelled/EOF everywhere it can appear.
Coordinates are 0-based cells, `(0, 0)` top-left. `bool` flags are
plain ints (`1`/`0`) only where noted — most predicates return `bool`.

## `tui.screen`

Own the terminal. Enter once, `defer` the exit, draw, flush.

| Function | Signature | Notes |
|----------|-----------|-------|
| `enter` | `() -> bool` | Raw + alt screen + hidden cursor. `false` when no TTY (terminal untouched) |
| `mode` | `() -> int` | Color capability: 3 truecolor, 2 256-color, 1 basic16, 0 none (`NO_COLOR` wins) |
| `exit` | `() -> unit` | Cursor back, main screen back, cooked mode. Idempotent, errors swallowed |
| `is_tty` | `() -> bool` | Gate `enter` on this for clean pipes |
| `size` | `() -> [int]` | `[cols, rows]`; `[80, 24]` when unknown |
| `goto` | `(x: int, y: int) -> unit` | Move cursor (0-based) |
| `clear` | `() -> unit` | Wipe + home |
| `clear_line` | `() -> unit` | Wipe current line |
| `hide` / `show` | `() -> unit` | Cursor visibility |
| `flush` | `() -> unit` | Push output now (call after every frame) |
| `write` | `(s: str) -> unit` | Print with no newline |

Lifecycle pattern:

```zz
screen.enter()
defer screen.exit()
```

## `tui.keys`

Codes: singles are raw bytes (`13` Enter, `32` Space, `127`
Backspace, `3` Ctrl+C, `9` Tab); arrows `1000`–`1003`
(Left, Right, Up, Down); `1004` Home, `1005` End, `1006` Delete,
`1007` PgUp, `1008` PgDn (the page keys extend `ask`'s table for
scrolling lists); `-1` EOF/closed stdin; `-3` (`decode` only) means
"need more bytes".

| Function | Signature | Notes |
|----------|-----------|-------|
| `eof`, `ctrl_c`, `tab`, `enter`, `esc`, `space`, `backspace` | `() -> int` | Named singles |
| `left`, `right`, `up`, `down`, `home`, `end`, `delete` | `() -> int` | Named specials |
| `pgup`, `pgdn` | `() -> int` | Page keys (scrolling lists) |
| `decode_csi` | `(fin: int) -> int` | CSI/SS3 final byte → code (pure) |
| `decode_tilde` | `(n: int) -> int` | `~`-sequence number → code (pure) |
| `decode` | `(buf: [int]) -> (int, int)` | `(key, consumed)`; `consumed == 0` needs more bytes (pure) |
| `read` | `() -> int` | One logical key. Lone Esc returns after 50 ms; EOF returns `-1` |

Byte layout decoded: `27` alone, `27, 91, X` (CSI), `27, 79, X`
(SS3), `27, 91, N, 126` (tilde). Unknown finals pass through as
their own byte; truncated buffers report `-3`.

## `tui.menu`

| Function | Signature | Notes |
|----------|-----------|-------|
| `step` | `(sel: int, n: int, key: int) -> int` | Pure wrap-around navigation (arrows/hjkl) |
| `choose` | `(title: str, options: [str]) -> int` | Fullscreen list; index or `-1` on Esc/EOF/empty; numbered fallback on pipes |

`choose` highlights with marker + reverse video so it reads on
colorless terminals too.

## `tui.textline`

| Function | Signature | Notes |
|----------|-----------|-------|
| `insert` | `(buf: str, pos: int, ch: str) -> (str, int)` | Pure splice → `(text, cursor)` |
| `erase` | `(buf: str, pos: int) -> (str, int)` | Pure backspace → `(text, cursor)` |
| `edit` | `(prompt: str, initial: str) -> (str, bool)` | Live edit → `(text, confirmed)`. Enter confirms, Esc/EOF cancels to `initial`; pipes do one plain `input()` |

Editing keys: printable insert, Backspace, Left/Right/Home/End,
Enter, Esc. The cursor tracks via horizontal-absolute positioning.

## `tui.canvas`

`Canvas` is `{ w, h, rows, ox, oy, og }` (value semantics —
builders return a new canvas). Rows hold plain glyphs; `styled`
overlays fully replace their target cell at render, so overlay
targets must be single-char cells while static chrome may carry ANSI
directly. Drawing touches only changed rows: boxes and frames cost
O(rows), single cells and overlays O(1).

| Function | Signature | Notes |
|----------|-----------|-------|
| `blank` | `(w: int, h: int, fill: str) -> Canvas` | Fresh grid |
| `set` | `(c: Canvas, x: int, y: int, g: str) -> Canvas` | One cell; OOB ignored |
| `get` | `(c: Canvas, x: int, y: int) -> str` | One cell; `""` OOB |
| `text` | `(c: Canvas, x: int, y: int, s: str) -> Canvas` | Row string, clipped |
| `box` | `(c: Canvas, x: int, y: int, w: int, h: int, pretty: int) -> Canvas` | Border; nonzero `pretty` = unicode. One splice per row |
| `hline` / `vline` | `(c: Canvas, x: int, y: int, n: int, g: str) -> Canvas` | Clipped runs, one pass each |
| `styled` | `(c: Canvas, x: int, y: int, g: str) -> Canvas` | Styled overlay replacing the cell at render (O(1) append) |
| `clear_ov` | `(c: Canvas) -> Canvas` | Drop overlays, keep the grid |
| `render` | `(c: Canvas) -> [str]` | Row strings (ANSI preserved) |
| `present` | `(rows: [str]) -> unit` | Blit from top-left + flush (no clear → no flicker) |
| `bar` | `(frac: float, w: int) -> str` | `"[█████░░░]"`, frac clamped to [0, 1] |

Glyphs should be single-cell (ASCII or wide-safe symbols like
`♞`); CJK/emoji widths are not measured.

## `tui.dialog`
| Function | Signature | Notes |
|----------|-----------|-------|
| `note` | `(lines: [str]) -> int` | Centered box until any key; returns the key (Enter on pipes) |
| `confirm_box` | `(msg: str) -> bool` | `y`/`Y`/Enter confirm; `n`/`N`/Esc/EOF decline; `[y/N]` prompt on pipes |

## `tui.multi`

Fullscreen checkbox list (screen twin of prompt-style picking).

| Function | Signature | Notes |
|----------|-----------|-------|
| `toggle` | `(picked: [int], idx: int) -> [int]` | Pure sorted insert/remove (pure) |
| `choose` | `(title: str, options: [str]) -> ([int], bool)` | Space toggles, Enter confirms set, Esc/EOF cancels; comma-list fallback on pipes |

## `tui.tabs`

Tab strip for multi-view screens (no loop prescribed — drive with `read`).

| Function | Signature | Notes |
|----------|-----------|-------|
| `step` | `(sel: int, n: int, key: int) -> int` | Wrap-around (arrows/hjkl), pure |
| `render` | `(titles: [str], sel: int) -> str` | `" a │ b "` strip, active reversed (pure) |

## `tui.layout`

Width-aware string ops (visible cells via `strip`, so styled strings align).

| Function | Signature | Notes |
|----------|-----------|-------|
| `visible` | `(s: str) -> int` | Cell count excluding escapes (pure) |
| `pad` | `(s: str, w: int) -> str` | Right-pad to width (pure) |
| `center` | `(s: str, w: int) -> str` | Center in width, extra space right (pure) |
| `trunc` | `(s: str, w: int) -> str` | Clip with `…` (plain strings; strip styled input first) (pure) |

## `tui.status`

| Function | Signature | Notes |
|----------|-----------|-------|
| `bar` | `(left: str, center: str, right: str, w: int) -> str` | One exact-width line; center truncates first (pure) |

## `tui.log`

`Log` is `{ lines: [str], cap: int }` (value semantics).

| Function | Signature | Notes |
|----------|-----------|-------|
| `blank` | `(cap: int) -> Log` | Empty buffer |
| `push` | `(l: Log, s: str) -> Log` | Append, drop oldest past cap |
| `view` | `(l: Log, height: int, scroll: int) -> [str]` | Last `height` lines ending `scroll` up from the tail |

## `tui.mouse`

SGR mouse (1006). Buttons: 0 left, 1 middle, 2 right, 64/65 wheel;
release adds 1000. Coordinates 0-based cells.

| Function | Signature | Notes |
|----------|-----------|-------|
| `enable` | `() -> bool` | Send mode escapes (false when no TTY, nothing sent) |
| `disable` | `() -> unit` | Leave mouse mode |
| `decode` | `(buf: [int]) -> Click` | Pure; `n` 0 = truncated, 1 = resync past non-mouse |
| `next` | `() -> Click` | Blocking read of one event; EOF yields `b = -1` |

`Click` is `{ b: int, x: int, y: int, n: int }`.

## `tui.clip`

OSC 52 clipboard (works over SSH).

| Function | Signature | Notes |
|----------|-----------|-------|
| `copy` | `(s: str) -> bool` | False when no TTY (nothing sent) |
| `paste` | `() -> str` | Query + timed read (~300 ms); `""` when unanswered (treat as cancel) |

## `tui.width`

Display widths: 0 controls/combining, 2 wide CJK/emoji blocks, else 1.

| Function | Signature | Notes |
|----------|-----------|-------|
| `width` | `(cp: int) -> int` | Codepoint width (pure) |
| `str_width` | `(s: str) -> int` | Whole-string width (pure) |

## `tui`

| Function | Signature | Notes |
|----------|-----------|-------|
| `version` | `() -> str` | Package version |

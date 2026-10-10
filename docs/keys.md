# Keys

`read_key` yields raw bytes; `tui.keys` turns them into logical keys.
This page is the complete map, probed on a real terminal against
xterm-style emulators.

## Singles (pass through untouched)

| Key | Byte |
|-----|------|
| Ctrl+C | `3` (raw mode passes it through — you own SIGINT handling) |
| Tab | `9` |
| Enter | `13` |
| Esc (lone) | `27` |
| Space | `32` |
| Printable | ASCII (`a` = `97`, `q` = `113`) |
| Backspace | `127` |

## Sequences

| Key | Bytes |
|-----|-------|
| Up / Down / Right / Left | `27, 91, 65` / `66` / `67` / `68` |
| Home (`ESC O H` and `ESC [ H`) | `27, 79, 72` / `27, 91, 72` |
| End (`ESC O F` and `ESC [ F`) | `27, 79, 70` / `27, 91, 70` |
| Delete | `27, 91, 51, 126` |
| Home (`ESC [ 1 ~`) | `27, 91, 49, 126` |
| End (`ESC [ 4 ~`) | `27, 91, 52, 126` |

Decoded codes: arrows `1000`–`1003` (Left, Right, Up, Down),
`1004` Home, `1005` End, `1006` Delete — the same numbering as
`ask`, learn it once. Unknown finals pass through as their own byte.

## The Esc problem (solved)

A leading `27` is ambiguous: lone Esc vs an unfinished sequence.
`read` waits 50 ms (`term.poll`) for the rest — bytes mean sequence,
silence means the user pressed Esc. No blocking, no guessing.

## EOF contract

Closed stdin returns `-1` on the first failed read and every
subsequent one. Callers treat it like Esc (cancel/quit), never retry
against no input. Piped stdin without a TTY should use the numbered
fallbacks (`menu.choose` does this automatically).

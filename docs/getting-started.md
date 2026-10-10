# Getting started

Ten minutes from zero to a working screen app.

## 1. Install

```bash
zz add tui
```

Needs a dev toolchain with `std.term.poll` (the released toolchain
lacks it — build `zz` from source).

## 2. First screen

```zz
import tui.screen
import tui.keys

func main() {
    if !screen.is_tty() {
        println("run me in a terminal")
        return
    }
    screen.enter()
    defer screen.exit()
    screen.clear()
    println("arrows move, q quits")
    live := 1
    while live == 1 {
        k := keys.read()
        if k == 113 {
            live = 0
        } else {
            if k == keys.up() {
                println("up")
            }
        }
    }
}
```

Run it: `zz run src/main.zz`. `defer screen.exit()` restores cooked
mode on every exit path — write that line before anything else.

## 3. Add a menu

```zz
import tui.menu

idx := menu.choose("Pick your hero", ["mage", "rogue", "bard"])
if idx == 0 - 1 {
    println("cancelled")
} else {
    println("picked " + str(idx))
}
```

Arrows, hjkl, Enter to confirm, Esc to cancel (`-1`). Over a pipe
you get a numbered list instead — same return contract.

## 4. Draw a board

```zz
import tui.canvas
import tui.screen

c := canvas.blank(20, 7, " ")
c = canvas.box(c, 0, 0, 20, 7, 1)
c = canvas.text(c, 2, 1, "quest log")
c = canvas.text(c, 2, 3, canvas.bar(0.625, 8))
canvas.present(canvas.render(c))
```

Cells hold display strings (style them with `std.colors` — styles
nest, so overlays re-wrap cleanly). `present` blits top-left without
clearing, so frames don't flicker.

## 5. Test without a terminal

Pure cores first: `keys.decode`, `menu.step`, `textline.insert`,
canvas builders — all covered hermetically in `tests/`. Run yours
the same way:

```bash
zz test
```

Live loops (which block on keys) stay out of unit tests by design;
exercise them through the `examples/` demos or a pty script.

## What's next

- [`api.md`](api.md) for every signature
- [`keys.md`](keys.md) for the full key table
- [`cookbook.md`](cookbook.md) for game loops, dialogs, progress

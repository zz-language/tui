# Cookbook

Copy-paste skeletons for the shapes every screen app needs.

## Game loop

```zz
import tui.screen
import tui.keys
import tui.canvas

func frame(selected: int) -> [str] {
    c := canvas.blank(40, 10, " ")
    c = canvas.box(c, 0, 0, 40, 10, 1)
    c = canvas.text(c, 2, 1, "move with arrows, q quits")
    c = canvas.text(c, 2, 3, "selected: " + str(selected))
    canvas.render(c)
}

func main() {
    if !screen.is_tty() {
        println("needs a terminal")
        return
    }
    screen.enter()
    defer screen.exit()
    sel := 0
    live := 1
    while live == 1 {
        canvas.present(frame(sel))
        k := keys.read()
        if k == 113 {
            live = 0
        } else {
            if k == keys.up() {
                sel = sel - 1
            } else {
                if k == keys.down() {
                    sel = sel + 1
                }
            }
        }
    }
}
```

One `present` per frame, one `flush` inside it. Build the whole frame
before blitting — never draw cell-by-cell to the live screen.

## Quit confirm

```zz
import tui.dialog

if dialog.confirm_box("Quit without saving?") {
    live = 0
}
```

## Progress display

```zz
c = canvas.text(c, 2, 4, canvas.bar(done / total, 30))
```

`bar` clamps, so overshoot never breaks the layout.

## Popup notice

```zz
import tui.dialog

dialog.note(["Saved.", "", "press any key"])
```

## Text prompt on a screen

```zz
import tui.textline

(name, ok) := textline.edit("Name: ", "")
if ok {
    println("hi " + name)
}
```

## Pipe-safe entry

```zz
func main(args: [str]) {
    if len(args) > 0 && args[0] == "demo" {
        // non-interactive path for CI / pipes
    } else {
        // screen app (checks is_tty inside)
    }
}
```

Keep a `--demo`-style path that never touches raw mode: CI, tests,
and curious users without a TTY all stay green.

# lv_drivers (NebulaOS fork)

`NebulaOS-KE/lv_drivers` is a fork of upstream
[`lvgl/lv_drivers`](https://github.com/lvgl/lv_drivers). It exists so that
[NebulaScreen](https://github.com/NebulaOS-KE/NebulaOS-nebulascreen) can carry two small patches
as a submodule; it is **not** a NebulaOS component in its own right.

If you are looking for NebulaOS's display and input code, most of it is not here -
see [Where the real display code lives](Where-The-Real-Display-Code-Lives).

## The short answer: two commits

`master` is upstream, untouched: `ea2e4cc`, *"Update README.md"* by LVGL's maintainer. Nothing
has been committed to it here.

The branch NebulaScreen uses is **`guppyscreen-ke-next`**, which is `master` plus exactly two
commits:

| Commit | What it changes | Used by the shipped app? |
|---|---|---|
| `8326182` | `display/fbdev.c`, `display/fbdev.h` - adds `fbdev_blank()` and `fbdev_unblank()`, two `FBIOBLANK` wrappers | **No** - superseded, see [The framebuffer helpers](The-Framebuffer-Helpers) |
| `0406ccb` | `indev/evdev.c` - accept any non-negative `ABS_MT_TRACKING_ID` as a press | **Yes** |

NebulaScreen pins `0406ccb` as its `lv_drivers` submodule - checked at `fd47cb6`, the screen
NebulaOS 1.0.2 ships, and at `84db432`, the current tip of NebulaScreen's `main`: both the
`.gitmodules` entry and the submodule object. That is the tip of `guppyscreen-ke-next`, so the
whole fork surface is those twelve-plus-five added lines.

## The pages

- **[The touch fix](The-Touch-Fix)** - `0406ccb`, which is live: every tap on the printer goes
  through it
- **[The framebuffer helpers](The-Framebuffer-Helpers)** - `8326182`, which is not: dead code,
  and one of the two helpers must never be called on this panel at all
- **[Where the real display code lives](Where-The-Real-Display-Code-Lives)** - in NebulaScreen,
  and why
- **[Changing something here](Changing-Something-Here)** - which branch, the submodule pin, and
  why a patch here costs more than the same patch in NebulaScreen

## Checked against NebulaOS 1.0.2 (2026-11-11, re-checked 2026-11-12)

Nothing here moved for the release, and that is the point of the page: `master` is still
`ea2e4cc`, `guppyscreen-ke-next` is still `0406ccb`, and the fork surface is still those two
commits (12 added lines in `display/fbdev.c`/`.h`, 5 added and 1 removed in `indev/evdev.c`).

What changed is which NebulaScreen pins it. NebulaOS 1.0.2 ships NebulaScreen `fd47cb6`
(tag `v1.0.1`, `NEBULASCREEN_PIN` in firmware `3bdf3ff`), and NebulaScreen's `main` has since
moved to `84db432`. **Both pin the same submodule commit, `0406ccb`** - so the shipped release
and the tip of the consumer agree, and this page describes both.

The three claims that would age badly were re-read at `fd47cb6` rather than assumed:

- `evdev_init()` and `evdev_read` are still called from `src/main.cpp` (`hal_init`, lines
  150-167), so every tap still goes through this fork's `indev/evdev.c`.
- `src/nebulascreen_fb.cpp` / `.h` are still the framebuffer output - `main.cpp`'s `nsfb_flush`
  is the LVGL `flush_cb`, and `fbdev_flush` is not used.
- The *"never `fbdev_blank()`"* rule is still written where the display-sleep code is, in
  `src/guppyscreen.cpp:269`: the panel cannot recover from `FB_BLANK_POWERDOWN`, so NebulaOS
  switches the backlight off and the black overlay is what shows.

## Verification state

| | |
|---|---|
| Fork surface | `master..guppyscreen-ke-next` = 2 commits, read at `0406ccb` |
| Upstream base | `ea2e4cc`, unmodified on `master` |
| Consumed by | NebulaScreen `fd47cb6` (shipped by NebulaOS 1.0.2, firmware `3bdf3ff`) and `84db432` (tip of `main`) - both pin `0406ccb` |
| `evdev` in use | **yes** - `evdev_init` / `evdev_read` in NebulaScreen `src/main.cpp`, wrapped by `evdev_read_smoothed` |
| `fbdev` in use | **no** - only comments reference it; `src/nebulascreen_fb.*` replaced it |
| Re-verified | 2026-11-11, against NebulaScreen `fd47cb6` and `84db432`; branch tips and the submodule object re-read 2026-11-12, all unchanged |
| Tests | none in this repository |
| Hardware verification | **none** - no printer access. The touch and panel behaviour described here is read from source and from NebulaScreen's own comments, not observed |

## Related

- [NebulaScreen wiki](https://github.com/NebulaOS-KE/NebulaOS-nebulascreen/wiki) - the consumer
- [Repository map](https://github.com/NebulaOS-KE/NebulaOS-firmware/wiki/Repository-Map)

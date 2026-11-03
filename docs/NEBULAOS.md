# This fork, and which parts of it are NebulaOS

`NebulaOS-KE/lv_drivers` is a fork of upstream [`lvgl/lv_drivers`](https://github.com/lvgl/lv_drivers).
It exists so that NebulaScreen can carry two small patches as a submodule; it is **not** a
NebulaOS component in its own right.

If you are looking for NebulaOS's display and input code, most of it is not here - see
*Where the real display code lives* below.

## The short answer: two commits

`master` is upstream, untouched: `ea2e4cc`, *"Update README.md"* by LVGL's maintainer. Nothing
has been committed to it here.

The branch NebulaScreen uses is **`guppyscreen-ke-next`**, which is `master` plus exactly two
commits:

| Commit | What it changes | Used by the shipped app? |
|---|---|---|
| `8326182` | `display/fbdev.c`, `display/fbdev.h` - adds `fbdev_blank()` and `fbdev_unblank()`, two `FBIOBLANK` wrappers | **No** - superseded, see below |
| `0406ccb` | `indev/evdev.c` - accept any non-negative `ABS_MT_TRACKING_ID` as a press | **Yes** |

NebulaScreen pins `0406ccb` as its `lv_drivers` submodule (NebulaScreen `36490ed`,
`.gitmodules` plus the submodule entry). That is the tip of `guppyscreen-ke-next`, so the whole
fork surface is those twelve-plus-five added lines.

## `0406ccb` - the touch fix, which is live

Upstream's evdev reader treated **only** tracking id `0` as an active contact:

```c
if(in.value == -1)      evdev_button = LV_INDEV_STATE_REL;
else if(in.value == 0)  evdev_button = LV_INDEV_STATE_PR;
```

A multitouch controller is free to assign any non-negative id, and many assign increasing ones.
Every contact after the first then produced no press at all - the touch simply did not register.
The fix treats `-1` as "lifted" and anything else as "pressed", which is what the kernel's
multitouch protocol actually specifies.

This one matters: `evdev_init()` and `evdev_read` are called from NebulaScreen's `main.cpp`, so
the printer's touchscreen goes through this code on every tap.

## `8326182` - the framebuffer helpers, which are not

The two helpers were added when NebulaScreen still drew through lv_drivers' fbdev driver. It no
longer does, and neither helper is called by the shipped application.

**`fbdev_blank()` must not be called on this panel at all.** NebulaScreen's own source says so
where the display-sleep code would otherwise use it: the KE's panel cannot recover from
`FB_BLANK_POWERDOWN`. Display sleep switches the backlight off through NebulaOS instead and
shows a black overlay.

**`fbdev_unblank()` was replaced for a subtler reason.** On the KE's Ingenic DPU,
`FBIOBLANK(FB_BLANK_UNBLANK)` is not a no-op for a panel that is already on: `ingenicfb_blank()`
calls `dpu_ctrl_resume()`, which stops the compositor, re-runs the controller setup and restarts
it, with no "already unblanked" check. Because lv_drivers' `fbdev_init()` issues that ioctl at
startup, the panel went black at that moment and stayed black until the app had built all its
pages and flushed a first frame - about half a second on the X2000, visible as a gap between the
boot card and the app.

So this fork's fbdev changes are **historical**: correct when written, and now dead code kept
only because removing them would mean rewriting the submodule pin for no benefit.

## Where the real display code lives

In NebulaScreen, not here: `src/nebulascreen_fb.cpp` / `.h`, which replace `fbdev_init()` and
`fbdev_flush()` on the printer. They map the framebuffer **without** any blank ioctl and without
clearing it, so the boot card stays on the glass while the app builds its pages; the first frame
is rendered straight over it, and only then is `unblank()` called once, for the rare case that
something else left the display blanked.

That code is LVGL-free and host-tested, which is why it could be moved out of the driver in the
first place.

## If you need to change something here

- **Change it on `guppyscreen-ke-next`**, not on `master`. `master` is kept equal to upstream on
  purpose, so the fork's whole surface stays readable as `master..guppyscreen-ke-next`. Two
  commits is a feature.
- **Then move NebulaScreen's submodule pin**, or nothing ships. The pin is the shipping
  authority here, exactly as elsewhere in NebulaOS.
- **Prefer NebulaScreen over this fork.** Anything that can live in `src/` and be host-tested
  should, as the framebuffer work shows: a patch here is invisible to NebulaScreen's test suite
  and travels only with a submodule bump.
- **Upstream has moved on.** The drivers were merged into LVGL itself in v9, so this repository
  is a maintenance fork of something upstream no longer develops separately. Do not expect to
  rebase onto an active branch.

## Verification state

| | |
|---|---|
| Fork surface | `master..guppyscreen-ke-next` = 2 commits, read at `0406ccb` |
| Upstream base | `ea2e4cc`, unmodified on `master` |
| Consumed by | NebulaScreen `36490ed`, submodule pinned at `0406ccb` |
| `evdev` in use | **yes** - `evdev_init` / `evdev_read` in NebulaScreen `src/main.cpp` |
| `fbdev` in use | **no** - only comments reference it; `src/nebulascreen_fb.*` replaced it |
| Tests | none in this repository |
| Hardware verification | **none** - no printer access. The touch and panel behaviour described here is read from source and from NebulaScreen's own comments, not observed |

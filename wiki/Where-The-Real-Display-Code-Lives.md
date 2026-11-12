# Where the real display code lives

Not here. Written against NebulaScreen `fd47cb6`, the screen NebulaOS 1.0.2 ships.

In NebulaScreen, not here: `src/nebulascreen_fb.cpp` / `.h`, which replace `fbdev_init()` and
`fbdev_flush()` on the printer. They map the framebuffer **without** any blank ioctl and without
clearing it, so the boot card stays on the glass while the app builds its pages; the first frame
is rendered straight over it, and only then is `unblank()` called once, for the rare case that
something else left the display blanked.

That code is LVGL-free and host-tested, which is why it could be moved out of the driver in the
first place.

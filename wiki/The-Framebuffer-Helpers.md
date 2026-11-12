# The framebuffer helpers: `8326182`

Dead code, kept deliberately - and one of the two helpers would damage the user's experience if
it were called. Re-read at NebulaScreen `fd47cb6` and `84db432`.

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

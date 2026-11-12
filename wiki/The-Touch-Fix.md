# The touch fix: `0406ccb`

The one commit in this fork that the shipped application actually runs. Re-read at NebulaScreen
`fd47cb6` (NebulaOS 1.0.2) and `84db432`.

Upstream's evdev reader treated **only** tracking id `0` as an active contact:

```c
if(in.value == -1)      evdev_button = LV_INDEV_STATE_REL;
else if(in.value == 0)  evdev_button = LV_INDEV_STATE_PR;
```

A multitouch controller is free to assign any non-negative id, and many assign increasing ones.
Every contact after the first then produced no press at all - the touch simply did not register.
The fix treats `-1` as "lifted" and anything else as "pressed", which is what the kernel's
multitouch protocol actually specifies.

This one matters: `evdev_init()` and `evdev_read` are called from NebulaScreen's `main.cpp`
(`hal_init`), so the printer's touchscreen goes through this code on every tap.

One nuance, so the reader is not surprised by the source: NebulaScreen does not register
`evdev_read` directly. It registers `evdev_read_smoothed`, which calls whichever reader is active
- raw `evdev_read`, or `lv_tc_indev_drv_init(&indev_drv_1, evdev_read)` when touch calibration is
on - and then eases the reported point with an exponential filter. The press/release state this
fork decides is passed through untouched; only the coordinates are filtered.

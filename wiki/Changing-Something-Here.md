# Changing something here

Four rules, and one of them is "prefer not to".

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

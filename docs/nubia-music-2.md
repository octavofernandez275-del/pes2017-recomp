# ZTE nubia Music 2 (Unisoc T7200, Mali-G57 MP1, 4 GB)

Not measured on the phone. The values are starting points chosen from the hardware.

| | |
|---|---|
| SoC | Unisoc T7200, 2x Cortex-A75 + 6x Cortex-A55 (12 nm) |
| GPU | Mali-G57 MP1: one shader core, no BC sampling, ASTC yes |
| Memory | 4 GB (about 3.6 GB seen by the app), eMMC 5.1 |
| Screen | 720x1612, 120 Hz |

## What the profile sets (`DeviceProfile.applyNubiaMusic2`)

Applied when the phone is recognised (model, `Z2460`, or SoC `T7200`) and the Low preset is chosen
or detected. Choosing Medium or High by hand gives those back.

- Scene at 56% (9/16) to start, falling to 38% (6/16, 480x270) under the load or heat rule
  (`PES17_SCENE_FLOOR=6`); FSR on; no multisampling; no crowd.
- 30 fps limit; heat and GPU-load rules on.
- Low-memory mode (no 512 MB vertex shadow), texture cache 256 MB, BC textures re-encoded to ASTC
  (`PES17_BC_ASTC`, a prototype: switch `astcTextures` off if textures look wrong).

## Runtime changes

- CPU topology (`platform.cpp`, `android/perf.cpp`): when every core reports the same top clock,
  the scheduler's `cpu_capacity` separates the A75s from the A55s, so the main thread is pinned to
  the fast cores. With two fast cores the other guest threads may use the little ones as well.
- `PES17_SCENE_FLOOR` (renderer): scene floor below one half; `tests/dynamic_scale_test.cpp` covers it.

## To tune once it runs

- HUD level Complete: fps, frame time, temperature. Share the log (Log, REPORT A BUG).
- `files/yield_tuning` `30 32`: sleeps the guest's spinning threads (cost 6 fps on an SD865).
- Texture budget 192 if the app is killed for memory.

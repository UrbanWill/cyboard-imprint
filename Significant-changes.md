# Significant Changes

## Force-Awake for Both Trackballs

**Date:** 2026-05-22
**Files changed:**

- `config/imprint_left.overlay` — created (adds `force-awake` to left trackball sensor)
- `config/imprint_right.overlay` — created (adds `force-awake` to right trackball sensor)

**Problem:** Both trackballs (especially the left scroll ball) required vigorous shaking to "wake up" after brief periods of inactivity. The left scroll ball sometimes couldn't self-wake at all.

**Root cause:** The PMW3610 driver (Cyboard-DigitalTailor/zmk-pmw3610-driver) defaults to aggressive power-saving with short downshift times. Without the `force-awake` device tree property, the driver has no mechanism to keep sensors responsive based on ZMK's activity state.

**Solution:** Added `force-awake;` to both trackball sensor nodes via per-shield DTS overlays in the config directory. Zephyr auto-includes `<shield>.overlay` from the config directory when building each shield. This keeps sensors in Run mode while ZMK is ACTIVE and allows normal rest modes when ZMK goes IDLE/SLEEP.

**To revert:**

1. Delete `config/imprint_left.overlay`
2. Delete `config/imprint_right.overlay`

---

## Increased Idle Timeout to 2 Minutes

**Date:** 2026-05-22
**Files changed:**

- `config/imprint.conf` — added `CONFIG_ZMK_IDLE_TIMEOUT=120000`

**Problem:** With the default 30-second idle timeout, sensors would enter rest modes quickly after stopping all input (e.g., while reading). Combined with `force-awake`, increasing the timeout extends the window where trackballs remain responsive after the last keypress/movement.

**Solution:** Increased `CONFIG_ZMK_IDLE_TIMEOUT` from 30000ms (30s) to 120000ms (2 minutes).

**Battery impact:** Estimated <10% reduction in battery life. Sensors draw ~3mA extra during the additional 90 seconds of ACTIVE state per idle cycle.

**To revert:**
Remove `CONFIG_ZMK_IDLE_TIMEOUT=120000` from `config/imprint.conf` (reverts to default 30s).

---

## Trackball Sensitivity Adjustments

**Date:** 2026-05-20
**Files changed:**

- `config/imprint.keymap` — modified `trackball_peripheral_listener` and `trackball_central_listener`

**Changes:**

- Right pointer trackball: added `&zip_xy_scaler 3 4` (75% speed)
- Left scroll ball: `&zip_xy_scaler 1 16` (pre-mapper) + `&zip_scroll_scaler 1 4` (post-mapper) for 64x total scroll reduction
- Both trackballs: added `&zip_temp_layer 2 500` to activate mouse layer on use

**To revert:**
Restore `trackball_peripheral_listener` and `trackball_central_listener` to their previous commented-out/default state in the keymap.

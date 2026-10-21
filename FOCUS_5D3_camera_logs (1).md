# 5D3 camera logs of the focus modules
## First 5D3 camera data (CAMF6C9E.zip, 16 Oct 2026 02:20-02:23)
- **The modules ran:** the old builds (CAMF C1 and V17C r3 from ML_FOCUS_HANDOVER_5D3.zip) loaded and drove the lens on the 5D3.
  - Picture buffer 720x480; frame_ms 33 (one search 40).
  - Lens reports id 36912 (the 18-135's id) at 29 mm.
- **Logs:** 15 CAMF searches (CAMF6C9E-6CAC) and 10 V17C runs (FZ7C71B3-71BC); CAMGAIN.CSV was in the same zip.
- **CAMF was ON during the V17C runs, so the two modules fought over the lens.** Card timestamps interleave a CAMF search ~2 s before each V17C log. The V17C results are therefore not usable:
  - 3 runs ended with stop 4 (step budget) after 17-22 s;
  - 4 "accepted" runs ended LESS sharp than they started (e.g. 71BC 159k → 12k; 71B9 333k → 109k).
- **The scene changed during the runs:** luma moved 37 → 203 within a run, and sharpness noise was 1-10%. This is consistent with handheld pointing.
- **CAMF results:** 7 accepted, 1 at peak, 4 no gain (went home), 3 image failures.
  - The image failures were searches 5, 6 and 15, with 22-45 invalid frames each.
  - Dark pixels in V17C samples at the same time reached 62% (above 50%). That matches the CAMF C1 dark-frame bug, already fixed in C2.
- **Next test:**
  - Install ML_FOCUS_PROFILES_REPLAY.zip.
  - CAMCAL at 29 mm on a tripod first.
  - V17C with CAMF OFF.
  - CAMF tested alone.
- **Idea:** a cross-module lock so V17C/CAMCAL and CAMF can't run on the lens together.

## Session 2 (CAMF232B.zip, 16 Oct 02:40-02:43) + half-shutter workflow
- **Builds:** CAMF C2 and V17C r4 loaded on the 5D3. This proves the FIO_OpenFile/FIO_ReadFile imports work. No lens profile existed yet (profile 0).
- **Runs:**
  - 16 CAMF searches;
  - 9 V17C runs at 24 mm: 7 accepted, 2 stop 5.
- **CAMF was again ON during the menu-triggered V17C runs.** Several "accepted" runs ended less sharp than they started (e.g. 325k → 164k).
- **The user's requirement:** half-press starts V17C; CAMF pauses while V17C runs; CAMF resumes when V17C stops or the half-press is released.
- **V17C r5:**
  - New menu "V17C Trigger": MENU ONLY / HALF-SHUTTER, default HALF-SHUTTER.
  - Rising edge of `get_halfshutter_pressed()` → one run, started after a 150 ms delay.
  - Release → gate bit 64 → stop at once, no restore.
  - Canon AF is moved to * via `assign_af_button_to_star_button()` and restored when the mode goes off or the module unloads.
- **CAMF C3:**
  - While the half-press is held: no commands; a search in progress is cut short (result 7).
  - 1.2 s after release: re-baseline without searching, then back to WATCH.
- **Sim `halfpress`:** 4/4 runs had 0 commands while held and ended in focus.
- **Imports added:** get_halfshutter_pressed, assign_af_button_to_star_button, restore_af_button_assignment (all used by the stock dot_tune module).
- **Package:** ML_FOCUS_5D3_HALFSHUTTER.zip.
  - 5D3 sha prefixes: CAMCAL 30b7c744, FZ17C 42c3a690, CAMF 78a83702.
- **Not covered:** CAMF can't see a menu-triggered V17C run or CAMCAL; CAMF must be OFF for those.

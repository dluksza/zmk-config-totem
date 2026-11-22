# Home Row Mod Tuning History

## Overview

This document tracks the tuning history and rationale for home row mod configurations on the TOTEM keyboard, particularly for the shift keys (A and ;).

## Key Position Reference

```
KEYS_L: 4 3 2 1 0 | 14 13 12 11 10 | 25 24 23 22 21 20   (left hand)
KEYS_R: 5 6 7 8 9 | 15 16 17 18 19 | 26 27 28 29 30 31   (right hand)
THUMBS: 32 33 34 35 36 37
```

## Home Row Mod Layout

| Finger | Left Key | Left Mod | Right Key | Right Mod |
|--------|----------|----------|-----------|-----------|
| Pinky  | A        | LSHIFT   | ;         | RSHIFT    |
| Ring   | S        | LCTRL    | O         | RCTRL     |
| Middle | D        | LALT     | I         | RALT      |
| Index  | F        | LGUI     | U         | RGUI      |

## Parameter Reference

| Parameter | Description |
|-----------|-------------|
| `tapping-term-ms` | How long key must be held before mod activates |
| `quick-tap-ms` | If pressed again within this time, bypasses hold logic (enables fast double-tap) |
| `require-prior-idle-ms` | Key must be idle this long before hold can activate |
| `hold-trigger-key-positions` | Hold only activates if these keys are pressed during hold window |
| `hold-trigger-on-release` | Wait for key release before deciding tap vs hold |
| `flavor` | `balanced` (waits for motion) or `tap-preferred` (defaults to tap) |

## Left Shift (hrmL_pinky) Tuning History

### Git Commit History

| Commit | tapping-term | quick-tap | prior-idle | flavor | hold-on-release | Notes |
|--------|-------------|-----------|------------|--------|-----------------|-------|
| Original | 220 (macro) | 310 (macro) | 160 (macro) | tap-preferred | yes | Used global macros |
| 2b6eab3 | **200** | macro | macro | **balanced** | yes | Changed flavor, reduced tap term |
| 2ebaf69 | **280** | **350** | **180** | balanced | yes | Increased to fix key repeat |
| 32d7496 | **250** | 350 | **240** | balanced | yes | Adjusted prior-idle |
| b44dab4 | 250 | **250** | **125** | balanced | **no** | Removed hold-on-release |
| 0f77542 | **200** | **175** | **none** | **tap-preferred** | no | Removed prior-idle entirely |
| 1c03165 | 200 | 175 | none | tap-preferred | no | Current - applied same config to right shift |

### Problems Encountered

1. **"ai" instead of "I", "at" instead of "T"**
   - Cause: `require-prior-idle-ms` too high (240ms), blocking shift during fast typing
   - Fix: Reduced to 125ms

2. **"aaae" instead of "E"**
   - Cause: Key repeat before shift activates due to high `quick-tap-ms` (350ms)
   - Fix: Reduced to 250ms

3. **Keystroke "forgotten" - hold A, press letter, nothing happens**
   - Cause: `hold-trigger-on-release` delays shift until A is released
   - Fix: Removed `hold-trigger-on-release`

4. **";;;;;;;s" instead of "S"**
   - Cause: Right shift had old conservative config while trying to use opposite-hand shift
   - Fix: Applied same aggressive config to right shift (commit 1c03165)

5. **"ag" instead of "G"**
   - Cause: G is on left hand, left shift only triggers with right-hand keys
   - Fix: Not a bug - use right shift for left-hand capitals (ergonomic standard)

### Values That Failed

- `tapping-term-ms = 200` with `balanced` flavor (commit 2b6eab3) - caused issues requiring increase to 280ms
- `require-prior-idle-ms = 180` (commit 2ebaf69) - was later increased to 240ms
- `require-prior-idle-ms = 125` (commit b44dab4) - still blocked shift during fast typing

### Current Configuration (1c03165)

Both shift keys now have identical aggressive config:

```dts
hrmL_pinky: home_row_mod_left_pinky {
    compatible = "zmk,behavior-hold-tap";
    flavor = "tap-preferred";
    hold-trigger-key-positions = <KEYS_R THUMBS>;
    tapping-term-ms = <200>;
    quick-tap-ms = <175>;
    #binding-cells = <2>;
    bindings = <&kp>, <&kp>;
};

hrmR_pinky: home_row_mod_right_pinky {
    compatible = "zmk,behavior-hold-tap";
    flavor = "tap-preferred";
    hold-trigger-key-positions = <KEYS_L THUMBS>;
    tapping-term-ms = <200>;
    quick-tap-ms = <175>;
    #binding-cells = <2>;
    bindings = <&kp>, <&kp>;
};
```

### Trade-offs of Current Config

- **No prior-idle requirement**: Relies solely on `hold-trigger-key-positions` to prevent false activations
- **tap-preferred flavor**: Defaults to tap unless opposite-hand key is pressed during hold window
- **Shorter timings**: More responsive but tighter windows
- **Cross-hand only**: Left shift works with right-hand keys only (and vice versa)

### Required Typing Pattern

**This config enforces opposite-hand shift usage:**
- Left shift (A) → right-hand letters (Y, U, I, O, P, H, J, K, L, N, M)
- Right shift (;) → left-hand letters (Q, W, E, R, T, G, A, S, D, F, Z, X, C, V, B)

This is the ergonomic standard and eliminates same-hand timing conflicts.

## Other Finger Configurations

### Left Ring (LCTRL on S)
- `flavor = "tap-preferred"`
- `tapping-term-ms = 260`
- `quick-tap-ms = 300`
- `require-prior-idle-ms = 160` (macro)

### Left Middle (LALT on D)
- `flavor = "tap-preferred"`
- `tapping-term-ms = 220` (macro)
- `quick-tap-ms = 225`
- `require-prior-idle-ms = 160` (macro)

### Left Index (LGUI on F)
- `flavor = "tap-preferred"`
- `tapping-term-ms = 220` (macro)
- `quick-tap-ms = 310` (macro)
- `require-prior-idle-ms = 160` (macro)

### Right Ring (RCTRL on O)
- `flavor = "balanced"`
- `tapping-term-ms = 280`
- `quick-tap-ms = 300`
- `require-prior-idle-ms = 180`
- Has `hold-trigger-on-release`
- Note: Was specifically tuned in commit 698958d to fix false Ctrl activation

### Right Pinky (RSHIFT on ;)
- Now matches left pinky config (see Current Configuration above)
- Previous conservative config caused ";;;;;;;s" key repeat issue

## Layout Asymmetry Note

The current layout has an asymmetry:
- **Left hand**: Mods on HOME row (A=Shift, S=Ctrl, D=Alt, F=GUI)
- **Right hand**: Mods on TOP row (U=GUI, I=Alt, O=Ctrl) + Shift on home row (;)

User observation: Left Ctrl/Alt/GUI (S, D, F) are rarely used - prefer left thumb GUI and right top-row mods. Consider simplifying left hand to shift-only if this continues.

## Design Principles

1. **Asymmetric timing**: Different fingers have different speeds
   - Pinkies: slower, need longer windows
   - Index: fastest, can use shorter windows

2. **Flavor strategy**:
   - `balanced` for shift/ctrl (pinkies/ring) - better for holding modifiers
   - `tap-preferred` for alt/gui (middle/index) - faster typing response

3. **Cross-hand hold-trigger**: Left mods only activate when right-hand keys pressed (and vice versa) to prevent false activations during same-hand typing

## Future Tuning Notes

If issues persist with current config (1c03165):

1. **False shift activations on "as"/"an"**: Add back `require-prior-idle-ms = <100>` as minimal guard
2. **Shift still missing**: Current config is most aggressive possible - if still failing, issue may be hardware or fundamental ZMK limitation
3. **Key repeat returns**: Increase `quick-tap-ms` to 200ms
4. **Accidental "A" or ";" taps when holding for shift**: Increase `tapping-term-ms` to 220ms

## Alternative Layout Options Considered

### Option 1: Thumb Shift (Recommended if HRM fails)
Replace DEL with `&mt LSHIFT DEL` (hold=shift, tap=del) or dedicated shift.
- Most reliable solution
- Thumb is strong, no timing ambiguity

### Option 2: Move Left Mods to Top Row
```
Top row:    Q    ⌘/W   ⌥/E   ^/R    T     |  Y    ⌘/U   ⌥/I   ^/O    P
Home row:  ⇧/A    S     D     F     G     |  H     J     K     L    ⇧/;
```
- Symmetric with right hand
- Physical movement creates natural delay
- Shift still on home row (main problem persists)

### Option 3: All Mods on Top Row
```
Top row:   ⇧/Q   ⌘/W   ⌥/E   ^/R    T     |  Y    ⌘/U   ⌥/I   ^/O   ⇧/P
Home row:   A     S     D     F     G     |  H     J     K     L     ;
```
- Home row completely free for fast typing
- Q/P are infrequent letters (0.1% and 1.9%)
- Eliminates HRM timing issues entirely

### Option 4: One-Shot Shift
```dts
&sk LSHIFT  // tap once, next key is shifted
```
- Eliminates all timing issues
- Different paradigm - requires mental adjustment
- Multiple capitals need multiple taps

## Key Insight

**Perfect home-row shift is likely unachievable** without adapting typing style. The fundamental conflict:
- Same finger position for common letters AND modifier activation
- No amount of timing tuning fully resolves this trade-off

**Top-row mods work better** because physical finger movement creates natural ~50-100ms delay that separates "typing" from "modifying".

**Current solution:** Aggressive config + opposite-hand shift discipline. This is the best balance of reliability and minimal retraining.

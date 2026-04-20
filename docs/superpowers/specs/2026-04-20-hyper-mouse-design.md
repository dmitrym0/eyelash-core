# Hyper Layer Mouse Movement Design

## Overview

Add mouse movement and clicking to the HYPER layer using vim-style movement keys (HJKL), with a fast-mouse sublayer activated by holding Space.

## Behaviors

Add a `mmv_fast` behavior node with aggressive acceleration alongside the existing `&mmv`:

```dts
&mmv_fast {
    time-to-max-speed-ms = <100>;
    acceleration-exponent = <1>;
    trigger-period-ms = <16>;
};
```

The existing `&mmv` (500ms to max speed) remains unchanged for normal movement on other layers.

## Hyper Layer Changes (layer 4)

Add to the existing `hyper_layer`:

| Key | Binding | Note |
|-----|---------|------|
| H | `mmv MOVE_LEFT` | vim left |
| J | `mmv MOVE_DOWN` | vim down |
| K | `mmv MOVE_UP` | vim up |
| L | `mmv MOVE_RIGHT` | vim right |
| U | `mkp LCLK` | left click |
| I | `mkp MCLK` | middle click |
| O | `mkp RCLK` | right click |
| Space (thumb) | `mo 6` | activate fast-mouse layer |

Space is currently `&trans` on the hyper layer, so it is free to use.

## New mouse_fast Layer (layer 6)

Activated only while Space is held within hyper mode. Mirrors hyper layer mouse bindings but uses `mmv_fast` for snappier movement. All other keys are `&trans`.

| Key | Binding |
|-----|---------|
| H | `mmv_fast MOVE_LEFT` |
| J | `mmv_fast MOVE_DOWN` |
| K | `mmv_fast MOVE_UP` |
| L | `mmv_fast MOVE_RIGHT` |
| U | `mkp LCLK` |
| I | `mkp MCLK` |
| O | `mkp RCLK` |
| Space | `&trans` (already held) |

## Layer Stack

```
default → hyper (hold Esc) → mouse_fast (hold Space)
```

## Key Decisions

- `mmv_fast` is a separate behavior node, not a macro — idiomatic ZMK, no runtime hacks
- Clicks (UIO) are the same on both layers — speed is irrelevant for clicking
- `ZMK_POINTING_DEFAULT_MOVE_VAL` and `ZMK_POINTING_DEFAULT_SCRL_VAL` are unchanged

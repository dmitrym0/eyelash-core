# Hyper Mouse Movement Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add vim-style mouse movement (HJKL) and clicks (UIO) to the HYPER layer, with a Space-held fast-mouse sublayer.

**Architecture:** Add a `mmv_fast` behavior node with aggressive acceleration. Modify the existing `hyper_layer` to wire HJKL→move, UIO→click, Space→`mo 6`. Add a new `mouse_fast` layer (layer 6) that mirrors those bindings but uses `mmv_fast`.

**Tech Stack:** ZMK firmware, devicetree keymap syntax (`.keymap` file), no build tooling needed locally — GitHub Actions builds on push.

---

## File Map

- Modify: `config/eyelash_corne.keymap`
  - Add `mmv_fast` behavior block
  - Update `hyper_layer` bindings
  - Add new `mouse_fast` layer

---

### Task 1: Add `mmv_fast` behavior node

**Files:**
- Modify: `config/eyelash_corne.keymap`

The existing `&mmv` block is at lines 26-30. Add `&mmv_fast` immediately after it, before `&soft_off`.

- [ ] **Step 1: Open `config/eyelash_corne.keymap` and locate the `&mmv` block**

It looks like this (lines 26-30):
```
&mmv {
    time-to-max-speed-ms = <500>;
    acceleration-exponent = <1>;
    trigger-period-ms = <16>;
};
```

- [ ] **Step 2: Add `mmv_fast` block immediately after `&mmv`**

Insert after the closing `};` of `&mmv` and before `&soft_off { hold-time-ms = <2000>; };`:

```
&mmv_fast {
    time-to-max-speed-ms = <100>;
    acceleration-exponent = <1>;
    trigger-period-ms = <16>;
};
```

- [ ] **Step 3: Commit**

```bash
git add config/eyelash_corne.keymap
git commit -m "feat: add mmv_fast behavior node for hyper mouse acceleration"
```

---

### Task 2: Update `hyper_layer` bindings

**Files:**
- Modify: `config/eyelash_corne.keymap` (hyper_layer, currently lines 163-173)

The hyper layer currently has all `&trans` on the right-hand side except `&ext_power EP_ON` on the home row. The layout is (right half, row by row):

```
Row 0 (top):    &kp HYPER(N6)  &kp HYPER(N7)  &kp HYPER(N8)  &kp HYPER(N9)  &kp HYPER(N0)  &trans
Row 1 (home):   &ext_power EP_ON  &trans  &trans  &trans  &trans  &trans
Row 2 (bottom): &trans  &trans  &trans  &trans  &trans  &trans
Thumb row:      &trans  &trans  &trans
```

Key positions on the right hand (right half of each row, after the 5-way joystick):
- **U** = row 0, position 2 from left on right half (after Y)
- **I** = row 0, position 3
- **O** = row 0, position 4
- **H** = row 1, position 1 (leftmost on right half, this is where `&ext_power EP_ON` currently sits)
- **J** = row 1, position 2
- **K** = row 1, position 3
- **L** = row 1, position 4
- **Space thumb** = left thumb row, position 3 (rightmost left thumb key, currently `&trans`)

- [ ] **Step 1: Replace the `hyper_layer` bindings block**

Find this block (lines 165-170):
```
        hyper_layer {
            display-name = "HYPER";
            bindings = <
&trans  &kp LS(LA(LG(LC(N1))))  &kp LS(LA(LG(LC(N2))))  &kp LS(LA(LG(LC(N3))))  &kp LS(LA(LG(LC(N4))))  &kp LS(LA(LG(LC(N5))))                    &trans          &kp LS(LA(LG(LC(N6))))  &kp LS(LA(LG(LC(N7))))  &kp LS(LA(LG(LC(N8))))  &kp LS(LA(LG(LC(N9))))  &kp LS(LA(LG(LC(N0))))  &trans
&trans  &trans                  &trans                  &trans                  &trans                  &kp LS(LA(LG(LC(G))))             &trans  &trans  &trans  &ext_power EP_ON        &trans                  &trans                  &trans                  &trans                  &trans
&trans  &trans                  &trans                  &trans                  &trans                  &trans                  &trans            &trans          &trans                  &trans                  &trans                  &trans                  &trans                  &trans
                                                        &trans                  &trans                  &trans                                                    &trans                  &trans                  &trans
            >;
```

Replace with:
```
        hyper_layer {
            display-name = "HYPER";
            bindings = <
&trans  &kp LS(LA(LG(LC(N1))))  &kp LS(LA(LG(LC(N2))))  &kp LS(LA(LG(LC(N3))))  &kp LS(LA(LG(LC(N4))))  &kp LS(LA(LG(LC(N5))))                    &trans          &kp LS(LA(LG(LC(N6))))  &mkp LCLK               &mkp MCLK               &mkp RCLK               &kp LS(LA(LG(LC(N0))))  &trans
&trans  &trans                  &trans                  &trans                  &trans                  &kp LS(LA(LG(LC(G))))             &trans  &trans  &trans  &mmv MOVE_LEFT          &mmv MOVE_DOWN          &mmv MOVE_UP            &mmv MOVE_RIGHT         &trans                  &trans
&trans  &trans                  &trans                  &trans                  &trans                  &trans                  &trans            &trans          &trans                  &trans                  &trans                  &trans                  &trans                  &trans
                                                        &trans                  &trans                  &mo 6                                                     &trans                  &trans                  &trans
            >;
```

Note: `&ext_power EP_ON` moves off H — it is displaced by `&mmv MOVE_LEFT`. If you want to keep it, relocate it to another unused key (e.g. row 2 right side). For now it is removed.

- [ ] **Step 2: Commit**

```bash
git add config/eyelash_corne.keymap
git commit -m "feat: add mouse movement and clicks to hyper layer"
```

---

### Task 3: Add `mouse_fast` layer (layer 6 — becomes layer 7)

**Files:**
- Modify: `config/eyelash_corne.keymap`

The current layer order is:
- 0: default
- 1: lower (NUMBER)
- 2: raise (SYMBOL)
- 3: layer_3 (Fn)
- 4: hyper_layer (HYPER)
- 5: hyper_letter_layer (HYPER-L)

The new `mouse_fast` layer will be **layer 6**, appended after `hyper_letter_layer`. The `mo 6` binding added in Task 2 references this layer by index — it must be the 7th layer definition (0-indexed = 6).

- [ ] **Step 1: Append `mouse_fast` layer after `hyper_letter_layer`**

Find the closing of `hyper_letter_layer` (the `};` before the closing `};` of `keymap {`):

```
        hyper_letter_layer {
            display-name = "HYPER-L";
            bindings = <
...
            >;

            sensor-bindings = <&scroll_encoder>;
        };
    };
};
```

Insert the new layer between `hyper_letter_layer`'s closing `};` and the `};` that closes `keymap {`:

```
        mouse_fast {
            display-name = "MOUSE-F";
            bindings = <
&trans  &trans  &trans  &trans  &trans  &trans                    &trans          &trans           &mkp LCLK        &mkp MCLK        &mkp RCLK        &trans  &trans
&trans  &trans  &trans  &trans  &trans  &trans  &trans  &trans  &trans  &mmv_fast MOVE_LEFT  &mmv_fast MOVE_DOWN  &mmv_fast MOVE_UP  &mmv_fast MOVE_RIGHT  &trans  &trans
&trans  &trans  &trans  &trans  &trans  &trans  &trans          &trans          &trans           &trans           &trans           &trans           &trans  &trans
                        &trans  &trans  &trans                                  &trans           &trans           &trans
            >;

            sensor-bindings = <&scroll_encoder>;
        };
```

- [ ] **Step 2: Commit**

```bash
git add config/eyelash_corne.keymap
git commit -m "feat: add mouse_fast layer for space-accelerated mouse movement"
```

---

### Task 4: Verify the full keymap compiles

ZMK builds via GitHub Actions on push to `main`. The workflow is defined in `.github/workflows/draw.yml` — check if there's also a build workflow, or rely on the ZMK build system.

- [ ] **Step 1: Check for build workflow**

```bash
ls .github/workflows/
```

- [ ] **Step 2: Push and check Actions**

```bash
git push
```

Then open the Actions tab in the GitHub repo and confirm the build succeeds. A green build means the keymap compiles and the firmware is ready to flash.

- [ ] **Step 3: Flash firmware and test**

1. Enter bootloader mode on the keyboard
2. Copy the built `.uf2` to the drive
3. Test the following:
   - Hold Escape → enter HYPER layer
   - Press H/J/K/L → mouse moves left/down/up/right at normal speed
   - Press U/I/O → left/middle/right click
   - Hold Space while in HYPER → hold, then H/J/K/L → mouse moves faster
   - Release Space → speed returns to normal
   - Release Escape → exit HYPER layer

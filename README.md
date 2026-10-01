# Hungarian AltGr — Windows-style AltGr characters on a Hungarian Mac

A custom macOS keyboard layout that keeps the standard Apple **Hungarian** layout
exactly as it is, and adds the programming characters that a Hungarian Windows
keyboard produces with `AltGr` (right Alt) to the **Option** key of a Mac.

| Key combination | Output |
| --------------- | ------ |
| Option + Q | `@` |
| Option + W | `\|` |
| Option + E | `\` |
| Option + F | `[` |
| Option + G | `]` |
| Option + V | `@` |
| Option + B | `{` |
| Option + N | `}` |
| Option + Í | `<` |
| Option + Y | `>` |
| Option + - | `*` |
| Option + C | `&` |
| Option + , | `;` |
| Option + É | `$` |

The existing colon mapping is **not** changed:

```text
Shift + . → :
```

## 1. Project description

This project ships one file: a native macOS keyboard layout
([`layout/Hungarian-AltGr.keylayout`](layout/Hungarian-AltGr.keylayout)) called
**Hungarian AltGr**. It is a copy of Apple's Hungarian layout in which exactly
fourteen keys were changed in the Option layer, and the same fourteen keys were
changed in the Shift+Option layer (Shift+Option mirrors Option). No other key
combination was modified.

The solution is 100% native macOS:

* no Karabiner Elements,
* no Rosetta,
* no background daemon or helper app,
* nothing running at all — just a text file that macOS compiles at login.

It works on Apple Silicon (verified on a MacBook Air M1, 2020) and is
architecture-independent (it is plain XML, not machine code).

## 2. Motivation

The physical keyboard is the same (Hungarian) on Windows and macOS, but the
third-level ("AltGr") characters live in different places:

| Character | Hungarian Windows | Hungarian macOS |
| --------- | ----------------- | --------------- |
| `\` | AltGr + Q | Option + Ü |
| `\|` | AltGr + W | Option + Í |
| `[` | AltGr + F | Option + 8 |
| `]` | AltGr + G | Option + 9 |
| `@` | AltGr + V | Option + Q |
| `{` | AltGr + B | Option + 7 |
| `}` | AltGr + N | Option + 0 |
| `<` | AltGr + Í | Shift + Option + Y |
| `>` | AltGr + Y | Shift + Option + X |

Someone who developed muscle memory on a Hungarian Windows keyboard constantly
reaches for `AltGr + Q` and gets `@` (or nothing) instead of `\`. This layout
moves the Windows `AltGr` characters onto the Mac **Option** key at the same
physical positions, while leaving every other key of the standard Hungarian
layout untouched.

## 3. Keyboard mappings

The complete custom mapping:

```text
Option + Q → @
Option + W → |
Option + E → \
Option + F → [
Option + G → ]
Option + V → @
Option + B → {
Option + N → }
Option + Í → <
Option + Y → >
Option + - → *
Option + C → &
Option + , → ;
Option + É → $
```

(`@` is intentionally on **both** Option + Q and Option + V.)

`Shift + Option` produces **exactly the same characters** for these fourteen
keys — Shift does not introduce an alternative character or an
uppercase/language-specific mapping:

```text
Shift + Option + Q → @
Shift + Option + W → |
Shift + Option + E → \
Shift + Option + F → [
Shift + Option + G → ]
Shift + Option + V → @
Shift + Option + B → {
Shift + Option + N → }
Shift + Option + Í → <
Shift + Option + Y → >
Shift + Option + - → *
Shift + Option + C → &
Shift + Option + , → ;
Shift + Option + É → $
```

The existing colon mapping remains unchanged:

```text
Shift + . → :
```

Nothing else was changed. The layout was machine-verified against the stock
Apple Hungarian layout of macOS 26.6.2: all 128 key codes, all 8 modifier
layers and all 5 dead-key states — including the dead-key **state transitions**
— are identical except for the cells listed above. That is exactly
27 key/layer combinations (fourteen keys × two layers, minus Option + Q which
already equals Apple's `@`) × 6 dead-key states = 162 differing cells; every
other cell of every table is byte-for-byte
identical to Apple's layout, and the compiled modifier-state table is
identical. (The comparison maps each layout's internal dead-key state numbers
to the five states by their terminator characters, because the compiled state
numbers are file-internal and differ between compilations.)

### What each change replaces

The Option layer previously produced the following characters on these keys
(this is the only behavior that changes):

| Key | Before | After |
| --- | ------ | ----- |
| Option + Q | `\` (set by the previous version of this layout; Apple's original is `@`) | `@` |
| Option + W | `ę` (e-ogonek) | `\|` |
| Option + E | `€` | `\` |
| Option + F | `ń` (n-acute) | `[` |
| Option + G | `©` | `]` |
| Option + V | `„` (double low quote) | `@` |
| Option + B | `”` (right double quote) | `{` |
| Option + N | starts the `~` dead key | `}` |
| Option + Í | `\|` | `<` |
| Option + Y | `«` | `>` |
| Option + - | `–` (en dash) | `*` |
| Option + C | `ć` | `&` |
| Option + , | starts the `–` (dash) dead key | `;` |
| Option + É | `…` (horizontal ellipsis) | `$` |

The Shift+Option layer previously produced these (now they mirror the Option
layer):

| Key | Before | After |
| --- | ------ | ----- |
| Shift + Option + Q | `ļ` (l-cedilla) | `@` |
| Shift + Option + W | `Ł` (L-stroke) | `\|` |
| Shift + Option + E | `š` | `\` |
| Shift + Option + F | `ž` (z-caron) | `[` |
| Shift + Option + G | `Ū` (U-macron) | `]` |
| Shift + Option + V | `‚` (single low quote) | `@` |
| Shift + Option + B | `'` (right single quote) | `{` |
| Shift + Option + N | `Ų` (U-ogonek) | `}` |
| Shift + Option + Í | `Ŕ` (R-acute) | `<` |
| Shift + Option + Y | `<` | `>` |
| Shift + Option + - | `—` (em dash) | `*` |
| Shift + Option + C | `©` | `&` |
| Shift + Option + , | `*` | `;` |
| Shift + Option + É | `ō` (o-macron) | `$` |

Notes:

* `@` is now on both Option + Q and Option + V. `\` moved from Option + Q to
  Option + E. `€` is no longer on Option + E.
* `$` is now on Option + É and Shift + Option + É. `…` (ellipsis) is no longer
  on Option + É, but it remains available as Command + Option + É. `ō`
  (o-macron) is no longer on Shift + Option + É.
* The `~` dead key no longer starts from Option + N, but `~` itself is still
  available as Command + Option + N.
* The `–` (dash) dead key no longer starts from Option + , (that key now
  produces `;`), but `–` itself is still available as a plain character on
  Command + Option + , and Command + Option + -.
* The other dead keys (Option + U `¨`, Option + I `^`,
  Shift + Option + Á `ˇ`) are unchanged and fully working — they were
  restored to exact Apple parity during this update (see below).
* `Shift + Option + X` is unchanged (still `>`; X is not one of the fourteen
  keys), and `<` remains available on Option + Í.

## Right Option, Left Option — the honest limitation

> **macOS keyboard layouts cannot distinguish the left and right Option keys
> during normal typing. The `Option` layer is shared: both Option keys trigger
> the same characters. This project therefore maps the AltGr characters to
> Option generally, and they work with *either* Option key.**

This was researched and empirically verified on macOS 26.6.2 rather than
assumed:

* The `.keylayout` XML format itself **does** define separate tokens
  (`option` = left, `rightOption` = right, `anyOption` = either), and the
  compiled layout engine on macOS 26 has separate internal bits for the left
  and right Option keys. (This is documented in Apple Technical Note TN2056
  and was confirmed by compiling and inspecting test layouts on this machine.)
* However, the *typing pipeline* (physical key → AppKit → text input) passes
  only a merged "Option" state to the layout engine. Synthetic Right-Option
  key events with the correct hardware flags were injected into a running app
  and translated to the **left**-Option layer. So a `rightOption`-only layer
  compiles correctly but is never selected while typing.

Consequence: a layout that puts the AltGr characters *only* on the
`rightOption` layer would look correct in XML but would produce nothing when
typing. That is why this layout puts them on the shared Option layer instead:
it works today, on real hardware, with either Option key.

The only ways to get strictly right-Option-only behavior are third-party
keyboard remappers (such as Karabiner Elements, excluded by the project
requirements) or a custom input method / event-tap application (a running
program requiring permissions — also excluded). Within the constraints
(native `.keylayout`, no daemons, no Karabiner, no Rosetta), the shared Option
layer is the technically correct solution.

## Fn / Globe key — not possible with a native .keylayout

A natural follow-up question is whether the same AltGr mappings could sit on
the physical **Fn/Globe key** (Fn + Q → `@`, …) instead of Option. The answer
is **no** — a native `.keylayout` cannot use Fn as a modifier at all. This was
researched and empirically verified on macOS 26.6.2 rather than assumed:

* **The XML format has no Fn modifier.** The `.keylayout` modifier vocabulary
  (Apple Technical Note TN2056, and the system DTD at
  `/System/Library/DTDs/KeyboardLayout.dtd`) is `shift`, `rightShift`,
  `anyShift`, `option`, `rightOption`, `anyOption`, `control`, `rightControl`,
  `anyControl`, `command`, `caps`. There is no `fn` token — Fn cannot even be
  written in the file.
* **The compiled engine has no Fn bit.** The layout engine selects its table
  from an 8-bit modifier state (command, shift, caps, option, control,
  rightShift, rightOption, rightControl — no Fn), and `UCKeyTranslate` masks
  its modifier parameter to bits 8–15, so the Fn event flag (`bit 23`,
  `NSEventModifierFlagFunction` / `kCGEventFlagMaskSecondaryFn`) can never
  reach the layout lookup. Verified by probing: passing the Fn bit to
  `UCKeyTranslate` produces **exactly the same output as no modifier at all**,
  on both Apple's compiled Hungarian layout and this project's compiled file.
* **The Fn key is consumed earlier in the pipeline.** On Apple hardware, Fn
  is handled by the HID stack for its own purposes (F1–F12 vs. media keys,
  Fn + arrows → Home/End/PageUp/PageDown, Fn + Backspace → Delete, …). For
  letter keys, Fn + letter simply types the plain letter. Applications can
  observe the Fn state as an event flag, but the keyboard-layout engine never
  receives it.

Consequence: with this (or any) `.keylayout`, `Fn + Q` types `q`, and
`Shift + Fn + Q` types `Q`. The only ways to make Fn produce the AltGr
characters are a third-party remapper (Karabiner Elements) or a custom input
method / event-tap daemon — a running program requiring permissions. Both are
excluded by this project's constraints (native file only, no daemons, no
Karabiner, no Rosetta), so Fn support is deliberately **not** part of the
layout. See `docs/development.md` and `docs/test-plan.md` for the full
investigation record.

## 4. Compatibility

* **Verified on:** macOS 26.6.2 (Sequoia/Tahoe-generation), Apple Silicon
  (MacBook Air M1, 2020, arm64).
* **Apple Silicon:** yes — a `.keylayout` is architecture-independent XML; it
  is compiled by macOS itself at login. Verified by compiling and translating
  the layout on this arm64 machine.
* **Intel:** expected to work the same way (the format is
  architecture-independent), but this was **not verified** on Intel hardware.
* **Older macOS versions:** the `.keylayout` format has been supported since
  macOS 10.6 and is still supported on current macOS, but only macOS 26.6.2
  was actually tested.
* **Additional software:** none. No Karabiner, no Rosetta, no helpers.

## 5. Installation

1. **Copy the layout file**

   Open **Terminal** and run:

   ```sh
   mkdir -p ~/Library/Keyboard\ Layouts
   cp layout/Hungarian-AltGr.keylayout ~/Library/Keyboard\ Layouts/
   ```

   (`~/Library/Keyboard Layouts` is the per-user location. To install for all
   users, copy to `/Library/Keyboard Layouts/` instead.)

2. **Log out and log back in**

   macOS only scans the keyboard-layout folder at login. Log out
   ( → Log Out …) and log in again.

3. **Add the input source**

   Open **System Settings → Keyboard**, scroll down and click
   **Text Input → Edit…**. In the window that opens:

   * click the **+** button (bottom left),
   * type `Hungarian` in the search field,
   * select **Hungarian AltGr** from the list,
   * click **Add**.

   (Custom layouts created from the Hungarian layout appear under the same
   group as Hungarian. If the layout does not appear, see Troubleshooting.)

4. **Select the layout**

   The layout is now available in the input-menu in the menu bar. Click the
   input-menu icon and choose **Hungarian AltGr** — or press the keyboard
   shortcut to cycle input sources
   (Control + Space by default).

5. **Restart applications**

   Fully quit and reopen applications that were running while you installed
   the layout (Terminal, editors, browsers). Newly launched apps pick up the
   layout automatically; some running apps keep using the old layout until
   restarted.

6. **Test**

   Type `Option + Q` → you should get `@`, `Option + E` → `\`,
   `Option + V` → `@`, `Option + Í` → `<`, `Option + Y` → `>`,
   `Option + -` → `*`, `Option + C` → `&`, `Option + ,` → `;`,
   `Option + É` → `$`, and `Shift + .` → `:`.
   See [`docs/test-plan.md`](docs/test-plan.md) for the complete checklist.

No logout is required *again* after adding the input source in System Settings.

## 6. Removal

To completely remove the layout:

1. In **System Settings → Keyboard → Text Input → Edit…**, select
   **Hungarian AltGr** in the list and click the **−** button to remove it.
2. Delete the file:

   ```sh
   rm ~/Library/Keyboard\ Layouts/Hungarian-AltGr.keylayout
   ```

3. Log out and log back in.

## 7. Reverting to the standard layout

Switch the input source back to **Hungarian**:

* via the input menu in the menu bar, or
* in **System Settings → Keyboard → Text Input → Edit…**.

No data is lost and no settings were changed by the custom layout; the stock
Apple Hungarian layout is always still installed.

## 8. Troubleshooting

**The layout does not appear in System Settings**

* You must **log out and log back in** after copying the file; macOS scans
  `~/Library/Keyboard Layouts` only at login.
* Check the file name and location: `~/Library/Keyboard Layouts/Hungarian-AltGr.keylayout`.
* Check that the XML is intact (it should have been copied, not edited).
* Look for compiler errors: open **Console.app**, filter for
  `uchr XML compiler`. A parse error points to a line in the file.

**The layout appears but does not work (keys produce nothing)**

* The most likely cause is a modified file. Re-copy the original
  `layout/Hungarian-AltGr.keylayout` and log out/in again.

**Changes are not visible / an application still uses the old layout**

* Quit and reopen the application. Some apps (especially Terminal and IDEs)
  cache the active input source per window.

**Wrong modifier behavior**

* Remember that macOS merges the two Option keys (see the section above):
  both Option keys produce the AltGr characters. `@` is on Option + Q and
  Option + V; `\` is on Option + E.
* If Option + letter produces nothing at all in an app, make sure the input
  source shown in the menu bar is **Hungarian AltGr**, not another layout.
* `Fn + Q` produces `q`, not `@` — Fn is not usable as a modifier by any
  macOS keyboard layout (see the "Fn / Globe key" section).

## 9. Development

See [`docs/development.md`](docs/development.md).

## 10. Testing

See [`docs/test-plan.md`](docs/test-plan.md).

## 11. Extending the layout

The layout is plain, commented XML. To add or change a mapping:

1. Open `layout/Hungarian-AltGr.keylayout`.
2. Find the `<keyMap index="3">` table (the Option layer; the table numbers
   are explained in [`docs/development.md`](docs/development.md)).
3. Change or add a `<key code="…" output="…" />` entry.

Examples:

```xml
<!-- Option + A currently produces ą (unchanged stock behavior) -->
<key code="0" output="&#x104;" />

<!-- Option + E was changed to backslash -->
<key code="14" output="\" />

<!-- Option + Í was changed to less-than (&#x3c; is a literal "<") -->
<key code="50" output="&#x3c;" />

<!-- Option + C was changed to ampersand (&#x26; is a literal "&") -->
<key code="8" output="&#x26;" />
```

Key points:

* Use the **numeric** character references shown in
  [`docs/development.md`](docs/development.md) for `<`, `>`, `&`, `"` and
  control characters — the macOS layout compiler does not support named
  entities such as `&lt;`.
* Only edit the cells you intend to change; the rest of the file is a verified
  copy of Apple's Hungarian layout.
* Re-verify by installing and typing — the full verification methodology is
  documented in [`docs/development.md`](docs/development.md).

This project was created with the help of an AI coding agent (OpenCode) and is
deliberately structured so that another developer — human or AI — can
understand and extend it without any prior conversation. See
[`AGENTS.md`](AGENTS.md).

## License

MIT — see [LICENSE](LICENSE).

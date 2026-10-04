# Development guide

How the `Hungarian-AltGr.keylayout` file is structured, how it was created
and verified, and how to extend it safely.

## Background: what a `.keylayout` is

A `.keylayout` is an XML file that macOS compiles at login into its internal
`uchr` keyboard-layout format. Apple documents the XML in Technical Note
**TN2056 — "Installable Keyboard Layouts"** (archived at
developer.apple.com). The format has existed since macOS 10.6 and is still the
native mechanism for custom keyboard layouts on current macOS.

Because macOS on modern systems (macOS 12+) no longer ships its built-in
layouts as readable XML (they live in the compiled
`AppleKeyboardLayouts-L.dat`), this project's layout was **reconstructed from
the live Apple Hungarian layout** by probing the text-input engine, then
verified cell-by-cell against it (see "Verification" below). The result is a
file that behaves identically to Apple's Hungarian layout except for exactly
fifteen keys changed in the Option layer and mirrored in the Shift+Option
layer (Shift+Option mirrors Option exactly), i.e. 30 key/layer combinations,
of which 29 differ from Apple's (Option+Q already equals Apple's `@`)
= 174 cells across the six dead-key states.

## File structure

```xml
<keyboard group="126" id="-12001" name="Hungarian AltGr" maxout="2">
  <layouts>…</layouts>          <!-- keyboard-type ranges -->
  <modifierMap id="common">…</modifierMap>
  <keyMapSet id="ANSI">…</keyMapSet>
  <terminators>…</terminators>  <!-- dead-key fallback outputs -->
</keyboard>
```

* `group="126"` — Unicode keyboard group (custom layouts).
* `id="-12001"` — unique numeric ID. Negative = custom Unicode layout. If it
  collides with another installed layout, macOS renumbers it automatically.
* `name` — shown in the input menu and System Settings.
* `maxout="2"` — some key presses (dead-key terminator + base character) emit
  two UTF-16 units.

## Modifier maps

The `<modifierMap>` selects one of 8 `<keyMap>` tables for each modifier
combination:

| Table | Selected by (XML `keys` value) | Meaning |
| ----- | ------------------------------ | ------- |
| 0 | `""` (+ default) | no modifiers |
| 1 | `anyShift caps?` | Shift (either) |
| 2 | `caps command?` | Caps Lock |
| 3 | `anyOption caps?` | **Option (either) — the AltGr layer** |
| 4 | `anyShift anyOption caps? command?` | Shift + Option |
| 5 | `anyOption caps command?` | Caps Lock + Option |
| 6 | `anyOption command` | Command + Option |
| 7 | (explicit enumeration) | Control |

Internally (verified on macOS 26.6.2) the engine uses an 8-bit modifier state:

```text
bit 0 (0x01) command      bit 4 (0x10) control
bit 1 (0x02) shift        bit 5 (0x20) rightShift
bit 2 (0x04) caps         bit 6 (0x40) rightOption
bit 3 (0x08) option       bit 7 (0x80) rightControl
```

Token semantics (per TN2056, confirmed by compiling test layouts):

* `option` — left Option only; `rightOption` — right Option only;
  `anyOption` — either or both. Same pattern for shift/control.
* `command` and `caps` have no left/right variants.
* `word?` = state irrelevant; absence = modifier must be up.

The table-7 select is an explicit enumeration of 63 `<modifier>` elements. It
maps every control combination to the control table while leaving the
physically unreachable all-eight-modifiers state (0xFF) on the default table,
exactly matching Apple's compiled layout. Do not replace it with a single
`anyControl …?` element; that would remap state 0xFF.

> **Important:** the XML tokens distinguish left/right Option, and the engine
> has separate bits for them — but the **typing pipeline delivers only the
> merged Option state** (verified: a synthetic Right-Option event is
> translated through the left-Option table). So in practice every Option
> layer is "either Option key". Keep the AltGr characters on `anyOption`
> (table 3), never on a `rightOption`-only layer.

## Key codes

The `code` attribute is the macOS virtual key code (ANSI keyboard):

```text
12 = Q     13 = W     14 = E     3 = F     5 = G     9 = V     11 = B
45 = N     50 = Í (key left of Y / ANSI `)    6 = Y (ANSI Z position)    7 = X
8 = C      43 = , (comma)    44 = - (hyphen, ANSI slash position)
47 = . (period; Shift+. is the colon mapping)
```

Other codes used by the layout include `0`=A, `1`=S, `14`=E, `15`=R, `16`=Z
(Hungarian QWERTZ: the ANSI Y key is Z), `18..29` = digit row,
`24`=Ó, `27`=Ü, `29`=Ö, `30`=Ú, `33`=Ő, `39`=Á, `41`=É, `42`=Ű.

## Output characters and XML escaping

The macOS layout compiler accepts **numeric character references only**.
Named entities (`&lt;`, `&gt;`, `&amp;`, `&quot;`) are parsed by generic XML
parsers but are **silently not supported** by the layout compiler — a file
that "validates" in an XML tool can still compile to empty outputs. Always
use:

| Character | Write as |
| --------- | -------- |
| `\` | `\` (literal) |
| `\|` | `\|` (literal) |
| `[` `]` | literal |
| `@` | literal |
| `{` `}` | literal |
| `<` | `&#x3c;` |
| `>` | `&#x3e;` (literal `>` also works, but be consistent) |
| `&` | `&#x26;` |
| `"` | `&#x22;` |
| control chars (e.g. ^A) | `&#x1;` |

(The C0 control-character references in the control table are rejected by
strict XML 1.0 validators such as `xmllint`, but Apple's own shipped layouts
contain exactly these references and macOS's compiler accepts them. `xmllint`
errors of the form "invalid xmlChar value" are expected here, not a bug.)

## Dead keys

The layout has five dead-key states (unchanged from Apple's Hungarian). The
states and their starters on Apple's layout, verified by probing the live
layout (a starter is a key whose translation leaves a non-zero dead-key
state):

| State | Terminator output | Started by (Apple) | Status in this layout |
| ----- | ----------------- | ------------------ | --------------------- |
| diaeresis | `¨` | Option + U | restored, identical to Apple |
| circumflex | `^` | Option + I | restored, identical to Apple |
| dash | `–` (en dash) | Option + , | **replaced**: Option + , now outputs `;` (the dash state is no longer enterable) |
| tilde | `~` | Option + N | **replaced** (earlier, intentional): Option + N now outputs `}` |
| caron | `ˇ` | Shift + Option + Á | restored, identical to Apple |

Each starter is an inline `<action>` of the form

```xml
<key code="32">
    <action>
        <when state="none" next="diaeresis" />
        <when state="diaeresis" output="¨" next="diaeresis" />
        <when state="circumflex" output="^" next="diaeresis" />
        <when state="dash" output="–" next="diaeresis" />
        <when state="tilde" output="~" next="diaeresis" />
        <when state="caron" output="ˇ" next="diaeresis" />
    </action>
</key>
```

i.e. pressing the dead key while another dead key is pending flushes the
pending one (outputs its terminator) and enters the new state — exactly the
behavior of Apple's compiled layout.

**History note:** the first reconstruction of this file contained no `next=`
attributes at all, so none of the five dead keys could be entered (the
verification at the time compared output characters for each *input* state,
and a missing starter is invisible to that comparison because a starter
outputs nothing). The starters were restored and the verification extended to
state transitions during the 2026 update that added the Fn investigation.

Each participating key carries inline `<action>` elements with one `<when>`
per dead-key state; non-participating keys fall through to the
`<terminators>` outputs. Composition examples: a→ä, o→ö, u→ü (diaeresis);
n→ň, s→š, c→č, … (caron) — exactly as in Apple's layout. Note that Apple's
Hungarian has no circumflex composition on e (Option + I then E yields
`^e`), and no circumflex composition on a either; the composition set matches
Apple's byte for byte.

## Adding a new mapping

Example: make `Option + A` produce `~` (it currently produces `ą`):

1. Open `layout/Hungarian-AltGr.keylayout`.
2. Find `<keyMap index="3">` (the Option layer).
3. Find the line for key code 0:

   ```xml
   <key code="0" output="&#x105;" />
   ```

4. Replace the output:

   ```xml
   <key code="0" output="~" />
   ```

5. Reinstall and test (below).

## Modifying an existing mapping

Same procedure. The fifteen AltGr entries currently look like this in the
Option layer (`<keyMap index="3">`):

```xml
<key code="12" output="@" />      <!-- Option+Q -->
<key code="13" output="|" />      <!-- Option+W -->
<key code="14" output="\" />      <!-- Option+E -->
<key code="3"  output="[" />      <!-- Option+F -->
<key code="5"  output="]" />      <!-- Option+G -->
<key code="9"  output="@" />      <!-- Option+V -->
<key code="11" output="{" />      <!-- Option+B -->
<key code="45" output="}" />      <!-- Option+N -->
<key code="50" output="&#x3c;" /> <!-- Option+Í (less-than) -->
<key code="6"  output="&#x3e;" /> <!-- Option+Y (greater-than) -->
<key code="7"  output="#" />      <!-- Option+X -->
<key code="44" output="*" />      <!-- Option+- (hyphen) -->
<key code="8"  output="&#x26;" /> <!-- Option+C (ampersand) -->
<key code="43" output=";" />      <!-- Option+, (comma) -->
<key code="41" output="$" />      <!-- Option+É -->
```

… and identically in the Shift+Option layer (`<keyMap index="4">`), so that
`Shift + Option + key` produces exactly the same character as `Option + key`:

```xml
<key code="12" output="@" />      <!-- Shift+Option+Q -->
<key code="13" output="|" />      <!-- Shift+Option+W -->
<key code="14" output="\" />      <!-- Shift+Option+E -->
<key code="3"  output="[" />      <!-- Shift+Option+F -->
<key code="5"  output="]" />      <!-- Shift+Option+G -->
<key code="9"  output="@" />      <!-- Shift+Option+V -->
<key code="11" output="{" />      <!-- Shift+Option+B -->
<key code="45" output="}" />      <!-- Shift+Option+N -->
<key code="50" output="&#x3c;" /> <!-- Shift+Option+Í (less-than) -->
<key code="6"  output="&#x3e;" /> <!-- Shift+Option+Y (greater-than) -->
<key code="7"  output="#" />      <!-- Shift+Option+X -->
<key code="44" output="*" />      <!-- Shift+Option+- (hyphen) -->
<key code="8"  output="&#x26;" /> <!-- Shift+Option+C (ampersand) -->
<key code="43" output=";" />      <!-- Shift+Option+, (comma) -->
<key code="41" output="$" />      <!-- Shift+Option+É -->
```

When you change one of these keys, change **both** entries, unless there is a
specific reason the Shift+Option layer should differ.

Rules:

* Only change the cell you intend to change. Every other entry in every table
  is a verified copy of Apple's layout.
* If you change a key that participates in dead-key actions, keep the
  `<action>` block intact (or, if the new output must not compose, replace the
  whole element with a plain `output=…` entry — this is what was done for
  Option + N and Option + ,).
* Keep `output` values to characters legal in XML attributes, escaped as
  above.

## Installing a development version locally

Two options:

**Standard (recommended):**

```sh
cp layout/Hungarian-AltGr.keylayout ~/Library/Keyboard\ Layouts/
```

Then log out and log back in, add/select the input source as described in
[`installation.md`](installation.md). Every file change requires another
logout/login cycle.

**Fast iteration (developer-only, private API):** a process can register the
file without logging out using the private `TISRegisterInputSource` (obtain it
via `dlsym(RTLD_DEFAULT, "TISRegisterInputSource")`):

```c
CFURLRef url = CFURLCreateFromFileSystemRepresentation(NULL, path, strlen(path), false);
((OSStatus (*)(CFURLRef))TISRegisterInputSource)(url);
```

Caveats learned while developing this project:

* The registered source only becomes selectable/enabled reliably after the
  session has processed it; calling `TISEnableInputSource` and
  `TISSelectInputSource` in the same process often returns `paramErr` (-50).
  A second process run usually succeeds.
* Repeated registrations of files with the same numeric `id` collide and get
  renumbered ("duplicate keyboard layout identifier … replaced with …" on
  stderr), which resets enable state. Use a unique id per variant, or clean up
  and log out/in.
* Registered sources persist for the session even if the file is deleted;
  they disappear at the next login.

For real typing tests, the standard logout/login flow is always the ground
truth.

## Testing changes

1. **Manual:** install, then run [`test-plan.md`](test-plan.md).
2. **Machine verification** (how this file was created and validated): probe
   the compiled layouts with `UCKeyTranslate` — the same API the typing
   pipeline uses — for every virtual key code (0–127), every modifier table,
   and every dead-key state, then diff:

   * Probe the stock Apple "Hungarian" input source (via
     `TISGetInputSourceProperty(…, kTISPropertyUnicodeKeyLayoutData)`).
   * Compile the edited XML (install it or `TISRegisterInputSource`), get its
     layout data, and probe it identically.
   * Diff both matrices. The accepted result for this project is: **exactly
     174 differing cells = 2 layers (Option, Shift+Option) × 15 keys
     (minus Option+Q, which equals Apple's `@` again) × 6 dead-key states**,
     and an identical 255-entry modifier table.

   Two things the original verification got wrong, corrected here:

   * **State transitions.** Compare the resulting dead-key state of each
     translation, not just the output characters. A dead-key *starter* outputs
     nothing, so dropping a starter is invisible to an output-only
     comparison — the original file silently lost all five starters.
   * **State numbering.** The compiled `deadKeyState` values are
     file-internal; different compilations number the same five states
     differently (the stock Apple layout uses 1–5 in one order, an
     XML-compiled file another order and even a different bit layout). Map
     each layout's state numbers to the five states by their terminator
     characters (translate a plain key, e.g. `x`, in that state and read the
     first character) before comparing.

   A small C program of ~150 lines is enough for this; it was used to generate
   the shipped file and to prove the equivalence. It is intentionally not part
   of the repository (the layout is meant to stay a single-file artifact), but
   the methodology above is complete.

3. **End-to-end (performed during development):** a Cocoa test app can select
   the layout, inject synthetic key events (Right- and Left-Option flagged)
   into its own text view, and capture the inserted text. This established
   that macOS merges the two Option keys and that Option-layer mappings work
   for both.

## Fn / Globe key — investigation record

The request to put the AltGr characters on the physical Fn/Globe key was
investigated on macOS 26.6.2. **Conclusion: not possible with a native
`.keylayout`; therefore not implemented, per the project rules** (no
Karabiner, no daemons, no Rosetta). The evidence:

1. **No XML token.** The modifier vocabulary of the `.keylayout` format
   (TN2056; also the system DTD
   `/System/Library/DTDs/KeyboardLayout.dtd`) is `shift`, `rightShift`,
   `anyShift`, `option`, `rightOption`, `anyOption`, `control`,
   `rightControl`, `anyControl`, `command`, `caps`. There is no `fn` token,
   so Fn cannot be expressed in the file at all.
2. **No engine bit.** The uchr modifier state is 8 bits
   (`keyModifiersToTableNum`, bit table documented under "Modifier maps"),
   and `UCKeyTranslate` masks its modifier parameter to bits 8–15
   (`(EventRecord.modifiers >> 8) & 0xFF` per the header documentation).
   The Fn event flag (`kCGEventFlagMaskSecondaryFn` /
   `NSEventModifierFlagFunction` = bit 23) is structurally unreachable.
3. **Empirical probe.** Passing the Fn bit (and also bit 15) to
   `UCKeyTranslate` for several keys produces byte-identical output to
   passing no modifier — on both the live Apple Hungarian layout and the
   compiled file from this project:

   ```text
   mods=none      code=12 -> q      (Q key, no modifiers)
   mods=Fn        code=12 -> q      (Fn bit 1<<23: ignored)
   mods=Fn+shift  code=12 -> Q      (same as shift alone)
   mods=Fn+option code=12 -> @      (same as option alone)
   ```

4. **Hardware layer.** The physical Fn key is consumed by the HID stack
   before text input: Fn+F1–F12 switches function/media keys, Fn+arrows
   produce Home/End/PageUp/PageDown, Fn+Backspace produces Delete. For
   letter keys Fn+letter simply types the letter. Applications observe Fn
   only as an event flag (`NSEventModifierFlagFunction`), which the text
   input system does not forward to the layout engine.

Consequence: `Fn + Q` types `q` and `Shift + Fn + Q` types `Q` with any
`.keylayout`. The only ways to give Fn the AltGr characters are a third-party
remapper (Karabiner Elements) or a custom input-method/event-tap program —
both excluded here. If a future macOS version exposes Fn to the layout engine,
re-run the `UCKeyTranslate` probe first and only then implement it.

## Removing the development version

```sh
rm ~/Library/Keyboard\ Layouts/Hungarian-AltGr.keylayout
```

then remove the input source in System Settings and log out/in.

## Known quirks (verified, intentional)

* `Option` here means *either* Option key (macOS limitation, see README).
* The layout name is **not localized** (`.keylayout` files cannot localize
  their display name).
* `xmllint` reports "invalid xmlChar value" for the control-table character
  references — expected; Apple's own layouts contain the same pattern and the
  macOS compiler accepts it.
* The all-eight-modifiers state (0xFF) falls through to the default table,
  matching Apple's compiled layout.
* The compiled dead-key `deadKeyState` numbers are file-internal and differ
  between compilations (the stock Apple layout and an XML-compiled layout
  number the same five states differently). Compare states semantically, by
  their terminator characters, never by raw state numbers.
* Fn/Globe cannot be used as a modifier by any `.keylayout` (see the
  investigation record above).

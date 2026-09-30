# AGENTS.md — instructions for AI coding agents

This file lets an AI coding agent (or a new human developer) understand this
project completely without the original conversation.

## Project purpose

The user is a Hungarian programmer on a MacBook Air (M1, 2020) who previously
used a Hungarian Windows keyboard. On Windows, `AltGr` (right Alt) produces the
programming characters at familiar positions:

```text
AltGr+Q → \   AltGr+W → |   AltGr+F → [   AltGr+G → ]   AltGr+V → @
AltGr+B → {   AltGr+N → }   AltGr+Í → <   AltGr+Y → >
```

On macOS those characters live in unintuitive places. This project ships a
native `.keylayout` that keeps Apple's Hungarian layout completely intact and
changes exactly thirteen keys — in the Option layer and, mirrored exactly, in
the Shift+Option layer — so that:

```text
Option+Q → @   Option+W → |   Option+E → \   Option+F → [   Option+G → ]
Option+V → @   Option+B → {   Option+N → }   Option+Í → <   Option+Y → >
Option+- → *   Option+C → &   Option+, → ;
```

`Shift + . → :` must remain unchanged, along with everything else.

## Architecture

* **One file does everything:** `layout/Hungarian-AltGr.keylayout`
  ("Hungarian AltGr", id `-12001`, group `126`, `maxout="2"`).
* macOS compiles `.keylayout` XML at login into its internal `uchr` format and
  uses it for all text input. There is **no runtime component**: no daemon, no
  app, no login item.
* Installation = copy the file to `~/Library/Keyboard Layouts/` (per user) or
  `/Library/Keyboard Layouts/` (all users), log out/in, then add the input
  source in **System Settings → Keyboard → Text Input → Edit…**.
* The file is a faithful reconstruction of Apple's Hungarian layout. Its
  structure follows Apple's own keyboard-layout XML (Apple Technical Note
  TN2056, "Installable Keyboard Layouts").

### Layout structure

* `<keyboard>` — one per file. `group="126"`, negative `id` (custom layouts),
  `name` (shown in the input menu).
* `<layouts>` — maps hardware keyboard ranges (`first="0" last="207"`) to a
  `<modifierMap>` and a `<keyMapSet>`.
* `<modifierMap>` — maps **modifier state combinations** to `keyMap` table
  indices. This file uses the same mapping Apple's Hungarian layout uses:
  * table 0: no modifiers (default)
  * table 1: shift (`anyShift caps?`)
  * table 2: caps lock (`caps command?`)
  * table 3: **Option layer** (`anyOption caps?`) ← the AltGr changes are here
  * table 4: shift+option (`anyShift anyOption caps? command?`) — the thirteen
    AltGr keys mirror the Option layer here (Shift + Option + key ==
    Option + key)
  * table 5: caps+option (`anyOption caps command?`)
  * table 6: command+option (`anyOption command`)
  * table 7: control — an explicit enumeration of 64 combinations with
    `anyControl` (see below)
* `<keyMapSet>` — the 8 `<keyMap>` tables; each maps virtual key codes to
  outputs or actions.
* `<key>` — `code` = macOS virtual key code; `output` = UTF-16 string, or an
  inline `<action>` for dead-key participation.
* `<actions>`/`<terminators>` — dead-key state machine. This file uses only
  inline actions plus `<terminators>`.

### Modifier internals (verified on macOS 26.6.2, do not re-derive blindly)

The compiled engine has an 8-bit modifier state (255-entry table):

```text
bit 0 (0x01) command      bit 4 (0x10) control
bit 1 (0x02) shift        bit 5 (0x20) rightShift
bit 2 (0x04) caps         bit 6 (0x40) rightOption
bit 3 (0x08) option       bit 7 (0x80) rightControl
```

The XML `keys` tokens map to these bits: `option` = left Option only,
`rightOption` = right Option only, `anyOption` = either/both, and likewise for
shift and control. `word?` means "state irrelevant"; absence means "must be
up".

The control table (table 7) reproduces Apple's exact state mapping by
enumerating all 64 combinations of `{command, shift, rightShift, caps, option,
rightOption}` together with `anyControl`, except the all-six combination, which
is emitted once with `control` and once with `rightControl`. Net effect: every
control state maps to table 7 except the physically unreachable
all-eight-modifiers state (0xFF), which falls through to the default table —
exactly like Apple's compiled layout. Do not "simplify" this list to a single
`anyControl anyShift? …` element: that changes state 0xFF and breaks the
verified parity with Apple's layout.

## Current mappings

The custom cells are in `<keyMap index="3">` (Option layer) and
`<keyMap index="4">` (Shift+Option layer). The Shift+Option cells **mirror**
the Option cells exactly — Shift must not introduce an alternative character
or an uppercase/language-specific mapping.

Option layer (`<keyMap index="3">`):

| Virtual key code | Physical key | Output | Replaced |
| ---------------- | ------------ | ------ | -------- |
| 12 | Q | `@` | `\` (previous version of this layout; Apple's original `@` is restored) |
| 13 | W | `\|` | `ę` |
| 14 | E | `\` | `€` |
| 3 | F | `[` | `ń` |
| 5 | G | `]` | `©` |
| 9 | V | `@` | `„` |
| 11 | B | `{` | `”` |
| 45 | N | `}` | `~` dead key starter |
| 50 | Í | `<` (written `&#x3c;`) | `\|` |
| 6 | Y | `>` (written `&#x3e;`) | `«` |
| 44 | - | `*` | `–` (en dash) |
| 8 | C | `&` (written `&#x26;`) | `ć` |
| 43 | , | `;` | `–` (dash dead key starter) |

Shift+Option layer (`<keyMap index="4">`), same outputs:

| Virtual key code | Physical key | Output | Replaced |
| ---------------- | ------------ | ------ | -------- |
| 12 | Q | `@` | `ļ` |
| 13 | W | `\|` | `Ł` |
| 14 | E | `\` | `š` |
| 3 | F | `[` | `ž` |
| 5 | G | `]` | `Ū` |
| 9 | V | `@` | `‚` |
| 11 | B | `{` | `'` |
| 45 | N | `}` | `Ų` |
| 50 | Í | `<` (written `&#x3c;`) | `Ŕ` |
| 6 | Y | `>` (written `&#x3e;`) | `<` |
| 44 | - | `*` | `—` (em dash) |
| 8 | C | `&` (written `&#x26;`) | `©` |
| 43 | , | `;` | `*` |

`Shift + . → :` lives in table 1 (shift layer), key code 47, and is unchanged.

### Dead-key starters (restored, verified)

Apple's Hungarian layout starts five dead keys from the Option layers. The
original reconstruction of this file had dropped all five starters (it had no
`next=` attributes anywhere), so no dead key could ever be entered. This was
found and fixed during the Fn-investigation update (macOS 26.6.2, verified by
probing both the live Apple layout and the compiled file with `UCKeyTranslate`,
including the dead-key state transitions — the original verification had only
compared output characters per input state):

* Option + U → `¨` (diaeresis) — restored, identical to Apple
* Option + I → `^` (circumflex) — restored, identical to Apple
* Shift + Option + Á → `ˇ` (caron) — restored, identical to Apple
* Option + N → tilde starter — deliberately replaced by `}` (unchanged
  intention from the original project)
* Option + , → dash starter — deliberately replaced by `;` (the new comma
  mapping). The `–` dash dead key is therefore no longer enterable, but `–`
  itself remains available as a plain character on Command + Option + , and
  Command + Option + - . This mirrors the existing tilde situation (`~` still
  on Command + Option + N).

Each restored starter is an inline `<action>` with `<when state="…" next="…">`
entries that flush any pending dead key first (output the pending state's
terminator) and then enter the new state — byte-for-byte the behavior of
Apple's compiled layout.

## Rules for modifications

You MUST:

* preserve all existing mappings and all AltGr mappings (the thirteen keys, in
  both the Option and the Shift+Option layers);
* avoid unrelated keyboard-layout changes — every other cell of every table is
  a verified copy of Apple's Hungarian layout;
* never introduce Karabiner Elements unless the user explicitly asks;
* never introduce background applications, daemons, or login items;
* never introduce Rosetta dependencies (nothing here needs it);
* never introduce dependencies (no package managers, no build systems);
* preserve Apple Silicon compatibility (the file is architecture-independent
  XML; nothing to compile);
* keep the project minimal — new files only when actually useful.

## Right Option handling

The original request asked for *Right Option only*. That is **not achievable
with a native `.keylayout`** — verified empirically on macOS 26.6.2:

1. The XML format compiles `option`/`rightOption`/`anyOption` into distinct
   engine states (verified: left Option = state 0x08, right Option = 0x40).
2. But the typing pipeline (WindowServer → AppKit → text input) delivers only
   the merged Option state: synthetic Right-Option key events with the correct
   hardware flags (`kCGEventFlagMaskAlternate | 0x100`) were translated through
   the left-Option table by a real app.

Therefore a layout that assigns the AltGr characters *only* to the
`rightOption` layer would compile cleanly but produce nothing when typing.
That is why the characters live on the shared Option layer: both Option keys
trigger them. This tradeoff is documented in the README and must not be
"fixed" by moving mappings to `rightOption` unless a future macOS is verified
(first) to deliver the right-Option state during real typing.

Any future agent claiming to distinguish left/right Option must **verify**
with a real test (e.g., the methodology in `docs/development.md`), not assume.

## Testing requirements

Every change to the keyboard layout must be tested manually where possible.
At minimum, after installing a changed layout, verify:

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
Shift + . → :
```

Also verify that unrelated combinations are unchanged (plain letters, shifted
letters, accented Hungarian letters, numbers, punctuation, the dead keys
listed in `docs/development.md`, other Option combinations). Full checklist:
`docs/test-plan.md`.

There is a machine-verification methodology: probe the live Apple layout with
`UCKeyTranslate` (all 128 key codes × 8 tables × dead-key states **including
state transitions**), compile the edited XML with the system compiler (register
the file with the private `TISRegisterInputSource` or simply install it), probe
it the same way, and diff **semantically** (mapping each layout's internal dead
key state numbers to the five states by their terminator characters — the
compiled state numbers are file-internal and differ between compilations). The
verified result for the current layout is exactly the 25 intended key/layer
combos (13 keys × 2 layers, Option and Shift+Option) across 6 dead-key states
= 150 differing cells, and nothing else. See
`docs/development.md` for details.

## Fn / Globe key handling

The request to put the AltGr characters on the physical **Fn/Globe key**
instead of Option is **not achievable with a native `.keylayout`** — verified
empirically on macOS 26.6.2:

1. The `.keylayout` XML modifier vocabulary (TN2056 and the system DTD) has no
   `fn` token. The modifiers are: `shift`, `rightShift`, `anyShift`, `option`,
   `rightOption`, `anyOption`, `control`, `rightControl`, `anyControl`,
   `command`, `caps`. Fn cannot even be expressed in the file format.
2. The compiled engine selects its table from an 8-bit modifier state (the
   table documented under "Modifier internals" above) that has no Fn bit, and
   `UCKeyTranslate` masks its modifier parameter to bits 8–15
   (`(EventRecord.modifiers >> 8) & 0xFF`), so the Fn event flag
   (`kCGEventFlagMaskSecondaryFn` / `NSEventModifierFlagFunction`, bit 23) can
   never reach the layout lookup. Verified by probing: passing the Fn bit to
   `UCKeyTranslate` produces exactly the same output as no modifier at all, on
   both Apple's compiled Hungarian layout and this project's compiled file.
3. The physical Fn key is consumed by the HID stack before text input (F-keys
   vs. media keys, Fn + arrows → Home/End/PageUp/PageDown, Fn + Backspace →
   Delete, …). For letter keys, Fn + letter simply types the plain letter.
   Apps can see the Fn state as an event flag, but the keyboard-layout engine
   never receives it.

Consequence: with any `.keylayout`, `Fn + Q` types `q` (and `Shift + Fn + Q`
types `Q`) — the layout cannot change this. Implementing Fn-as-AltGr would
require a runtime input method or event-tap (a running program with
permissions, e.g. Karabiner Elements), which is excluded by the project rules.
Do not "fix" this by introducing such a dependency; document it instead (README
and `docs/development.md`). If a future macOS exposes Fn to the layout engine,
**verify** it first with the `UCKeyTranslate` methodology, then implement.

## Documentation requirements

Whenever keyboard behavior changes:

* update `README.md` (mapping tables, "what each change replaces", limitations);
* update the relevant documentation;
* update `docs/test-plan.md` if the test checklist is affected;
* explain the change in a clear commit message.

## Git requirements

The user will initialize the Git repository themselves. You must NOT:

* run `git init`,
* create a `.git` directory,
* create Git metadata or hooks,
* make commits.

Create and edit files only.

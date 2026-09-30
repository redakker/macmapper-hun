# Manual test plan

Run these checks after installing the **Hungarian AltGr** layout (see
[`installation.md`](installation.md)). Use a plain text editor for the
character checks, then repeat the "Required mappings" block in each
application listed at the bottom.

Remember: **both Option keys behave the same** on macOS (the layout engine
cannot separate them during typing). Test with either Option key; expect
identical results.

## Required mappings

Verify the **actual resulting characters** (copy the text out and inspect it),
not merely that a key event was generated.

- [ ] Option + Q → `@`
- [ ] Option + W → `|`
- [ ] Option + E → `\`
- [ ] Option + F → `[`
- [ ] Option + G → `]`
- [ ] Option + V → `@`
- [ ] Option + B → `{`
- [ ] Option + N → `}`
- [ ] Option + Í → `<`
- [ ] Option + Y → `>`
- [ ] Option + - → `*`
- [ ] Option + C → `&`
- [ ] Option + , → `;`

(`@` is intended on **both** Option + Q and Option + V.)

Shift+Option must produce exactly the same characters (Shift introduces no
alternative character and no uppercase/language-specific mapping):

- [ ] Shift + Option + Q → `@`
- [ ] Shift + Option + W → `|`
- [ ] Shift + Option + E → `\`
- [ ] Shift + Option + F → `[`
- [ ] Shift + Option + G → `]`
- [ ] Shift + Option + V → `@`
- [ ] Shift + Option + B → `{`
- [ ] Shift + Option + N → `}`
- [ ] Shift + Option + Í → `<`
- [ ] Shift + Option + Y → `>`
- [ ] Shift + Option + - → `*`
- [ ] Shift + Option + C → `&`
- [ ] Shift + Option + , → `;`

## Fn / Globe key — not possible (documented limitation)

Fn cannot be used as a modifier by any macOS `.keylayout`; this was
investigated and machine-verified on macOS 26.6.2 (see
[`development.md`](development.md) and the README). The desired mappings
`Fn + Q → @`, `Fn + W → |`, … **cannot be implemented** with the native
approach, and implementing them would require a third-party remapper or a
background daemon (excluded by this project's rules).

These checks document the actual, expected behavior — they are *expected to
fail as custom mappings*, and that failure is correct:

- [ ] `Fn + Q` → `q` (plain letter — not `@`)
- [ ] `Fn + W` → `w` (plain letter — not `|`)
- [ ] `Fn + E` → `e` (plain letter — not `\`)
- [ ] `Shift + Fn + Q` → `Q` (shifted letter — not `@`)
- [ ] `Shift + Fn + E` → `E` (shifted letter — not `\`)
- [ ] Fn alone does nothing on letter keys (no character is inserted)

If a future macOS version exposes Fn to the keyboard-layout engine, revisit
the investigation in [`development.md`](development.md) and add real Fn tests
here.

## Existing behavior (must be unchanged)

- [ ] `Shift + .` → `:`
- [ ] normal letters work (a, s, d, f …)
- [ ] Hungarian accented letters work (á é í ó ö ő ú ü ű on their own keys)
- [ ] Shift + letters work (A, S, D …; capital accented letters)
- [ ] numbers work (0–9, including the Hungarian digit-row order)
- [ ] punctuation works (`,`, `-`, `.`, `'`, `+`, `%`, `/`, `=`, `(`, `)`)
- [ ] Shift + number symbols work (`'`, `"`, `+`, `!`, `%`, `/`, `=`, `(`, `)`)
- [ ] unrelated Option combinations still work:
  - [ ] Option + 1 → `&` (still)
  - [ ] Option + 8 → `[` (still)
  - [ ] Option + 9 → `]` (still)
  - [ ] Option + 7 → `{` (still)
  - [ ] Option + 0 → `}` (still)
  - [ ] Option + Ü → `\` (still; the Ü key was never changed)
  - [ ] Option + X → `»`
  - [ ] Shift + Option + X → `>` (still; X is not one of the thirteen keys)
  - [ ] Command + Option + N → `~` (still)
  - [ ] Command + Option + , → `–` (still; the dash dead key is gone, but the
    character itself remains here)
  - [ ] Option + Ú → `~` (still)
- [ ] dead keys still work (restored to exact Apple parity):
  - [ ] Option + U, then A → `ä`; Option + U, then O → `ö`;
        Option + U, then U → `ü`; Option + U, then Space → `¨`
  - [ ] Option + I, then E → `^e` (Apple's Hungarian has no ê composition);
        Option + I, then Space → `^`
  - [ ] Shift + Option + Á, then N → `ň`; Shift + Option + Á, then Space → `ˇ`
  - [ ] chaining: Option + U, then Option + I → `¨` (flushes the pending
        diaeresis and enters circumflex)
- [ ] Option + N → `}` (the tilde dead key intentionally does not start here)
- [ ] Option + , → `;` (the dash dead key intentionally does not start here)

## Applications

Repeat the "Required mappings" block (at least Option + Q, E, F, G, V, Í, Y,
-, C, , and the corresponding Shift+Option rows) in each of:

- [ ] Terminal
- [ ] iTerm2 (if installed)
- [ ] a text editor (TextEdit or another)
- [ ] a browser text field (address bar or a form)
- [ ] an IDE (VS Code / Xcode / JetBrains — whichever is available)

Notes:

* If an application shows the old behavior, quit and reopen it.
* In terminal apps, `Option` can be captured as the Meta/Esc key by the app's
  own settings; if Option + letter inserts nothing there but works elsewhere,
  check the app's Option-key preferences before assuming a layout problem.

## Regression checklist after any layout edit

1. Re-run the Required mappings block (both the Option and the Shift+Option
   rows).
2. Re-run the colon check (`Shift + .` → `:`).
3. Re-run the dead-key block (Option + U / I, Shift + Option + Á) — dead keys
   are easy to break with a bad `next=` edit.
4. Spot-check 3–5 untouched keys across different layers (e.g. `é`,
   `Shift + 1` → `'`, `Option + Ü` → `\`, `Option + 1` → `&`,
   `Shift + Option + X` → `>`).
5. If an entry was added/changed on a key that participates in a dead-key
   action, re-test that dead key.
6. Re-install (log out/in) before concluding a change has no effect.

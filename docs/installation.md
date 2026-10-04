# Installation guide

Installing the **Hungarian AltGr** layout on macOS, step by step, from
downloading/cloning the project to typing `Option + E` → `\`.

The steps were verified on macOS 26.6.2 (System Settings era UI).

## 1. Locate the layout file

The layout is a single file:

```text
layout/Hungarian-AltGr.keylayout
```

Nothing else from the repository needs to be installed. The other files are
documentation.

## 2. Copy it to the correct directory

macOS scans keyboard-layout files from two fixed locations at login:

* `~/Library/Keyboard Layouts/` — only for your user account (recommended)
* `/Library/Keyboard Layouts/` — for all users (requires admin rights)

In Terminal, run (adjust the path to where the repository is):

```sh
mkdir -p ~/Library/Keyboard\ Layouts
cp layout/Hungarian-AltGr.keylayout ~/Library/Keyboard\ Layouts/
```

Check that the file is there:

```sh
ls ~/Library/Keyboard\ Layouts/Hungarian-AltGr.keylayout
```

Note: `~/Library` is hidden in Finder. To browse there with Finder, press
`Cmd + Shift + G` and type `~/Library/Keyboard Layouts`.

## 3. Log out and log back in

macOS compiles keyboard layouts **only at login**. Copying the file is not
enough by itself.

*  → **Log Out …**
* Log in again.

If the layout still does not appear afterwards, see step 8.

## 4. Open the Keyboard settings

1. Open **System Settings**.
2. Click **Keyboard** in the sidebar.
3. Scroll down to the **Text Input** section and click **Edit…**.

## 5. Add the custom input source

In the **Input Sources** window:

1. Click the **+** button (bottom left).
2. In the search field, type `Hungarian AltGr` (typing `Hungarian` is enough).
3. Select **Hungarian AltGr** in the list.
4. Click **Add**.

The layout appears under the same group as the other Hungarian layouts because
it is based on the Hungarian script. If it is not visible, use the search
field; if it is still missing, see step 8.

## 6. Select the layout

After adding, the layout can be switched to in two ways:

* Click the input-menu icon in the menu bar (usually the flag or a letter
  symbol near the clock) and select **Hungarian AltGr**, or
* press the default shortcut to cycle input sources: **Control + Space**
  (or **⌃ + Space**), which cycles through Hungarian, Hungarian AltGr, etc.

The currently active layout is shown in the menu-bar input menu.

## 7. Test it

Open any text editor (TextEdit is fine) and type:

| You type | You get |
| -------- | ------- |
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
| Option + X | `#` |
| Option + - | `*` |
| Option + C | `&` |
| Option + , | `;` |
| Option + É | `$` |
| Shift + Option + Q … , | same as Option alone (see note) |
| Shift + . | `:` |

`Shift + Option` produces exactly the same character as `Option` alone for
these fifteen keys — e.g. `Shift + Option + Q` → `@`, `Shift + Option + E` →
`\`, `Shift + Option + V` → `@`, `Shift + Option + Y` → `>`. (Remember: both
Option keys behave the same — see the README section on the Right Option
limitation.)

Then verify a few untouched combinations, e.g. `é`, `Shift + 1` → `'`,
`Option + Ü` → `\`, `Option + U` then `A` → `ä`, `Option + A` → `ą`.
The full checklist is in [`test-plan.md`](test-plan.md).

## 8. Restarting applications

Applications that were already running before the layout was added can keep
using the previous input source until they are restarted:

* Fully quit and reopen Terminal, editors, IDEs and browsers after switching
  layouts.
* Some apps remember the input source **per window**, so switch the layout in
  the window where you need it.

A logout/reboot is **required once**, after copying the file. It is *not*
required after adding the input source in System Settings.

## 9. Troubleshooting if the layout does not appear

1. **File location and name.** The file must be exactly
   `~/Library/Keyboard Layouts/Hungarian-AltGr.keylayout` (or in
   `/Library/Keyboard Layouts/`), with the `.keylayout` extension.
2. **Log out/in was skipped.** macOS scans the folder only at login.
3. **File was edited.** If the XML is damaged, macOS silently skips the
   layout. Re-copy the original file from the repository and log out/in again.
4. **Compiler errors.** Open **Console.app**, type `uchr XML compiler` in the
   search field. An error there points to a line in the file.
5. **Search instead of scrolling.** In the Input Sources window, the custom
   layout may be listed under the Hungarian group; the search field finds it
   reliably.

If the layout appears but typing produces nothing, see the Troubleshooting
section of the README.

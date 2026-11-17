# Glossary

One name per idea, on both platforms. Use these words in the UI, the docs, the log, and VoiceOver and Narrator labels.

Shortcuts below are the Mac's. On Windows, ⌘ is Ctrl. Words in a platform's own UI (System Settings, Start, the taskbar) follow that platform.

| Term | Means | Not |
| --- | --- | --- |
| Leader | The global chord that starts a sequence, such as Hyper+Space. Each leader opens its own top-level menu. | trigger, hotkey, prefix, activator |
| Key | A single key pressed after the leader to pick a menu or binding (in the config, a "step key"). | step, shortcut, letter |
| Sequence | Everything from pressing a leader until a binding runs, you cancel, or it times out. | chord chain, session, combo |
| Menu | A named group of keys inside a sequence, such as Applications. It opens more choices and runs nothing itself. | folder, group, submenu, category |
| Binding | A key that runs an action, such as E → Editor. | shortcut, command, item, mapping, hotkey |
| Direct binding | A binding on a global chord, such as Hyper+T → Terminal. It runs at once, with no sequence and no guide. | hotkey, global shortcut, quick key |
| Home | The menu above the leaders, listing every leader and direct binding by group. Back from a leader's menu opens it. | root, top level, start menu, launcher menu |
| Group | In Home, the top-level entries that share the same modifiers, such as Hyper. | layer, modifier set, section |
| Entry | Config-file word for a menu or binding. Never shown in the UI; say "binding" or "menu". | (UI) |
| Action | What a binding does: open an app, open a file or URL, run a command, send a shortcut, choose by app, or nothing. | command, task, handler |
| Built-in action | An action every config has without defining it, named by a binding, such as `window-place-left-half` or `window-focus-left` (spec v11). | preset action, default action, standard command |
| Override | An action in the config with a built-in action's name. It replaces the built-in action everywhere the config names it. | shadow, replace, remap |
| Use Application | The `application` action: focus an app, start it, or both, by its mode. | Open App, launch, activate, switch to |
| Focus or Start | The default mode of Use Application: bring the app forward, starting it first if it isn't running. | open, launch or activate |
| Focus | A mode of Use Application: bring the app forward only if it's running. | activate, switch to |
| Start | A mode of Use Application: start a new copy of the app, even if one is running. | launch, open, new instance |
| Open | The `open` action: open a file, folder or URL in its default app. | launch, browse |
| Run Command | The `exec` action: start a program with arguments. | execute, exec, shell, script |
| Send Shortcut | The `send` action: press one or more key combinations in the app you were using. | send keys, keystroke, type, simulate |
| Context Aware | The `contextual` action: run a different action depending on the app in front. | contextual, conditional, smart action, Choose by App |
| Do Nothing | The `noop` action. | noop, placeholder, empty |
| Show Help | The `help` action: open the Help page of the Bindings window. | help, home, open help |
| Context | A named set of apps, such as "browser", that Context Aware actions check against. | app group, profile, scope |
| App in front | The app that was active when you pressed the leader. Actions go to it. | focused app, frontmost, target app |
| Guide | The small panel that shows where you are in a sequence and which keys come next (HUD in the code). | HUD, overlay, popup, palette, cheat sheet |
| Path | The breadcrumb row at the top of the guide: "Commands › Applications". | breadcrumbs, trail, location |
| Back | Go up one menu (Backspace by default). | up, previous, return |
| Cancel | End the sequence without running anything (Escape by default). | close, abort, exit, dismiss |
| Config file | The one source of truth for bindings and settings. Where it lives: CONFIG-SPEC.md, The file. | settings file, preferences, profile, keymap |
| Reload Config | Read the config file again and use it if it's valid. | refresh, restart, re-read |
| Enabled | Whether the app is capturing leaders. When it's off, no keys are captured. | active, running, paused, armed |
| Permissions | Mac: the Accessibility and Input Monitoring access in System Settings that the app needs to capture keys. Use Apple's names for each one. Windows has no equivalent for keys, but can't control apps running as administrator unless it runs as administrator too. | rights, access, privileges |
| Bindings window | The window for editing bindings, actions and contexts, opened by Edit Bindings… (⌘B). | editor, config editor, settings window |
| Help page | The Help page of the Bindings window: your leaders and direct bindings, the keys that work in every sequence, and links. | manual, documentation, cheat sheet |
| Settings | The Settings window (Settings… ⌘,): Guide, keys, feedback, Open at Login. | Preferences, options |

## Window actions (spec v2)

The UI names below are settled in the spec. The words in each app's forms get their final check with the `ux-design` skill when they ship.

| Term | Means | Not |
| --- | --- | --- |
| Arrange Windows | The `window.arrange` action: lay out the windows used last, on one display, in a grid of columns or rows. | tile, pair, snap |
| Cell | One part of the grid Arrange Windows makes, for one window. | slot, tile, pane |
| Group | A column or a row of cells in that grid. | stack, split |
| Fill | The `window.fill` action: fill the screen's usable area, less the gap, and press again to restore. Not full screen. `window.maximize` is its old name. | maximize, zoom, full screen |
| Focus Window | The `window.focus` action: focus the nearest window in a direction, the next or previous one, the first or last one, one by number, or the window used before. | switch, activate |
| Minimize | The `window.minimize` action. | hide, dock |
| Move Window | The `window.move` action: move by points, or to a spot such as centered. | position, drag |
| Resize Window | The `window.resize` action: resize by points, to a fraction, or step through widths. | size, scale |
| Place Window | The `window.place` action: move and resize in one step, such as Left Half. | snap, tile, layout |
| Preset | A named part of the screen for Place Window, such as Left Half or Top Right. | layout, zone, snap area |
| Swap Window | The `window.swap` action: trade place and size with the window in a direction, the next or previous one, the first or last one, one by number, or the window used before. | exchange, switch, flip |
| Window used before | The window that had focus before the focused one, from the focus history the app keeps while it runs. The `recent` target. | previous window, last window, switch |
| Usable area | The screen without the menu bar and Dock, or the taskbar. Window sizes are fractions of it. | visible frame, work area |
| Anchor | The point of a window that stays put while it resizes, such as its top left corner. | origin, pin, gravity |
| Grow, Shrink | Step a window's width to the next wider or narrower of its widths. | expand, cycle |

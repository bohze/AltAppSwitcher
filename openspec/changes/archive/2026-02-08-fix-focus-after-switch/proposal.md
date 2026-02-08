## Why

After switching applications through AltAppSwitcher, the selected window is shown but does not receive keyboard focus, so typing fails until the user clicks inside the app. This breaks the expected Alt-Tab parity and makes keyboard-first switching unreliable.

## What Changes

- Define required post-switch behavior so the selected target window becomes the active keyboard input target immediately after switching.
- Ensure focus transfer handles common desktop app cases where window activation and input focus can diverge.
- Add explicit acceptance criteria for manual verification (for example, switching to Notepad++ and typing immediately without mouse interaction).

## Capabilities

### New Capabilities

- `switch-target-input-focus`: AltAppSwitcher must bring the selected window to the foreground and ensure it can receive immediate keyboard input after a switch.

### Modified Capabilities

- None.

## Impact

- Affected code: app-switch execution and Win32 focus/activation handling in `Sources/AltAppSwitcher/`.
- Runtime behavior: post-switch user interaction flow (keyboard input immediately after switch).
- Validation: manual verification of switching and typing behavior across representative desktop apps.

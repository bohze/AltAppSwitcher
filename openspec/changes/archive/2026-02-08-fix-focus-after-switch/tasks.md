## 1. Shared Activation Helper

- [x] 1.1 Add a shared helper in `Sources/AltAppSwitcher/` that activates a selected target root window using `GetAncestor(GA_ROOT)`, `BringWindowToTop`, `SetForegroundWindow`, and `SetActiveWindow`.
- [x] 1.2 Implement thread-ID capture and temporary `AttachThreadInput` handling in the helper, with explicit cleanup that always detaches attached threads.
- [x] 1.3 Add activation-outcome verification (`GetForegroundWindow`) and a best-effort retry path when the first activation attempt does not foreground the selected target.

## 2. Integrate Apply Paths

- [x] 2.1 Update App mode apply flow (`ApplySwitchApp` in `Sources/AltAppSwitcher/AppMode.c`) to keep existing restore/order behavior and invoke the shared helper for final target activation.
- [x] 2.2 Update Win mode final apply (`ApplyWin` in `Sources/AltAppSwitcher/WinMode.c`) to use the shared helper instead of direct one-off foreground calls.
- [x] 2.3 Ensure both keyboard-only and mouse-driven apply paths use the same final activation behavior without introducing new config flags.

## 3. Validate Focus Behavior

- [x] 3.1 Build Debug x86_64 (`mingw32-make ARCH=x86_64 CONF=Debug`) and fix any compile/lint issues introduced by the focus changes.
- [x] 3.2 Manually verify immediate typing after switch in Notepad++ and at least one additional desktop app, including a minimized-window restore case.
- [x] 3.3 Validate fallback/cleanup safety by confirming no lingering thread-input attachment side effects after repeated switches.

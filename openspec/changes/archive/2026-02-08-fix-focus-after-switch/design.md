## Context

AltAppSwitcher currently brings the chosen app window into view, but in App mode the apply path relies on Z-order updates (`DeferWindowPos`) and thread-input attachment without a final explicit activation of the selected target window. As a result, users can see the switched app (for example Notepad++) but keyboard input does not go there until a mouse click. Existing comments already note that naive `SetFocus` on top-level windows is undesirable because it can override child-control focus behavior.

## Goals / Non-Goals

**Goals:**
- Ensure the selected switch target becomes the active foreground window and can receive keyboard input immediately after apply.
- Preserve existing behavior for grouped windows (window stacking order and optional restore of minimized windows).
- Use a consistent activation strategy across App mode and Win mode paths.

**Non-Goals:**
- Redesign window grouping or selection logic.
- Guarantee focus behavior for OS-restricted cases outside app control (for example, foreground lock, privilege boundaries).
- Introduce new user-facing settings for focus behavior in this change.

## Decisions

1. Add a dedicated activation helper for switch targets.
Use a single helper (shared by App mode and Win mode) to perform foreground activation and input-focus handoff for the selected top-level target window.
Alternatives considered:
- Patch only `ApplySwitchApp` with one extra `SetForegroundWindow` call. Rejected because focus behavior would remain inconsistent across switch paths.
- Keep per-call ad hoc focus APIs in each mode. Rejected due duplicated logic and harder debugging.

2. Use an explicit post-switch activation sequence instead of relying on Z-order side effects.
After existing ordering/restore logic, run a deterministic activation sequence on the selected target root window:
- Resolve root target window (`GetAncestor(..., GA_ROOT)`).
- Capture current foreground window and involved thread IDs.
- Temporarily attach input where needed (`AttachThreadInput`) so activation calls are allowed in common cases.
- Call activation APIs in order (`BringWindowToTop`, `SetForegroundWindow`, `SetActiveWindow`).
- Detach thread input in a guaranteed cleanup path.
- Validate outcome with `GetForegroundWindow` and fall back to a best-effort retry path if activation did not land.
Alternatives considered:
- `SetFocus` as primary. Rejected due cross-thread limits and child-focus regressions already observed.
- UI Automation `SetFocus` as primary. Rejected due known inconsistency in comments and current code history.

3. Keep group ordering logic, then activate only the final selected target.
`ApplySwitchApp` keeps its current responsibility for restoring and ordering all windows in the selected group, but it hands off final keyboard activation to the new helper for the chosen target window only.
Alternative considered:
- Activate every window during ordering. Rejected because this increases focus churn and unpredictability.

4. Reuse the same helper in Win mode final apply.
`ApplyWin` should use the shared activation helper instead of a single direct `SetForegroundWindow` call so both modes enforce the same input-focus contract.
Alternative considered:
- Leave Win mode unchanged. Rejected because it keeps two different focus behaviors and complicates maintenance.

## Risks / Trade-offs

- [Foreground restrictions (OS policy, privilege boundary, elevated windows)] -> Mitigation: best-effort activation with thread attach + validation, and explicit manual verification against elevated/non-elevated scenarios.
- [Incorrect thread attach/detach pairing can leave threads coupled] -> Mitigation: track attach state flags and always detach in one cleanup block.
- [Behavior change for multi-window groups] -> Mitigation: keep existing ordering logic unchanged and limit new behavior to final target activation.
- [Potential visual flicker from restore/activation calls] -> Mitigation: preserve current restore-minimized configuration behavior and avoid unnecessary extra window state transitions.

## Migration Plan

1. Implement the shared target-activation helper in AltAppSwitcher runtime code.
2. Update App mode apply flow to call the helper after group ordering.
3. Update Win mode apply flow to call the same helper.
4. Manually validate keyboard focus immediately after switch in representative apps (Notepad++, Explorer, browser, elevated app where applicable).
5. If regressions appear, revert to previous activation call path while keeping helper behind compile-time guard for diagnosis.

## Open Questions

- Should a UIA-based fallback be enabled only when foreground activation succeeds but text input still does not land in target controls?
- Should activation success criteria accept owned/child foreground windows of the target root as equivalent success?

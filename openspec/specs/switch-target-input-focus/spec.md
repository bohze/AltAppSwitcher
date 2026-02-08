## Purpose

Define focus and activation requirements for AltAppSwitcher so the selected target app receives immediate keyboard input after switching.

## Requirements

### Requirement: Selected switch target receives immediate keyboard input
AltAppSwitcher SHALL make the selected target application's root window the active foreground window at switch completion, and it MUST allow keyboard input without additional mouse interaction.

#### Scenario: Switch to Notepad++ and type immediately
- **WHEN** user switches to Notepad++ using AltAppSwitcher and the switch action completes
- **THEN** Notepad++ is foreground and typed characters appear immediately in its editor without clicking

#### Scenario: Keyboard-only switch remains keyboard-first
- **WHEN** user performs selection and apply entirely by keyboard
- **THEN** the selected target accepts subsequent keyboard events immediately after the switch

### Requirement: Focus transfer works for restored minimized targets
When the selected target window is minimized and restore-on-switch is enabled, AltAppSwitcher SHALL restore that window and MUST complete foreground/input activation in the same switch action.

#### Scenario: Restored window is immediately interactive
- **WHEN** selected target is minimized and AltAppSwitcher restores it during switch
- **THEN** the restored target becomes foreground and receives immediate keyboard input without mouse interaction

### Requirement: Focus transfer handles activation failure paths safely
If the initial activation sequence does not place the selected target in foreground, AltAppSwitcher SHALL perform a fallback activation attempt before ending the switch, and it MUST release any temporary thread-input attachment used during focus transfer.

#### Scenario: Activation retry and cleanup
- **WHEN** the first activation attempt does not make the selected target foreground
- **THEN** AltAppSwitcher retries activation and completes without leaving thread-input attachment active between unrelated threads

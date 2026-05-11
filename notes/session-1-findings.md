# Session 1 findings (2026-05-10)

## $ITERM_SESSION_ID format

- Full value: `w<W>t<T>p<P>:<UUID>` (window/tab/pane prefix + colon + UUID).
- Example: `w1t0p0:3545859D-C3B7-46E6-AA0C-3898E13BAB0C`.
- The UUID after the colon is byte-equal to AppleScript `unique id of session`.
- **Transform required:** strip everything up to and including the first colon before comparing.
- Falls back to: `${ITERM_SESSION_ID#*:}` in bash, `iterm_session_id.split(":", 1)[-1]` in Python.

## AppleScript probes verified

```applescript
# List all sessions:
tell application "iTerm"
  repeat with w in windows
    repeat with t in tabs of w
      repeat with s in sessions of t
        -- (unique id of s) is the matchable UUID
      end repeat
    end repeat
  end repeat
end tell

# Focus a session by UUID:
tell application "iTerm"
  repeat with w in windows
    repeat with t in tabs of w
      repeat with s in sessions of t
        if unique id of s is "<UUID>" then
          select s
          activate
        end if
      end repeat
    end repeat
  end repeat
end tell

# Get currently frontmost session id (for Stop suppression):
tell application "iTerm" to return unique id of current session of current window
```

## TTY availability inside hooks (TBD)

- `tty` returned "not a tty" when run via Claude Code's Bash tool. The hook may run in a different environment.
- Verify by capturing actual hook payloads — relying on payload's `session_id` or interactivity flag is safer than `isatty()`.

## Existing ~/.claude/settings.json (pre-modification)

```json
{
  "env": { "DISABLE_AUTOUPDATER": "1" },
  "permissions": { "allow": [".*"] },
  "model": "opus[1m]",
  "enabledPlugins": { "slack@claude-plugins-official": true },
  "skipDangerousModePermissionPrompt": true
}
```

No existing `hooks` key — adding hooks is purely additive. Still backup before editing.

## Outstanding for Phase 1

- [ ] Capture real hook payload shape (Notification, Stop, SubagentStop). Need to wire the script first or use temp `tee`.
- [ ] Confirm `terminal-notifier -execute` can run a multi-line AppleScript or whether we need to inline it / write a helper script.
- [ ] Install `terminal-notifier` via Homebrew.

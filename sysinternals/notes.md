# Troubleshooting with the Windows Sysinternals Tools

## Tools direct access

1. `https://live.sysinternals.com/procexp.exe`
2. `\\live.sysinternals.com\tools\procexp.exe`

## Procexp

### Symbols setup

1. Download dbghelp.dll from `https://developer.microsoft.com/en-us/windows/downloads/windows-sdk/` debugging tools.
2. Setup symbol path to `c:\symbols\private;srv*c:\symbols\public*https://msdl.microsoft.com/download/symbols` in _NT_SYMBOL_PATH

### Procexp tips & tricks

- `Space` to pause refresh, `F5` for manual refresh to see the process diff (new and killed processes, possible delta columns)
- `Ctrl-c` on row to copy in TSV format
- Right click menu interesting items:
  - `Set affinity` - restrict process to specified cores (can keep a process running but it would occupy only a single core)
  - `Window` - bring the process'es window up front
  - `Kill process` - can kill system-critical processes as well, results in blue screen
  - `Restart` - would keep the arguments but loose environment, inherited handles and security context
  - `Suspend` - keep proces in memory but don't run it, useful for malware that monitors that it's service is in memory and keeps restarting it in case it's killed
  - `Debug` - launches `HKLM\Software\Microsoft\Windows NT\CurrentVersion\AeDebug -p <PID>`. Terminating debugger without detaching kills the debugged process.
  - `Create dump` - creates full or minidump without terminating a process

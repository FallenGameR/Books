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
- Process columns:
  - `Image/Autostart location` - what causes process to auto-start
  - `Comment` - persists between procexp runs, one for a unuque image path

## Procmon

- `BUFFER OVERFLOW` is not an error result, this is how usually WinAPI is used to find out needed buffer length
- Filter rules regarding the same column are joinsed with OR, filter rules about different columns are joined with AND
- `-is` operation allows to select value from the combobox (no need to do string matches with `contains`), but doesn't work on the `Path` column
- It is possible to set `Category is Write` for all the events that modify the file system
- Highlight rules `Ctrl-H` work the same way as the filter rules with regard to AND and OR (can be not intuitive)
- `F4` and `Shift-F4` for quick navigation between highlighted items, also an event properties have navigation buttons
- `F6` and `Shift-F6` for quick navigation between bookmarked items, `Ctrl-B` to toggle a bookmark

### Injecting custom debug traces from your program into Procmon

- Enable profiling events toggle
- Use native or manged API from https://github.com/Wintellect/ProcMonDebugOutput (that's just a wrapper, if you want you can just directly emmit ETW messages in the corresponding log)
- Optionally use `Exclude events before` and `Exclude events after` to remove unnecessary events that happen outside of your debug windows
- The same machanism is used in the `ProcDump` tool, btw

## ProcDump

Creates mini or full process dump on a condition.

- `PID` would create minidump in the current folder
- `-w partial_process_name` would poll every second for the process to be created and then would create a minudump (it would not be able to capture the initial second of the process lifetime)
- `-e 1 -f "" -x c:\dumps pwsh.exe -File Script.ps1` would start the process and attach the debugger to it
  - To debug a process whenever it is executed create subkey `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\Process.exe`. In that subkey create string value `Debugger` and place all the parameters except the last two - executable name and arguments, since they would be supplied automatically.
  - Instead of executable name you can specify UWP package name as can be resolved via `HKCU\Software\Classes\ActivatableClasses\Package` and optionally add `!AppName` to it.
- `-i` in the current folder would attach procdump as the AutoEnabled debugger making it trigger for every un-captured exception. `-u` to unregister.

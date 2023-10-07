# Troubleshooting with the Windows Sysinternals Tools

## Direct access

1. `https://live.sysinternals.com/procexp.exe`
2. `\\live.sysinternals.com\tools\procexp.exe`

## Procexp symbols setup

1. Download dbghelp.dll from `https://developer.microsoft.com/en-us/windows/downloads/windows-sdk/` debugging tools.
2. Setup symbol path to `c:\symbols\private;srv*c:\symbols\public*https://msdl.microsoft.com/download/symbols` in _NT_SYMBOL_PATH


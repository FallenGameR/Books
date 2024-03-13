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

## PsGetSid

```ps1
# Get SID of the current machine
psgetsid -nobanner

# Get SID of local administrator that would contain the SID of the local machine
psgetsid -nobanner administrator

# Works only in Active Directory, since there is no Service Operators group on a workstation
psgetsid -nobanner $Env:LOGONSERVER S-1-5-32-549

# List known SIDs for standart groups
0x220..0x240 | %{ psgetsid -nobanner S-1-5-32-$psitem 2>$null }

# Names of first 10 local users and groups
$machine = (psgetsid -nobanner $Env:LOGONSERVER)[2]
1000..1009 | %{ psgetsid -nobanner $machine-$psitem 2>$null }
```

## PsService

```ps1
# List w32time service settings
» psservice config w32time

PsService v2.26 - Service information and configuration utility
Copyright (C) 2001-2023 Mark Russinovich
Sysinternals - www.sysinternals.com

SERVICE_NAME: W32Time
DISPLAY_NAME: Windows Time
Maintains date and time synchronization on all clients and servers in the network. If this service is stopped, date and time synchronization will be unavailable. If this service is disabled, any services that explicitly depend on it will fail to start.
        TYPE              : 20 WIN32_SHARE_PROCESS
        START_TYPE        : 3  DEMAND_START
        ERROR_CONTROL     : 1  NORMAL
        BINARY_PATH_NAME  : C:\WINDOWS\system32\svchost.exe -k LocalService
        LOAD_ORDER_GROUP  :
        TAG               : 0
        DEPENDENCIES      :
        SERVICE_START_NAME: NT AUTHORITY\LocalService
        FAIL_RESET_PERIOD : 86400 seconds
        FAILURE_ACTIONS   : Restart     DELAY: 60000 seconds
                          : Restart     DELAY: 120000 seconds
                          : None        DELAY: 0 seconds

# Who is allowed to do what with a service
psservice security w32time

PsService v2.26 - Service information and configuration utility
Copyright (C) 2001-2023 Mark Russinovich
Sysinternals - www.sysinternals.com

SERVICE_NAME: W32Time
DISPLAY_NAME: Windows Time
        ACCOUNT: NT AUTHORITY\LocalService
        SECURITY:
        [ALLOW] NT AUTHORITY\SYSTEM
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                Pause/Resume
                Start
                Stop
                User-Defined Control
                Read Permissions
        [ALLOW] BUILTIN\Administrators
                All
        [ALLOW] NT AUTHORITY\INTERACTIVE
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                User-Defined Control
                Read Permissions
        [ALLOW] NT AUTHORITY\SERVICE
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                User-Defined Control
                Read Permissions
        [ALLOW] NT AUTHORITY\LOCAL SERVICE
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                Start
                User-Defined Control
                Read Permissions
        [ALLOW] NT AUTHORITY\LOCAL SERVICE
                Query Config
                Interrogate
                Enumerate Dependents
                Stop
                Read Permissions
        [ALLOW] NT SERVICE\autotimesvc
                All

# Find dns servers in the current workgroup                
» psservice find "dns server"
```

## VmMap

- tracing feature injects dll into a process to trace all allocations
- graphs show relative region sizes, they don't show fragmentation and memory positions
- can capture memory snaphots for diff - periodic, pausable with ctrl+space, manual with F5

## DebugView

- can start agent on one machine and view debug output on another one

## SigCheck

```ps1
# Show embedded app manifest
sigcheck -m c:\tools\xts\xts.exe -nobanner

# List cat file contents
sigcheck -d c:\tools\DriScripts\DriScripts.cat -nobanner

# List cert store contents -t (machine) -tu (user)
sigcheck -tu -nobanner
```

## AccessChk

```ps1
# Check who has what access for a file or folder contents
accesschk c:\windows\explorer.exe -nobanner

# Check who has what access for a folder itself
accesschk -d c:\windows\ -nobanner
accesschk -d \\reddog\builds\branches\git_azure_compute_move_master_latest\ -nobanner

# Check access rights for a local network share and list all the shares
accesschk -h share -nobanner
accesschk -h * -nobanner

# Check access rights for a local windows service
accesschk -c w32time -nobanner

# Check recuresivelly current folder checking that current user has read access for all elements
accesschk -rs $env:USERNAME .

# Check recuresivelly program files showing what folders prohibit current user of anything
accesschk -ns $env:USERNAME 'C:\Program Files\'

# Looks like -o option may use usefil for resolving the user name in domain scenarios
# And -i option for looking up where inheritance rules are broken and access is set explicitly
```

## AccessEnum

Checks who have access to a particular folder. Network share paths are not supported.

## Streams

Allow to see if there are alternative streams in the files.

```ps1
# Write and read alternative stream from a file
cmd /c "echo alloha > test.txt:altdata"
cmd /c "more < test.txt:altdata"
streams .\test.txt -nobanner

# Recuresivelly remove marker that is added to the files downloaded from the internet
streams -s -d downloads
```

## FindLinks

Finds all the hardlinks that point to that file

```ps1
# Will show the link to WinSxS (side-by-side component store)
findlinks C:\WINDOWS\system32\notepad.exe -nobanner

# To do the same in latest Windows OSes (shows more links for some reason)
fsutil hardlink list C:\WINDOWS\system32\notepad.exe 
```

## DiskView

Shows cluster view of files. Uses defragmentation API. Small files less that 4kb are stored directly in the `Master File Table`.

## Binary file compare

This is a feature of Windows, but I learned about it only via SysInternals.

```ps1
"me mo" > a.txt
"me mi" > b.txt
fc.exe /b a.txt b.txt
```

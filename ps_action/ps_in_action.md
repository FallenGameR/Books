# Powershell in Action, 3rd ed

## Chapter 1 - Wellcome

```ps1
# Check how parameters are bound
Trace-Command -Name ParameterBinding -Option All -Expression {123 | Write-Output} -PSHost

# Debug object structure
ls | Format-Custom -depth 1

# Non-terminating output to a string
ls | Format-Table -AutoSize | Out-String -Stream > file.txt
```

## Chapter 2 - Types

```ps1
# Expand string from a constant
$ExecutionContext.InvokeCommand.ExpandString('a is $a')

# Pseudo arrays - every object has .Count
([object]2).Count

# Arrays are auto-expandable
$a = 1,2,3  # [object[]]
$a += 4,5   
$a.Length   # [object[]]

# Nice consice array print
"$a"

# Direct casting into a single element array
[int[]]'1'

# Specifying generic constraints
[system.collections.generic.dictionary[string,int]]

# Convert string into integers
[int[]] [char[]] "Hello"

# Scriptblock parameters
ls *.xlm | ren -path {$_.Name} -new {$_.Name -replace '\.xml$', '.txt'} -whatif
```

## Chapter 3 - Operators & Expressions

```ps1
# Assignment of hash fields with type conversion
$e = @{}
$e.level, [int]$e.lower, [int]$e.upper = -split $line 

# Wildcards in file names
ls [fm]*.[c-s] -Recurse

# Replace operator
${c:old.txt} -replace 'is (red|blue)', 'was $1' > new.txt
# $number - matched group value by index
# ${name} - matched group value by name
# $$      - escape $
# $&      - entire match
# $`      - before match
# $'      - after match
# $+      - last match
# $_      - entire argument

# Split operator accepts arguments like a regex
'a*b*c' -split '*',0,'SimpleMatch'
# where to split
# limit on the result count
# type of the regex match, don't use regexes here

# Split based on a function that works on chars
$count = @(0) # trick for scriptblock to modify the value by reference
'a,b,c,d,e,f,g' -split {($_ -eq ',') -and (++$count[0] % 2 -eq 0)}
# a,b  c,d  e,f  g
```

## Chapter 4 - Advanced Operators & Variables

```ps1
# Array slicing by index arrays
$indexes = 0,3,5
(1..9)[($indexes | % {$_ + 1})]

# Reverse array
$a = 1..9
"$($a[($a.Length)..0])"
# 9 8 7 6 5 4 3 2 1

# Using statement, PS5+
using assembly  System.Windows.Forms
using namespace System.Windows.Forms

## Attribute constrained variables
[ValidateLength(0,5)] [string] $a = ''

# Splatting variables, named arguments
function s{param($x,$y,$z) "$x,$y,$z,$args"}
$p = @{x="first"; y="second"}
s -z "third" @p 1 2 3
# first,second,third,1 2 3
```

## Chapter 5 - Control Flow

```ps1
# Access iterators directly
foreach( $i in 1..10 ){ [void] $foreach.MoveNext(); $i + $foreach.Current }

# Access switch iterator directly (useful for arg parser)
switch( $options ) {
  '-force' { $force = $true }
  '-file' { [void] $switch.MoveNext(); $file = $switch.Current }
}

# Use named tags to break particular loop
$tag = 'outer'
:outer while( $true ) {
  while( $true ) { break $tag }
}

# Start, Process, End scripts in regular foreach
gps svchost | foreach {$h=0} {$h+=$_.handles} {$h}

# Pass arguments for method invocations in foreach
'test', 'me' | foreach Replace -ArgumentList 'st', 'hi'

# Access input argument directly
function sum{
  $total = 0
  while( $input.MoveNext() ) {
    $total += $input.Current
  }
  $total
}

# WhatIf syntax support
# [CmdletBinding(SupportsShouldProcess=$true)] # adds -WhatIf switch
# if( $PSCmdlet.ShouldProcess($target, $operation) ) { $actually_do_the_processing_here }

# Assign argument name to a property from the piped in input
function test{ param([Parameter(ValueFromPipelineByPropertyName=$true)] $DayOfWeek) process { $DayOfWeek }}
Get-Date | test

# For type checking and loading classes from a module, auto-load has its limitations
# using -module <name>

# It should be possible to load just a few functions or aliases from a module
# and extract them into a custom object (this exact command doesn't work though)
$module = Import-Module PsToolset -Alias parse -AsCustomObject

# To debug module loading
ipmo PSToolset -Verbose

# Unassign a variable
Remove-Item -verbose variable:total
```

## Chapter 10 - Metaprogramming

```xml
<Types>
  <Type>
    <Name>System.Array</Name>
    <Members>
      <ScriptMethod>
        <Name>Sum</Name>
        <Script>
          $r = $null
          foreach( $e in $this ) { $r += $e }
          $r
        </Script>
      </ScriptMethod>
    </Members>
  </Type>
</Types>
```

```ps1
# Add Sum extension method to an array
Update-TypeData SumMethod.ps1xml
(1,2,3,4,5).Sum()
(@{a=1},@{b=2},@{c=3}).Sum()

# Dynamic module compiled in memory
$bin = Add-Type $csCode -PassThrough
$bin.Assembly | Import-Module
```

## Chapter 11 - Remoting

```ps1
# Enable PS or PS core remoting
Enable-Psremoting -Verbose -Confirm

# Trust a computer you connect to, this list is automated for domains
# You can specify * if you trust all destinations (at home closed network)
# Successful HTTPS euthentication also makes computer trusted via a shared cert root
Set-Item wsman:\localhost\client\TrustedHosts "alexko-remote"

# Note about script blocks used in remoting
# - they are compiled locally to capture syntax errors
# - they are more hardened against injection attacks

# Download file from remote
Copy-Item -Path test.txt -FromSession $s

# Upload file to remote
Copy-Item -Path test.txt -Destination C:\temp -ToSession $s

# Implicit remoting with a proxy dynamically created module
Import-PsSession -Session $s -CommandName Get-Bios
Get-Bios # called locally, but executed remotelly via Session $s

# Find location of a module
(gcm source).Module.Path

# Save remote command as a module
Export-PsSession -OutputModule bios -Session $s -Type function -CommandName Get-Bios -AllowClobber
Import-Module bios
Get-Bios # will start to create a session to the remote computer, but then it'll fail since command isn't there yet

# One way copy of var into remote session
Invoke-Command -Session $s -ScriptBlock {"myvar is $using:myvar"}

# Stdio and stdout in remoting are not redirected properly for native code
# For PS scripts themselves you'll need switch from [Console] API to:
# NOTE: Returned objects are deserialized with DEPTH of 1 by default
remote> Write-Host "hi"
remote> Read-Host "input"

# PS remoting stack
# - TCP/IP - routing
# - HTTPS/HTTP (basic auth) - auth and encryption
# - SOAP - xml deserialization
# - WS-MAN - implemented by WinRM, starts host process, IIS can be used if many incoming connection are expected and it is expensive to create a process per user (this is a custom configuration scenario, no process isolation)
# - MS-PSRP - interpretation by the PS itself
Set-PsSessionConfiguration Microsoft.Powershell -ShowSecurityDescriptorUI

# Remote commands to Hyper-V VMs, this bypasses network and firewalls, requires admin rights on both sides
Get-VM
$cred = Get-Credential -Credential VMName\Administrator
Invoke-Command -VMName VMName -ScriptBlock {Get-Process} -Credential $cred # or use -VMGuid instead of name
```

## Chapter 13 - Scheduled Jobs

```ps1
# Register scheduled job
Register-SheduledJob -Name DailyJob -ScriptBlock {Get-Process} -Trigger $daily -RunNow

# List scheduled jobs, they use the following paths:
# - Task Scheduler\Library\Microsoft\Windows\Powershell\ScheduledJobs - registrations
# - $home\AppData\Local\Microsoft\Windows\Powershell\ScheduledJobs - last 32 outputs, controlled by -MaxResultCount
Get-Job -Name DailyJob -Newest 2

# Start job outside of schedule - this doesn't preserve output on disk
Start-Job -DefinitionName DailyJob

# Setup scheduled job options, can hide from Task Scheduler UI
# for RunElevated you'll need to provide -Credential in Register- or Set-
New-ScheduledJobOption
```

## Chapter 14 - Errors

```ps1
# Redirect errors to a specific variable
ls -ErrorVariable errs

# Don't capture errors at all, even in the specific variable
ls -ErrorVariable errs -ErrorAction ignore

# Scoped error action preference override
& {
  $ErrorActionPreference = 'Stop'
  ls
}

# List event logs, PS5 only
Get-EventLog -List

# Read entry from log, PS5 only
# Other useful switches are -Index, -InstanceId, -Newest, -MaximumSize
Get-EventLog -LogName Application -Message '*Defender*' -After 'April 30/2022'

# List event logs, works everywhere
# IMPORTANT: can open classic *.evt files as well
Get-WinEvent -ListLog Microsoft-Windows-Powershell*

# Find event
$start = (Get-Date).AddDays(-2)
$end = $start.AddDays(1)
Get-WinEvent -FilterHashtable @{LogName='Application'; StartTime=$start; EndTime=$end}


```
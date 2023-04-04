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
```

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

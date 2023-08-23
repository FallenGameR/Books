# Notes

## Hide some members or add scripted properties without xml

- [Source](https://powershellexplained.com/2016-10-28-powershell-everything-you-wanted-to-know-about-pscustomobject/)
- Would not work for `psobject` altough they should be synonims

```ps1
$myObject = [PsCustomObject] @{
    PSTypeName = "My.Object"
    Name       = "Test"
    Language   = "Powershell"
    State      = "Washington"
}

# Hide display of some members
$params = @{
    TypeName = "My.Object"
    DefaultDisplayPropertySet = "Name", "Language"
}
Update-TypeData @params
$myObject

# Add script property
$params = @{
    TypeName = "My.Object"
    MemberType = "ScriptProperty"
    MemberName = "UpperCaseName"
    Value = {$this.Name.toUpper()}
}
Update-TypeData @params
$myObject.UpperCaseName

# Validate that object is of that custom type
# param( [PSTypeName('My.Object')] $Data )
```

## Powershell remoting

- [Remoting setup and troubleshooting guide](https://github.com/devops-collective-inc/secrets-of-powershell-remoting/blob/master/SUMMARY.md)

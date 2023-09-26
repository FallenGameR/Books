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

## Event log

App\Powershell-Core\Operational and App\Windows PowerShell - log every command

## Certificates

- Execution policy that can by bypass at any time is not a security boundary (regarding Set-ExecutionPolicy).
- Scripts signed by a trusted authority (publisher) are automatically trusted and executed without interruptions (regarding signing).
- [Azure DevOps pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/architectures/devops-pipelines-baseline-architecture?view=azure-devops) - can be used to automatically sign your code and perform various tests on different deployment stages

```ps1
# Create certificate
New-SelfSignedCertificate

# Sign with a certificate
Set-AuthenticodeSignature
```

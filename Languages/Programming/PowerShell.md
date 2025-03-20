# PowerShell

## Brief Definition
PowerShell is a task automation and configuration management framework from Microsoft, consisting of a command-line shell and scripting language. It's built on .NET and provides powerful features for system administration and automation.

## Historical Context
Released in 2006, PowerShell was designed to replace the older Windows Command Prompt and provide a more powerful scripting environment. It has evolved through multiple versions and is now cross-platform, supporting Windows, Linux, and macOS.

## Core Concepts
- Pipeline-based command chaining
- Object-oriented output
- Cmdlets (Command-Let)
- Modules and snap-ins
- Remoting capabilities
- Error handling
- Regular expressions
- XML handling
- WMI integration
- Active Directory integration

## Implementation
```powershell
# Function definition
function Get-Greeting {
    param(
        [string]$Name
    )
    return "Hello, $Name!"
}

# Pipeline example
Get-Process | Where-Object { $_.CPU -gt 10 } | Sort-Object CPU -Descending

# Error handling
try {
    Get-Content "nonexistent.txt"
} catch {
    Write-Error "File not found: $_"
}
```

## Examples
```powershell
# File operations
Get-ChildItem -Path C:\ -Recurse -Filter *.log | 
    Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-7) }

# System administration
Get-Service | Where-Object { $_.Status -eq 'Running' } |
    Select-Object Name, Status

# Active Directory
Get-ADUser -Filter * | 
    Where-Object { $_.Enabled -eq $true } |
    Select-Object SamAccountName, EmailAddress
```

## Advantages and Limitations
Advantages:
- Powerful system administration
- Object-oriented pipeline
- Rich set of cmdlets
- Cross-platform support
- Active Directory integration
- Strong community
- Regular updates

Limitations:
- Windows-centric features
- Learning curve for complex scripts
- Performance overhead
- Limited GUI capabilities
- Security restrictions
- Version compatibility issues
- Some features require elevated privileges

## Related Concepts
- System Administration
- Windows Management
- Automation
- DevOps
- Active Directory
- Windows Server
- Configuration Management 
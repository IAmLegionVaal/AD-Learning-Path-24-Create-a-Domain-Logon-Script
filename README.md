# AD Learning Path 24 — Create a Domain Logon Script

## Objective
Deploy a small, controlled domain logon script from SYSVOL and validate execution without embedding credentials or adding slow, fragile work to user sign-in.

## Prerequisites
- Healthy SYSVOL
- Test user and client
- Group Policy Management
- A harmless script objective

## Setup
1. Create a dedicated folder under `\\corp.lab\SYSVOL\corp.lab\scripts`.
2. Add a script that writes the current user, computer, and timestamp to a local log.
3. Ensure domain users can read the file and only administrators can modify it.
4. Configure a User Configuration logon-script policy in a test GPO.
5. Link the GPO to the test Users OU.
6. Run policy refresh, sign out/in, and verify the local execution log.
7. Repeat the sign-in to confirm the script is idempotent and fast.

```powershell
$ScriptRoot = '\\corp.lab\SYSVOL\corp.lab\scripts\LearningPath'
New-Item -Path $ScriptRoot -ItemType Directory -Force | Out-Null

@'
$Log = Join-Path $env:LOCALAPPDATA 'AD-Learning-Path\logon.txt'
New-Item -Path (Split-Path $Log) -ItemType Directory -Force | Out-Null
"{0:o} User={1} Computer={2}" -f (Get-Date),$env:USERNAME,$env:COMPUTERNAME |
    Add-Content -Path $Log
'@ | Set-Content -Path "$ScriptRoot\Logon.ps1" -Encoding UTF8
```

## Validation
```powershell
Get-ChildItem '\\corp.lab\SYSVOL\corp.lab\scripts\LearningPath'
gpupdate.exe /force
gpresult.exe /h "$env:TEMP\gpresult.html"
Get-Content "$env:LOCALAPPDATA\AD-Learning-Path\logon.txt" -Tail 10
```

## Evidence
Store the script path and permissions, GPO configuration, `gpresult` report, local log, repeated-sign-in test, offline test, troubleshooting, and final result under `evidence/`.

## Troubleshooting
- Script does not run: inspect GPO scope, user configuration, execution policy, and the GroupPolicy Operational log.
- Slow sign-in: remove long waits and network-dependent processing.
- SYSVOL access issue: review read versus modify permissions.

## Security notes
Never embed credentials or tokens. Treat SYSVOL modification rights as privileged because scripts can execute for many users.

## Cleanup
Disable or unlink the test GPO, wait for policy refresh, then remove the script after confirming no dependency remains.

## References
- Microsoft Learn: Group Policy logon scripts
- Microsoft Learn: SYSVOL

## Next activity
`AD-Learning-Path-25-Create-a-Group-Policy-Central-Store`

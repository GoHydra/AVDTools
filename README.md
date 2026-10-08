# AVD Tools

General Azure Virtual Desktop and Windows utilities. These scripts do not require Hydra.

## Script catalog

| Script | Purpose | Run from / requirements |
| --- | --- | --- |
| [Get-ActiveAvdUniqueUserCount.ps1](Get-ActiveAvdUniqueUserCount.ps1) | Counts distinct user UPNs across AVD sessions; includes active and disconnected sessions by default. | PowerShell 5.1+ with Az.Accounts and Az.DesktopVirtualization, authenticated to Azure with session-read access. |
| [Get-InstalledSoftware.ps1](Get-InstalledSoftware.ps1) | Lists installed software, versions, dates, and publishers from machine and current-user uninstall registry keys. | Windows PowerShell on the machine to inventory. HKCU reflects the account running the script. |
| [Set-OSVersionTag.ps1](Set-OSVersionTag.ps1) | Copies the OS version reported by AVD onto the associated Azure VM as a tag. | Azure PowerShell session or a PowerShell timer-triggered Function App, with Az.Accounts and Az.Resources. |
| [Set-AVDAgentVersionTag.ps1](Set-AVDAgentVersionTag.ps1) | Copies the agent version reported by AVD onto the associated Azure VM as a tag. | Same Azure execution context as the OS version tag script. |

## Count users

Authenticate with `Connect-AzAccount` first. With no scope arguments, the script uses the current subscription.

```powershell
.\Get-ActiveAvdUniqueUserCount.ps1
.\Get-ActiveAvdUniqueUserCount.ps1 -AllSubscriptions
.\Get-ActiveAvdUniqueUserCount.ps1 -SubscriptionId '<subscription-id>' -ShowSessions
.\Get-ActiveAvdUniqueUserCount.ps1 -SessionState Active -QuietCount
```

`-SubscriptionId` accepts multiple IDs. `-ShowSessions` displays the matching sessions; `-QuietCount` returns only the count.

## Inventory installed software

Edit the settings at the top of `Get-InstalledSoftware.ps1`. CSV export is enabled by default, writes to `C:\temp`, and generates a timestamped filename unless `$CsvFileNameOverride` is set. Set `$ExportToCsv = $false` for console output only.

## Synchronize version tags

Both scripts query AVD session-host metadata through Azure APIs; they do not inspect the guest directly. Their identity needs access to read host pools/session hosts and update VM tags. They reuse an existing Az context or attempt managed-identity authentication when no context exists.

| Environment variable | Default | Meaning |
| --- | --- | --- |
| `AVD_TAG_WHATIF` | `true` | Preview changes. Set to `false` to write tags. |
| `AVD_TAG_ALL_SUBSCRIPTIONS` | `false` | Scan all enabled subscriptions visible to the identity. |
| `AVD_TAG_SUBSCRIPTION_IDS` | Unset | Comma- or semicolon-separated subscription IDs; otherwise uses the current subscription when all-subscriptions mode is off. |
| `AVD_OS_VERSION_TAG_NAME` | `OSVersion` | Tag name for the OS script. |
| `AVD_UNKNOWN_OS_VERSION_TAG_VALUE` | `Unknown` | Fallback when OS version is unavailable. |
| `AVD_AGENT_VERSION_TAG_NAME` | `AVD-AgentVersion` | Tag name for the agent script. |
| `AVD_UNKNOWN_AGENT_VERSION_TAG_VALUE` | `Unknown` | Fallback when agent version is unavailable. |

Example preview from an authenticated PowerShell session:

```powershell
$env:AVD_TAG_SUBSCRIPTION_IDS = '<subscription-id>'
$env:AVD_TAG_WHATIF = 'true'
.\Set-OSVersionTag.ps1
.\Set-AVDAgentVersionTag.ps1
```

For a Function App, configure these values as application settings and supply the timer trigger and module dependencies separately; this folder contains the script bodies, not a complete Function App project.

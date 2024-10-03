# VALENTIN PARIS - SCRIPTS

This page contains some useful scripts from Valentin Paris, Customer Engineer at Microsoft.

```powershell
```

# GATHER ALL TYPES OF FORWARDING CURRENTLY IN PLACE

```powershell
#GATHER ALL TYPES OF FORWARDING CURRENTLY IN PLACE
 
# Define output CSV file path
$outputFile = "C:\ForwardingReport.csv"
 
# Initialize an empty array to hold results
$results = @()
 
# 1. Remote Domain Auto Forward Settings
$remoteDomains = Get-RemoteDomain | Select-Object @{Name="ForwardingType";Expression={"Remote Domain"}}, DomainName, @{Name="ForwardingEnabled";Expression={$_.AutoForwardEnabled}}
 
# Add Remote Domain results to the array
$results += $remoteDomains
 
# 2. Inbox Rules Auto Forward
$inboxRules = Get-Mailbox -ResultSize Unlimited | ForEach-Object {
   $mbx = $_
   Get-InboxRule -Mailbox $mbx.UserPrincipalName | Where-Object { $_.ForwardTo -ne $null -or $_.RedirectTo -ne $null } |
   Select-Object @{Name="ForwardingType";Expression={"Inbox Rule"}}, @{Name="User";Expression={$mbx.UserPrincipalName}}, Name, ForwardTo, RedirectTo
}
 
# Add Inbox Rule results to the array
$results += $inboxRules
 
# 3. Mail Flow Rules Auto Forward or Redirect
$mailFlowRules = Get-TransportRule | Where-Object { $_.Actions -match 'RedirectMessageTo' -or $_.Actions -match 'ForwardTo' } |
Select-Object @{Name="ForwardingType";Expression={"Mail Flow Rule"}}, Name, Enabled, Priority, Actions
 
# Add Mail Flow Rule results to the array
$results += $mailFlowRules
 
# Export results to CSV, with headers
$results | Export-Csv -Path $outputFile -NoTypeInformation
 
# Output completion message
Write-Output "Forwarding report exported to $outputFile"
```

# ENABLE SINGLE ITEM RECOVERY FOR ALL TENANT MAILBOXES

```powershell
# ENABLE SINGLE ITEM RECOVERY FOR ALL TENANT MAILBOXES
 
Get-Mailbox -ResultSize Unlimited | ForEach-Object {
   Set-Mailbox $_.Identity -SingleItemRecoveryEnabled $true
}
 
# Output completion message
Write-Output "Single Item Recovery has been enabled for all mailboxes."
```

#RETENTION POLICIES SCRIPT

```powershell
#RETENTION POLICIES SCRIPT:
#Script to list and export EXO tenant retention policies
 
# Connect to Exchange Online
$UserCredential = Get-Credential
Connect-ExchangeOnline -UserPrincipalName $UserCredential.UserName -ShowProgress $true
 
# Get all retention policies
$RetentionPolicies = Get-RetentionPolicy
 
# Export retention policies to CSV
$ExportPath = "C:\RetentionPolicies.csv"
$RetentionPolicies | Select-Object Name,RetentionPolicyTagLinks,RetentionEnabled | Export-Csv -Path $ExportPath -NoTypeInformation
 
# Disconnect from Exchange Online
Disconnect-ExchangeOnline -Confirm:$false
 
Write-Output "Retention policies have been exported to $ExportPath"
```

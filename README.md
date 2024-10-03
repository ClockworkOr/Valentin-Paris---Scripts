# VALENTIN PARIS - SCRIPTS

This page contains some useful scripts from Valentin Paris, Customer Engineer at Microsoft.

The document contains several PowerShell scripts designed to manage and report on various aspects of an Exchange Online environment. Here are the key sections:

- Forwarding Settings Report: This script gathers all types of forwarding currently in place, including remote domain auto forward settings, inbox rules auto forward, and mail flow rules auto forward or redirect. The results are exported to a CSV file named ForwardingReport.csv1.

- Single Item Recovery: This script enables single item recovery for all tenant mailboxes. It iterates through all mailboxes and sets the SingleItemRecoveryEnabled property to $true1.

- Retention Policies Script: This script lists and exports Exchange Online tenant retention policies. It connects to Exchange Online, retrieves all retention policies, and exports them to a CSV file named RetentionPolicies.csv1.

```powershell

# TIP: You can copy - paste the scripts using the top-right icon on each code section.

$FirstName = "Valentin"
$LastName = "Paris"
$Role = "Customer Engineer"

Write-Host "Hello, I am $FirstName $LastName, and I am a $Role. `n I will be the engineer who will take care of you today."

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

# RETENTION POLICIES SCRIPT

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

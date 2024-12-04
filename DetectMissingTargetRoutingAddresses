# Work In Progress. Adding raw code for push to repo by the repo owner.

DETECT & RESOLVE MISSING TARGET ADDRESS FOR EXCHANGE HYBRID MIGRATIONS

# Instructions to Use the Script (May want to add as ReadMe)

Replace OU=Users,DC=YourDomain,DC=com with the Distinguished Name (DN) of the OU containing your users.
Replace mail.tenantName.onmicrosoft.com with the correct target routing address for your tenant.
Save the script as a .ps1 file (e.g., CheckUsersRoutingAddress.ps1).
Run the script on a server or workstation with the Active Directory module installed and the necessary permissions.
# Output will generate two CSVs:
UsersWithAddress.csv: Lists users with the target routing address.
UsersMissingAddress.csv: Lists users without the target routing address.
*Optional Automation
To automatically add the missing address, uncomment the Set-ADUser line within the foreach loop. Ensure you test in a non-production environment first to confirm it works as expected!
PowerShell Full Script Canvas - Can be ran as a PS1. Make sure to replace "sample" values with the appropriate ones for your organization.


# Define the Organizational Unit (OU) to scope the users
$OU = "OU=Users,DC=YourDomain,DC=com" # Replace with the actual OU path
$TargetDomain = "mail.tenantName.onmicrosoft.com" # Replace with your tenant domain

# Output CSV files
$WithAddressCSV = "UsersWithAddress.csv"
$MissingAddressCSV = "UsersMissingAddress.csv"

# Import the Active Directory module
Import-Module ActiveDirectory

# Fetch all users in the specified OU
$Users = Get-ADUser -Filter * -SearchBase $OU -Properties ProxyAddresses | Where-Object { $_.Enabled -eq $true }

# Arrays to hold user details
$UsersWithAddress = @()
$UsersMissingAddress = @()

foreach ($User in $Users) {
    # Check if the target routing address exists in the ProxyAddresses attribute
    if ($User.ProxyAddresses -contains "SMTP:$TargetDomain") {
        $UsersWithAddress += [PSCustomObject]@{
            SamAccountName = $User.SamAccountName
            DisplayName = $User.Name
            ProxyAddresses = ($User.ProxyAddresses -join "; ")
        }
    } else {
        $UsersMissingAddress += [PSCustomObject]@{
            SamAccountName = $User.SamAccountName
            DisplayName = $User.Name
            ProxyAddresses = ($User.ProxyAddresses -join "; ")
        }

        # Optionally add the target address to the user (uncomment to enable)
        # Set-ADUser -Identity $User.SamAccountName -Add @{ProxyAddresses="SMTP:$TargetDomain"}
    }
}

# Export the results to CSV files
$UsersWithAddress | Export-Csv -Path $WithAddressCSV -NoTypeInformation -Encoding UTF8
$UsersMissingAddress | Export-Csv -Path $MissingAddressCSV -NoTypeInformation -Encoding UTF8

Write-Host "Report generated! Users with the target address are saved in $WithAddressCSV, and users missing it are in $MissingAddressCSV."

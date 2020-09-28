 Function PasswordChange ($user) {
    
    $first = $user.GivenName.Substring(0,1)
    $last = $user.Surname.Substring(0,1)

    $newpassword = "#Menlo4$first$last!"
    $securepassword = ConvertTo-SecureString $newpassword -AsPlainText -force

    try {
        Set-ADAccountPassword -Identity $user -Reset -NewPassword $securepassword
        Write-Host "Password Change Successful"
    }

    catch {
        Write-Host "Could not change password"
    }

}

Function LicenseCheck ($user, $license, $switch) {

    switch ($switch) {
    
        "remove" {
        
            try {
                Set-MsolUserLicense -UserPrincipalName $user.EmailAddress -RemoveLicenses $license -ErrorAction STOP # Remove Business Standard Licenses
                Write-Host ("Removed $license license")
            }

            catch {
    
                $MsolUserError = $_.Exception.Message

                if ($MsolUserError -match "Unable to assign this license because it is invalid") {
                    Write-Host "Removing $license : an error has occurred (most likely license has already been removed)"
                }
    
                else {
                    Write-Host "Other error has occured"
                    $error
                    throw
                }
            }
        }
        
        "add" {

            try {
                Set-MsolUserLicense -UserPrincipalName $user.EmailAddress -AddLicenses "mppc:STANDARDWOFFPACK" -ErrorAction STOP # Add E2 Licenses
                Write-Host "Added $license license"
            }

            catch {
    
                $MsolUserError = $_.Exception.Message

                if ($MsolUserError -match "Unable to assign this license because it is invalid") {
                    Write-Host "Adding $license : an error has occurred (most likely license has already been added)"
                }
    
                else {
                    Write-Host "Other error has occured"
                    $error
                    throw
                }
            }
        }

        Default {
            Write-Host "Invalid license command (you can either 'remove' or 'add' licenses)"
        }

    }
}

# Connect to O365 Tenant
Connect-MsolService 

$username = Read-Host "Please enter the user's AD username"
$user = Get-ADUser -identity $username -Properties UserPrincipalName, GivenName, Surname, EmailAddress

$forwardname = Read-Host "Forwarding username? "
$forward = Get-ADUser -identity $forwardname -Properties UserPrincipalName, GivenName, Surname, EmailAddress

# Change Password
PasswordChange $user

# Remove Licenses 
LicenseCheck $user "mppc:O365_BUSINESS_PREMIUM" "remove" # remove business premium
LicenseCheck $user "mppc:THREAT_INTELLIGENCE" "remove" # remove ATP
LicenseCheck $user "mppc:STANDRDWOFFPACK" "add" # add E2 License


# Change Account Description
Set-ADUser $username -Description "Disabled on $(Get-Date -format "d")"

# Set Forwarding
try {
    Set-Mailbox -Identity $user.EmailAddress -ForwardingAddress $forward.EmailAddress
    Write-Host "Forwarding ", $user.EmailAddress, "to",  $forward.EmailAddress
}
catch {
    Write-Host "Forwarding did not work!"
}

# Set Auto Response
$message = "Sorry to have missed you. I am no longer on staff with Menlo Church. Your email has been forwarded to", [string]$forward.GivenName, [string]$forward.SurName, "for response. Thank you."
Set-MailboxAutoReplyConfiguration -Identity $username -AutoReplyState Enabled -InternalMessage $message -ExternalMessage $message

# Remove AD Groups
Get-AdPrincipalGroupMembership -Identity $username | Where-Object -Property Name -Ne -Value 'Domain Users' | Remove-AdGroupMember -Members $username # Strip all AD Groups

# Remove OWA Groups


# Sync AD
Start-ADSyncSyncCycle -PolicyType Delta

 

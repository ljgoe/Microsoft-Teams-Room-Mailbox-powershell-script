## Use the powershell script to create an MTR mailbox from Powershell

### 1. Copy the powershell script [CreatingTeamsRoomAccount.ps1](https://github.com/ljgoe/MS-Teams-room-creation/blob/main/CreatingTeamsRoomAccount.ps1)

#### Install these modules if you have never connected to Office 365 / MS Online / Exchange Online
> You can Skip any modules you already have installed, on a new built PC you will run these  comands
> Note that if you are having issues then try using the x64 bit version of powershell running as admin
```
#### Modules #####
- # Skip publisher check 
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12bInstall-Module PowerShellGet -RequiredVersion 2.2.4 -SkipPublisherCheck

- # Install Nuget
Install-PackageProvider -Name nuget -MinimumVersion 2.8.5.201 -force

# Install PnP.PowerShell with version 1.12.0 
Install-Module -Name "PnP.PowerShell" -RequiredVersion 1.12.0 -Force -AllowClobber

#If you are connecting to Azure AD / Azure Resource Manager from your Windows PC for the first time, you must first run the following PowerShell commands (as Administrator):
Install-Module -Name AzureAD
Install-Module -Name Az -MinimumVersion 3.0.0 -AllowClobber -Scope AllUsers

# Install   
Set-ExecutionPolicy RemoteSigned
Install-Module PowershellGet -Force
Update-Module PowershellGet
Install-Module -Name MSOnline –Force
import-Module MSOnline
Install-Module -Name ExchangeOnlineManagement
Import-Module ExchangeOnlineManagement
install-module AzureADPreview
```

#### Connect to M365 and Exchange Online with your Tenant Admin Account
```
# if you get an error like this "you must use multi-factor authentication to access XYZ"
# the just issue the base command e.g "Connect-ExchangeOnline" and authenticate 
$UserCredential = Get-Credential
Connect-MsolService -Credential $UserCredential
Connect-ExchangeOnline -Credential $UserCredential -ShowProgress $true


#Get the Meeting Room License SKU and nake sure to set it in the variables in the next step $license="xyz"
Get-MsolAccountSku

#Set the variables for Meeting Room account
$newRoom="MTR-DemoTeamsRoom@testit.vc"
$name="MTR-Demo"
$pwd="yourpassword"
$license="testitvideo:Microsoft_Teams_Rooms_Pro"
$location="AU"

# Create a mailbbox resource

<# 
Set the calendar processing with some key parameters and details
1 - Setting AutomateProcessing to AutoAccept means that meetings will be processed and accepted automatically if there are no conflicts
2 - Setting AddOrganizerToSubject to false ensures that the original subject is preserved and not replaced by the organizers’ name
3 - Setting ProcessExternalMeetingMessages to true
3 - Setting the RemovePrivateProperty to false ensures that the private flag for meeting requests is preserved (private meetings stay private)
4 - Setting DeleteComments and DeleteSubject to false is critical and ensures that your meeting invitation has a “Join” button
5 - The AdditionalResponse parameters are there to send useful information in the message back to the requester
#> 

New-Mailbox -MicrosoftOnlineServicesID $newRoom -Name $name -Room -RoomMailboxPassword (ConvertTo-SecureString -String $pwd -AsPlainText -Force) -EnableRoomMailboxAccount $true
Start-Sleep -Seconds 31
Set-MsolUser -UserPrincipalName $newRoom -PasswordNeverExpires $true -UsageLocation $location
Set-MsolUserLicense -UserPrincipalName $newRoom -AddLicenses $license
Set-Mailbox -Identity $newRoom -MailTip “This room is equipped to support MS Teams Meetings”
Set-CalendarProcessing -Identity $newRoom -AutomateProcessing AutoAccept -AddOrganizerToSubject $false -ProcessExternalMeetingMessages $True -RemovePrivateProperty $false -DeleteComments $false -DeleteSubject $false -AddAdditionalResponse $true -AdditionalResponse “Your meeting is now scheduled and if it was enabled as a Teams Meeting will provide a seamless click-to-join experience from the conference room.”


#Connect to MS Online to set password to never expires

Connect-MsolService -Credential $UserCredential
Set-MsolUser -UserPrincipalName $newRoom -PasswordNeverExpires $true
Get-MsolUser -UserPrincipalName $newRoom | Select PasswordNeverExpires

<# Optional
Use the Set-Place cmdlet to update room mailboxes with additional metadata, which provides a better search and room suggestion experience”
#>

Set-Place -Identity $newRoom -IsWheelChairAccessible $true -AudioDeviceName “Audiotechnica Wireless Mics” -VideoDeviceName “POLY STUDIO X70”

<# 
Meeting Room Voice Configuration
If you want the meeting room to be able to make calls to the PSTN you need to enable Enterprise Voice and configure a way for the user to place calls. 
If you’re using Calling Plans from Microsoft, you need to assign the user a calling plan license. 
If, on the other hand, you’re using Direct Routing through your own SBC or that of a Service Provider, you can grant the user account a Voice Routing Policy.
#>

Set-CsUser -Identity $newRoom -EnterpriseVoiceEnabled $true
Grant-CsOnlineVoiceRoutingPolicy -Identity $newRoom -PolicyName “Policy Name”


<#CHECK YOUR Settings that have been applied#>

Get-mailbox -Identity $newRoom | Fl
Get-Place -Identity $newRoom | Fl
```











## Troubleshooting

Remember to Run this from powershell X64 as administrator 

![powershell_X64](https://github.com/ljgoe/MS-Teams-room-creation/blob/main/Images/Images/Powershell_X64.png)

If you dont you may face an error like this:  
```
import-Module : Could not load file or assembly 'file:///C:\Program Files\WindowsPowerShell\Modules\MSOnline\1.1.183.66\Microsoft.Online.Administration.Automation.PSModule.dll' or one of 
its dependencies. An attempt was made to load a program with an incorrect format.
```
![error](https://github.com/ljgoe/MS-Teams-room-creation/blob/main/Images/Images/Powershell_error.png)

## Create a Virtual Microsoft Teams Room for Lab purposes

Full instructions are on my website: [techblogwiki](https://techblogwiki.azurewebsites.net/index.php/knowledge-base/microsoft-teams-room-creation/)

Prerequisites:
- Windows VM running winodws 11
- Office 365 tennant with administration rights
- Microsoft teams Pro or Basic licence
- Access to the windows 11 Image

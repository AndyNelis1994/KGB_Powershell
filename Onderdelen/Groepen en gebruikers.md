#Groepen en gebruikers Nicolai

**Users in AD aanmaken**

    New-ADUser -Name JSmith -....

Je kan ook gebruiken maken van een functie om eenvoudig users aan te maken waarbij users uit csv file worden gemaakt

    Function Create-Users{
    param($fileName, $emailDomain, $userPass, $numAccounts=10)
    if($fileName -eq $null ){
    [array]$users = $null
    for($i=0; $i -lt $numAccounts; $i++){
    $users += [PSCustomObject]@{
    FirstName = 'Random'
    LastName = 'User' + $i
    }
    }
    } else {
    $users = Import-Csv -Path $fileName
    }

Eigen script om gebruikers te importeren (in juiste OU en aanmaken van OU) adhv csv file:

	Import-Module ActiveDirectory
    
    $ADUsers = Import-Csv werknemers.csv
    
    if ([ADSI]::Exists("LDAP://OU=Technical,DC=Projecten2,DC=be") -eq $false) {
    	New-ADOrganizationalUnit -Name Technical -ProtectedFromAccidentalDeletion $false
    }
    
    if ([ADSI]::Exists("LDAP://OU=HR,DC=Projecten2,DC=be") -eq $false) {
    	New-ADOrganizationalUnit -Name HR -ProtectedFromAccidentalDeletion $false
    }
    
    if ([ADSI]::Exists("LDAP://OU=Sales,DC=Projecten2,DC=be") -eq $false) {
    	New-ADOrganizationalUnit -Name Sales -ProtectedFromAccidentalDeletion $false
    }
    
    
    foreach ($User in $ADUsers)
    {
    	$Name = $User.GivenName + " " + $User.Surname
    	$OU = $User.ParentOU
    	$Username = $User.GivenName.Substring(0,2) + $User.Surname.Substring(0,3) + $User.Number
    	$City = $User.City
    	$Title = $User.Title
    	$GivenName = $User.GivenName
    	$Surname = $User.Surname
    	$StreetAddress = $User.StreetAddress
    	$Postcode = $User.Postcode
    	$Country = $User.Country
    	$Telefoon = $User.Telefoon
    	$Gender = $User.Gender
    	$Birthday = $User.Birthday
    	$CountryFull = $User.CountryFull
    	
    if (Get-ADUser -F {SamAccountName -eq $Username})
    {
    	Write-Warning "A user with $Username already exists in your Active Directory."
    
    }
    else
    {
    	New-ADUser -SamAccountName $Username -Name $Name -Path $OU -City $City -Title $Title -GivenName $GivenName -Surname $Surname -StreetAddress $StreetAddress -PostalCode $Postcode -Country $Country -MobilePhone $Telefoon -AccountPassword (ConvertTo-SecureString "Projecten2" -AsPlainText -Force) -Description "Gender: $Gender, birthday: $Birthday, country: $CountryFull" -ScriptPath logon.vbs -PassThru | Enable-ADAccount
    }
    
    	Get-ADUser -Filter * -SearchBase "OU=HR,DC=Projecten2,DC=be" | Set-ADUser -CannotChangePassword:$false -PasswordNeverExpires:$false -ChangePasswordAtLogon:$True
    	Get-ADUser -Filter * -SearchBase "OU=Sales,DC=Projecten2,DC=be" | Set-ADUser -CannotChangePassword:$false -PasswordNeverExpires:$false -ChangePasswordAtLogon:$True
    	Get-ADUser -Filter * -SearchBase "OU=Technical,DC=Projecten2,DC=be" | Set-ADUser -CannotChangePassword:$false -PasswordNeverExpires:$false -ChangePasswordAtLogon:$True
    }

**Zoeken naar en rapporteren op AD users**

Voer eerst de volgende code uit:

    Get-ADUser -Filter * -Properties SamAccountName, DisplayName, `
    ProfilePath, ScriptPath | `
    Select-Object SamAccountName, DisplayName, ProfilePath, ScriptPath

Om de gedisabled gebruikers te vinden:

    Get-ADUser –Filter 'Enabled -eq $false'

Om gebruikers te vinden die de laatste 30 dagen niet meer hebben ingelogged:

    $logonDate = (Get-Date).AddDays(-30)
    Get-ADUser -Filter 'LastLogonDate -lt $logonDate' | Select-Object
    DistinguishedName

Om accounts te vinden die al meerdere malen verkeerde inlog gegevens hebben ingegeven:

    $primaryDC = Get-ADDomainController -Discover -Service PrimaryDC
    Get-ADUser -Filter 'badpwdcount -ge 5' -Server $primaryDC.Name `
    -Properties BadPwdCount | Select-Object DistinguishedName,
    BadPwdCount


#Groepen en gebruikers Andy
###Gebruikers

***Eerste gebruiker aanmaken***<br>
Een gebruiker aanmaken door alle gegevens te specifieren.

``` 
$SecurePW = Read-Host -Prompt "Enter a password" -asSecureString
New-ADUser -Name "Charlie Russel" `
           -AccountPassword $SecurePW  `
           -SamAccountName 'Charlie' `
           -DisplayName 'Charlie Russel' `
           -EmailAddress 'Charlie@TreyResearch.net' `
           -Enabled $True `
           -GivenName 'Charlie' `
           -PassThru `
           -PasswordNeverExpires $True `
           -Surname 'Russel' `
           -UserPrincipalName 'Charlie'
``` 

***Administrator account aanmaken***<br>
Een administrator account aanmaken. Ook wel het '500' account genoemd omdat zijn SID eindigd op 500. Dit account moet zeer goed beveiligd zijn.

``` 
Get-ADUser -Identity Administrator
``` 
***Gebruikers aanmaken doormiddel van een CSV bestand***<br>
Het is natuurlijk veel gemakkelijker om gebruikers via een CSV bestand toe te voegen dan elk individueel. Om dit te doen gebruiken we het script `Create-TreyUsers.ps1` dat ook te vinden is in het mapje scripts.

``` 
<#
.Synopsis
Creates the TreyResearch.net users
.Description
Create-TreyUsers reads a CSV file to create an array of users. The users are then added
to the users container in Active Directory. Additionally, Create-TreyUsers adds the
user Charlie to the same AD DS Groups as the Administrator account.
.Example
Create-TreyUsers
Creates AD Accounts for the users in the default "TreyUsers.csv" source file
.Example
Create-TreyUsers -Path "C:\temp\NewUsers.txt"
Creates AD accounts for the users listed in the file C:\temp\NewUsers.txt"
.Parameter Path
The path to the input CSV file. The default value is ".\TreyUsers.csv".
.Inputs
[string]
.Notes
    Author: Charlie Russel
 Copyright: 2015 by Charlie Russel
          : Permission to use is granted but attribution is appreciated
   Initial: 3/26/2015 (cpr)
   ModHist:
          :
#>
[CmdletBinding()]
Param(
     [Parameter(Mandatory=$False,Position=0)]
     [string]
     $Path = ".\TreyUsers.csv"
     )

$TreyUsers = @()
If (Test-Path $Path ) {
   $TreyUsers = Import-CSV $Path
} else {
   Throw  "This script requires a CSV file with user names and properties."
}

ForEach ($user in $TreyUsers ) {
   New-AdUser -DisplayName $User.DisplayName `
              -GivenName $user.GivenName `
              -Name $User.Name `
              -SurName $User.SurName `
              -SAMAccountName $User.SAMAccountName `
              -Enabled $True `
              -PasswordNeverExpires $true `
              -UserPrincipalName $user.SAMAccountName `
              -AccountPassword (ConvertTo-SecureString -AsPlainText -Force -String
"P@ssw0rd!" )
   If ($User.SAMAccountName -eq "Charlie" ) {
      $cprpwd = Read-Host -Prompt 'Enter Password for account: Charlie' -AsSecureString
      Set-ADAccountPassword -Identity Charlie -NewPassword $cprpwd -Reset
      $SuperUserGroups = @()
      $SuperUserGroups = (Get-ADUser -Identity "Administrator" -Properties * ).MemberOf

      ForEach ($Group in $SuperUserGroups ) {
         Add-ADGroupMember -Identity $Group -Members "Charlie"
      }
      Write-Host "The user $user.SAMAccountName has been added to the following AD
Groups: "
      (Get-ADUser -Identity $user.SAMAccountName -Properties * ).MemberOf
   }
}
``` 

Het CSV bestand moet volgende opmaak hebben als je bovenstaand script wilt gebruiken.

``` 
Name,GivenName,Surname,DisplayName,SAMAccountName,Description
David Guy,David,Guy,Dave R. Guy,Dave,Customer Appreciation Manager
Alfredo Fettucine,Alfredo,Fettuccine,Alfie NoNose,Alfie,Shop Foreman
Stanley Behr,Stanley,Behr,Stanley T. Behr, Stanley,WebMaster
``` 

***Aangemaakte gebruikers bekijken***<br>
Met volgend commando krijg je een lijst van de aangemaakte gebruikers.

``` 
(Get-ADUser -Filter {Enabled -eq "True"} -Properties DisplayName).DisplayName
``` 

###Groepen
***Nieuwe groep aanmaken***<br>

``` 
New-ADGroup –Name 'Accounting Users' `
            -Description 'Security Group for all accounting users' `
            -DisplayName 'Accounting Users' `
            -GroupCategory Security `
            -GroupScope Universal `
            -SAMAccountName 'AccountingUsers' `
            -PassThru
``` 

***Gebruiker aan groep toevoegen***<br>
R. Guy en Stanley T. toevoegen aan groep AccountingUsers.

``` 
Add-ADGroupMember -Identity AccountingUsers -Members Dave,Stanley -PassThru
``` 

Bekijke de leden van de groep met:

``` 
Get-ADGroupMember -Identity AccountingUsers
``` 

***Array van gebruikers toevoegen aan groep***<br>
Array van managers maken.

``` 
$ManagerArray = (Get-ADUser -Filter {Description -like "*Manager*" } `
                            -Properties Description).SAMAccountName
``` 

Dan de array toevoegen aan de groep managers.

``` 
Add-ADGroupMember -Identity "Managers" -Members $ManagerArray -PassThru
``` 

Bekijk of de gebruikers aan de groep toegevoegd zijn.

``` 
Get-ADGroupMember -Identity Managers | ft -auto SAMAccountName,Name
``` 

***Gebruiker aan meerdere groepen toevoegen***<br>
Array van groepen maken.

```
$Groups = (Get-ADUser -Identity Charlie -Properties *).MemberOf
```

Gebruiker toevoegen aan de array van groepen.

```
Add-ADPrincipalGroupMembership -Identity Alfie -MemberOf $Groups
```

Bekijk of de gebruiker aan de groepen toegevoegd zijn.

``` 
(Get-ADUser -Identity Alfie -Properties MemberOf).MemberOf
``` 

***Gebruiker verwijderen uit een groep***<br>

```
Remove-ADPrincipalGroupMembership -Identity Alfie `
                                  -MemberOf "Enterprise Admins",`
                                            "Schema Admins",`
                                            "Group Policy Creator Owners" `
                                  -PassThru
```

Bekijk of de gebruiker verwijderd is uit de groepen.

``` 
(Get-ADUser -Identity Alfie -Properties MemberOf).MemberOf
``` 

###OU's
***OU aanmaken***<br>

``` 
New-ADOrganizationalUnit -Name Engineering `
                         -Description 'Engineering department users and computers' `
                         -DisplayName 'Engineering Department' `
                         -ProtectedFromAccidentalDeletion $True `
                         -Path "DC=TreyResearch,DC=NET" `
                         -PassThru
```   
***Gebruikers en computers aan de OU toevoegen.***<br>
Om gebruikers of computers te verplaatsen naar een OU hebben we hun DN of GUID nodig. Deze paramter vullen we in bij `Identity`. Bij `TargetPath` vullen we de doel OU en de DC in.

Om de GUID van bijvoorbeeld Harlod te vinden gebruiken we.

``` 
Get-ADUser -Identity Harold
``` 

Om de GUID van de computer van Harold te vinden gebruiken we.

``` 
Get-ADComputer -Filter {Description -like "*Harold*" }
``` 
Nu kunnen we de computer van Harold toevoegen aan de OU Engineering.

``` 
Move-ADObject -Identity "46df71bd-ba88-4b26-9091-b8db6e07261a" `
              -TargetPath " OU=Engineering,DC=TreyResearch,DC=net" `
              -PassThru
```                        
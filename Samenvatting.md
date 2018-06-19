
# Windows Server 2012 Automation with PowerShell Cookbook #


## Notities door Andy Nelis ##


**Algemeen**

Voor het werken met Powershell help eerst updaten:

    Update-Help -Module * -Force

Ipconfig naar textfile:

    ipconfig /all > ipconfig.txt
	notepad ipconfig.txt

Toestemming vragen voor het uitvoeren van een commando adhv parameter

	Stop-Process -Name *bits* -Confirm


Voorbeelden zien van een commando in help:

    Get-Help Get-Process -Examples

Aanmaken van een alias voor iets waar nog geen alias voor is:

    Get-Help New-Alias -full
	New-Alias gh Get-Help

Volledige hulp pagina krijgen:

    Get-Help Get-Process -full

## Understanding PowerShell Scripting

**Security**: welke scripts mogen worden uitgevoerd?

    Set-ExecutionPolicy Restricted | AllSigned | RemoteSigned | Bypass | Undefined

Restrictued: geen scripts mogen worden uitgevoerd<br>
AllSigned: enkel uitvoeren door vertrouwde<br>
Unrestricted: toegang om alle scripten uit te voeren<br>

**Functies maken**: aantal dagen tot een bepaald moment (hier Kerstmis). Tussen de <# #> kan informatie meegegeven worden over de functie.

    Function Get-DaysTilChristmas
	{
	<#
	.Synopsis
	This function calculates the number of days until Christmas
	.Description
	This function calculates the number of days until Christmas
	.Example
	DaysTilChristmas
	.Notes
	Ed is really awesome
	.Link
	Http://blog.edgoad.com
	#>
	$Christmas=Get-Date("25 Dec " + (Get-Date).Year.ToString() + "
	7:00 AM")
	$Today = (Get-Date)
	$TimeTilChristmas = $Christmas - $Today
	Write-Host $TimeTilChristmas.Days "Days 'til Christmas"
	}

**Zelf modules maken** die we eventueel kunnen importeren in Powershell

1. Functies maken die we samen kunnen groeperen

        Function Get-Hello
   		 {
    	Write-Host "Hello World!"
    	}
    	Function Get-Hello2
    	{
    	Param($name)
    	Write-Host "Hello $name"
    	}
    	www.it-ebooks.info
    	Understanding PowerShell Scripting
    	16

2. Slaag op in een bestand
    
    	`name Hello.PSM1`

3. Als de folder nog niet bestaat, maak een nieuwe aan
   
    	$modulePath = "$env:USERPROFILE\Documents\WindowsPowerShell\
    	Modules\Hello"
    	if(!(Test-Path $modulePath))
   		 {
   		 New-Item -Path $modulePath -ItemType Directory
   		 }
    	Copy Hello.PSM1 to the new module folder.
    	$modulePath = "$env:USERPROFILE\Documents\WindowsPowerShell\
    	Modules\Hello"
    	Copy-Item -Path Hello.PSM1 -Destination $modulePath

4. execute Get-Module –ListAvailable om ze op te noemen


**Variabelen in functies** [] geeft aan welk type $ geeft de naam van de var aan.

    Function Add-Numbers
    {
    Param(
    [int]$FirstNum = $(Throw "You must supply at least 1 number")
    , [int]$SecondNum = $FirstNum
    )
    Write-Host ($FirstNum + $SecondNum)
    }

Oproepen door:

	Add-Numbers 7 5
	Add-Numbers -FirstNum 7 -SecondNum 5

**Resultaat van functie opslaan in variabele**

    $foo = Add-Numbers 7 5

**Functie voor testen of input geldige telefoonnummer is**: moet zijn "3decimalen-4decimalen" 

    Function Test-PhoneNumber
    {
    param([ValidatePattern("\d{3}-\d{4}")] $phoneNumber)
    Write-Host "$phoneNumber is a valid number"
    }

**Functie parameter van pipeline laten aanvaarden**: parameter ValueFromPipeline

    Function Square-Num
    {
    Param([float]
    [Parameter(ValueFromPipeline = $true)]
    $FirstNum )
    Write-Host ($FirstNum * $FirstNum)
    }


   Uitvoeren als:

	5 | Square-Num

**Opnemen van alle commando's tijdens powershell sessie** in apart document om later terug te kunnen bekijken

    Start-Transcript -Path C:\Gebruikers/ANLS/Documenten
	Stop-Transcript

**Email laten sturen als er bijvoorbeeld een bepaalde taak achter de rug is**

   Maak volgende functie aan:
    
    function Send-SMTPmail($to, $from, $subject, $smtpServer, $body)
    {
    $mailer = new-object Net.Mail.SMTPclient($smtpServer)
    $msg = new-object Net.Mail.MailMessage($from, $to, $subject,
    $body)
    $msg.IsBodyHTML = $true
    $mailer.send($msg)
    }

Oproepen door:

    Send-SMTPmail -to "admin@contoso.com" -from "mailer@contoso.com" `
    -subject "test email" -smtpserver "mail.contoso.com" -body
    "testing"

**Sorteren** <br>
Filtering:

    Get-Process | Where-Object {$_.Name -eq "chrome"} --> $. komt uit pipe
    Get-Process | Where-Object Name -eq "chrome" --> equals
    Get-Process | Where-Object Name -like "*hrom*" --> komt voor
    Get-Process | Where-Object Handles -gt 1000 --> greater then
    Get-Process | Sort-Object Handles -Descending --> dalend sorteren op handles

Sorting:

    Get-Process | Sort-Object Handles
    Get-Process | Sort-Object Handles -Descending
    Get-Process | Sort-Object Handles, ID –Descending

Selecting:

    Select-String -Path C:\Windows\WindowsUpdate.log -Pattern
    "Installing updates"
    Get-Process | Select-Object Name -Unique

Grouping:

    Get-Process | Format-Table -GroupBy ProcessName

Formatting numbers:

    $jenny = 1206867.5309
    Write-Host "Original:`t`t`t" $jenny
    Write-Host "Whole Number:`t`t" ("{0:N0}" -f $jenny)
    Write-Host "3 decimal places:`t" ("{0:N3}" -f $jenny)
    Write-Host "Currency:`t`t`t" ("{0:C2}" -f $jenny)
    Write-Host "Percentage:`t`t`t" ("{0:P2}" -f $jenny)
    Write-Host "Scientific:`t`t`t" ("{0:E2}" -f $jenny)
    Write-Host "Fixed Point:`t`t" ("{0:F5}" -f $jenny)
    Write-Host "Decimal:`t`t`t" ("{0:D8}" -f [int]$jenny)
    Write-Host "HEX:`t`t`t`t" ("{0:X0}" -f [int]$jenny)

**Werken met errors**: try catching: bij 2 nummers --> correct, bij 2 strings --> error boodschop wordt weergegeven in de catch
    
    Function Multiply-Numbers
    {
    Param($FirstNum, $SecNum)
    Try
    {
    Write-Host ($FirstNum * $SecNum)
    }
    Catch
    {
    Write-Host "Error in function, present two numbers to
    multiply"
    }
    }

**Performantie**: meet hoe lang het duurt om iets uit te voeren

    Measure-Command { opvolging van verschillende commando's }

## Managing Windows Network Services with PowerShell

**Configureren van statisch netwerking**


1: Interfaces vinden

	Get-NetIPInterface

2: IP info instellen

	New-NetIPAddress -AddressFamily IPv4 -IPAddress 10.10.10.10 -PrefixLength 24 -InterfaceAlias Ethernet

3: DNS instellen

	Set-DnsClientServerAddress -InterfaceAlias Ethernet -ServerAddresses "10.10.10.10","10.10.10.11"

4: Default route instellen: nieuw netwerk route --> default gateway instellen 10.10.10.1 is router address

	New-NetRoute -DestinationPrefix "0.0.0.0/0" -NextHop "10.10.10.1" -InterfaceAlias Ethernet

   Resultaat: IPv4 instellingen:<br>
   Use the following IP address:<br>
   - IP Address: 10.10.10.11<br>
   - Subnet mask: 255.255.255.0<br>
   - Default gateway: 10.10.10.1<br>
   Use the following DNS server addresses:<br>
   - Preferred DNS: 10.10.10.10<br>
   - Alternate DNS: 10.10.10.11<br>

**Installeren van domain controllers** eens ip configuratie in orde is

1. Open powershell als admin

2. Identify Windows Features om te installen

    	Get-WindowsFeature | Where-Object Name -like *domain*
    	Get-WindowsFeature | Where-Object Name -like *dns*

3. installeer de nodige features

    	Install-WindowsFeature AD-Domain-Services, DNS –
    	IncludeManagementTools

4. configureer het domein

    	$SMPass = ConvertTo-SecureString 'P@$$w0rd11' –AsPlainText -Force
    	Install-ADDSForest -DomainName corp.contoso.com –
    	SafeModeAdministratorPassword $SMPass –Confirm:$false

5. Indien we nog een pc hebben die we lid willen maken van het domein: CORPDC2 wordt lid van domain

    	$secString = ConvertTo-SecureString 'P@$$w0rd11' -AsPlainText
    	-Force
    	$myCred = New-Object -TypeName PSCredential -ArgumentList "corp\
    	administrator", $secString
    	Add-Computer -DomainName "corp.contoso.com" -Credential $myCred –
    	NewName "CORPDC2" –Restart

6. Willen we deze CORPDC2 instellen als domain controller:

    	Install-WindowsFeature –Name AD-Domain-Services, DNS
    	-IncludeManagementTools –ComputerName CORPDC2
    	Invoke-Command –ComputerName CORPDC2 –ScriptBlock {
    	$secPass = ConvertTo-SecureString 'P@$$w0rd11' -AsPlainText –Force
    	$myCred = New-Object -TypeName PSCredential -ArgumentList "corp\
    	administrator", $secPass
    	$SMPass = ConvertTo-SecureString 'P@$$w0rd11' –AsPlainText –Force
    	Install-ADDSDomainController -DomainName corp.contoso.com –
    	SafeModeAdministratorPassword $SMPass -Credential $myCred –
    	Confirm:$false
    	}

**Configureren van zones in DNS**

1. Ga na welke features moeten worden geïnstalleerd:

		Get-WindowsFeature | Where-Object Name -like *dns*

2. Installeer DNS (als dit nog niet gedaan is):

		Install-WindowsFeature

3. Maak een reverse lookup zone:

		Add-DnsServerPrimaryZone –Name 10.10.10.in-addr.arpa –
		ReplicationScope Forest
		Add-DnsServerPrimaryZone –Name 20.168.192.in-addr.arpa –
		ReplicationScope Forest

4. Maak een primary zone en voeg static records toe:

    	Add-DnsServerPrimaryZone –Name contoso.com –ZoneFile contoso.com.
		dns
		Add-DnsServerResourceRecordA –ZoneName contoso.com –Name www –
		IPv4Address 192.168.20.54 –CreatePtr

5. Maak een conditional forwarder:

		Add-DnsServerConditionalForwarderZone -Name fabrikam.com
		-MasterServers 192.168.99.1

6. Maak een secondary zone aan:

		Add-DnsServerSecondaryZone -Name corp.adatum.com -ZoneFile corp.
		adatum.com.dns -MasterServers 192.168.1.1

Noem al de zones op

    Get-DnsServerZone

**Configureren van DHCP scopes**

1. Installeer DHCP en de management tools:

		Get-WindowsFeature | Where-Object Name -like *dhcp*
		Install-WindowsFeature DHCP -IncludeManagementTools

2. Maak een DHCP scope aan:

		Add-DhcpServerv4Scope -Name "Corpnet" -StartRange 10.10.10.100
		-EndRange 10.10.10.200 -SubnetMask 255.255.255.0


3. Set DHCP options

		Set-DhcpServerv4OptionValue -DnsDomain corp.contoso.com -DnsServer
		10.10.10.10 -Router 10.10.10.1


4. Activeer DHCP

		Add-DhcpServerInDC -DnsName corpdc1.corp.contoso.com

5. Voeg DHCP reservations toe:

	    Add-dhcpserverv4reservation –scopeid 10.10.10.0 –ipaddress
	    10.10.10.102 –name test2 –description "Test server" –clientid 12-
	    34-56-78-90-12
	    Get-dhcpserverv4reservation –scopeid 10.10.10.0

6. Voeg DHCP exclusions toe:

	    Add-DhcpServerv4ExclusionRange –ScopeId 10.10.10.0 –StartRange
		10.10.10.110 –EndRange 10.10.10.111
		Get-DhcpServerv4ExclusionRange

**Werken met Private Key Infrastructure en CA's** waarbij we certificaten gaan vertrouwen die certificaten gaat uitdelen aan users en computers die hen toegang zullen geven

1. Installeer certificaat server:

	    Get-WindowsFeature | Where-Object Name -Like *cert*
	    Install-WindowsFeature AD-Certificate -IncludeManagementTools
	    -IncludeAllSubFeature

2. Configure de server als enterprise CA:

	    Install-AdcsCertificationAuthority -CACommonName corp.contoso.com
	    -CAType EnterpriseRootCA -Confirm:$false

3. Install root certificate to trusted root certification authorities store:

   		Certutil –pulse

4. Request machine certificate from CA:

	    Set-CertificateAutoEnrollmentPolicy -PolicyState Enabled -Context
	    Machine -EnableTemplateCheck

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


**Vinden van vervallen computers in AD**

Om recent vervallen computers te vinden in de AD doe::

    $30Days = (Get-Date).AddDays(-30)
    Get-ADComputer -Properties lastLogonDate -Filter 'lastLogonDate
    -lt $30Days' | Format-Table Name, LastLogonDate

Om oudere accounts te vinden, voer dit uit:

    $60Days = (Get-Date).AddDays(-60)
    Get-ADComputer -Properties lastLogonDate -Filter 'lastLogonDate
    -lt $60Days' | Format-Table Name, LastLogonDate

##Managing Hyper-V with PowerShell

**Installeren en configureren van Hyper-V**

1. Folders aanmaken voor de VM en VHDX bestanden

	    New-Item E:\VM -ItemType Directory
	    New-Item E:\VHD -ItemType Directory

2. Indien je geen gebruik maakt van gratis Hyper V server, installeer de role (enkel als WS 2012)

    	Install-WindowsFeature Hyper-V -IncludeManagementTools -Restart

3. Indien je de server vanop afstand beheertd, installeer hypver-v management tools op een ander systeem. Dit zal zowel de GUI als de Powershell administratie tools installeren
	    
	    Install-WindowsFeature RSAT-Hyper-V-Tools

4. Configureer hyper-v om de geselecteerde folders te gebruiken

	    Set-VMHost -ComputerName HV01 -VirtualHardDiskPath E:\vhd `
	    -VirtualMachinePath E:\vm

5. bevestig de settings voor het volgende uit te voeren:
	    
	    Get-VMHost -ComputerName HV01| Format-List *

**NUMA configureren (Non-Uniform Memory Architecture)** zorgt ervoor dat het systeem geheugen zich gaat splitsen tussen de verschillende beschikbare processor (NUMA zones). Wanneer een processor toegang nodig heeft tot het geheugen van een andere processor moet de processor aan de andere processor een request sturen. Default is dit enabled op Hypver-V.

1. Disable NUMA spanning:

	    Invoke-Command -ComputerName HV01 -ScriptBlock {
	    Set-VMHost -NumaSpanningEnabled $false
	    Restart-Service vmms
	    }

2. Bekijk de NUMA status:

    	Get-VMHost -ComputerName HV01 | Format-List *

3. Enable NUMA opnieuw

	    Invoke-Command -ComputerName HV01 -ScriptBlock {
    	Set-VMHost -NumaSpanningEnabled $true
    	Restart-Service vmms
    	}

**Hyper-V VM's aanmaken (wanneer je er meerdere wilt aanmaken)**

1. Maak een nieuwe VM aan

	    New-VM -ComputerName HV01 -Name Accounting02 -MemoryStartupBytes
	    512MB `
	    -NewVHDPath "e:\VM\Virtual Hard Disks\Accounting02.vhdx" `
	    -NewVHDSizeBytes 100GB -SwitchName Production

2. Configureer voor dynamisch geheugen:

	    Set-VMMemory -ComputerName HV01 -VMName Accounting02 `
	    -DynamicMemoryEnabled $true -MaximumBytes 2GB

3. Bekijk de VM configuratie:

 	  	Get-VM -ComputerName HV01 -Name Accounting02 | Format-List *

**De status van een of meerdere VM's eenvoudig beheren**: interessant voor scripting waarbij je bv een lijst opvraagt van de mogelije (inactieve) VM's en waarbij er input wordt gegeven waar je kan opgeven welke VM je wilt selecteren en welke actie je wilt laten gelden voor deze VM('s)

1. Bekijk de status van de huidige VM

    	Get-VM -ComputerName HV01 -Name Accounting02

2. Start de VM (indien nog niet gestart)

		Start-VM -ComputerName HV01 -Name Accounting02

3. Opslaan van de staat van VM

		Save-VM -ComputerName HV01 -Name Accounting02

4. Resume de VM

		Start-VM -ComputerName HV01 -Name Accounting02

5. Shut down VM

		Stop-VM -ComputerName HV01 -Name Accounting02

6. Sluit af 

		Stop-VM -ComputerName HV01 -Name Accounting02 -TurnOff

**Aanpassingen maken aan instellingen van een VM** (RAM, dynamisch geheugen, CPU's)

1. Verander het RAM geheugen (oorspronkelijk 512MB)

		Set-VMMemory -ComputerName HV01 -VMName Accounting02 -StartupBytes 1GB

2. Enable dynamisch geheugen

		Set-VMMemory -ComputerName HV01 -VMName Accounting02 
		-DynamicMemoryEnabled $true -MaximumBytes 2GB -MinimumBytes 128MB -StartupBytes 1GB
	
3. Virtuele CPU's aanpassen (aantal)

		Set-VMProcessor -ComputerName HV01 -VMName Accounting02 -Count 4

**Snapshots** voor de wijzigingen te ontdoen

1. Maak een snapshot

	    Checkpoint-VM -ComputerName HV01 -VMName Accounting02 
	    -SnapshotName "First Snap"		

2. Maak wijzigingen en maak een nieuwe snapshot

	    Checkpoint-VM -ComputerName HV01 -VMName Accounting02 
	    -SnapshotName "Second Snap"

3. Bekijk de snapshots

    	Get-VMSnapshot -ComputerName HV01 -VMName Accounting02

4. Rollback naar vorige snapshot

	    Restore-VMSnapshot -ComputerName HV01 -VMName Accounting02 
	    -Name "Second Snap" -Confirm:$false		

5. Verwijder alle snapshots van de VM

   		 Remove-VMSnapshot -ComputerName HV01 -VMName Accounting02 -Name *

**Migreren van geheugenopslag tussen hosts**

1. Log in op server en open PowerShell console

2. Voer het volgende uit:

	    Move-VM -Name VM1 -DestinationHost HV02 
	    -DestinationStoragePath E:\vm -IncludeStorage

## Managing Storage with PowerShell

**Permissies wijzigen op een bestand (NTFS)**

1. Lees permissie van bestand en schrijf naar bestand

    	$acl = Get-Acl M:\Sales\goals.xls

2. Maak nieuw FileSystemAccessRule voor de gebruiker met de juiste permissies --> joe.smith heeft volledige controle in ace variabele

    	$ace = New-Object System.Security.AccessControl.
    	FileSystemAccessRule "joe.smith","FullControl","Allow"

3. Voeg de permissie toe
	    
	    $acl.SetAccessRule($ace)

4. Pas de permissie toe

  		$acl | Set-Acl M:\Sales\goals.xls

**Ownership nemen over file/directory en permissie aanpassen**: bv iemand heeft iets nodig waar 1 iemand toegang tot heeft maar die is op vakantie.

1. Open Powershell en neem de ownership van het bestand (in dit geval bestand)

	    $folder = "M:\Groups\Projections"
	    takeown /f $folder /a /r /d Y
	    
2. Voeg permissie toe voor diegene die het bestand nodig heeft (hier Joe)

	    $acl = Get-Acl $folder
	    $ace = New-Object System.Security.AccessControl.
	    FileSystemAccessRule `
	    "joe.smith","FullControl","Allow"
	    $acl.SetAccessRule($ace)
	    Set-Acl $folder $acl

3. Recursief overschrijven van permissie

	    Get-ChildItem $folder -Recurse -Force |`
	    ForEach {
	    Get-Acl $acl | Set-Acl -Path $_.FullName
	    }

**Deduplication enablen voor het besparen van geheugen**

1. Installeer de feature
	    
	    Add-WindowsFeature FS-Data-Deduplication

2. Voer ddpeval uit om de mogelijke besparingen te evalueren
	    
	    ddpeval.exe M:\

3. Configureer de schijf voor deduplicatie

   	 	Enable-DedupVolume M:\

4. Set de age (hoe lang een file onveranderd moet blijven vooraleer we gaan dedupliceren)
	    
	    Set-DedupVolume M: -MinimumFileAgeDays 0

5. Start de deduplicatie
	    
	    Start-DedupJob M: -type Optimization

6. Bekijk de status en besparingen 

		Get-DedupJob
		Get-DedupStatus

**Disk quota (hard en soft)**: in voorbeeld soft quota voor e-mail settings met alerts

1. Installeer File System Resource Manager:

  	  	Install-WindowsFeature FS-Resource-Manager -IncludeManagementTools

2. Ga de huidge e-mail config na, en configureer e-mail alerts voor soft en harde quota's.

	    Get-FsrmSetting
	    Set-FsrmSetting -SmtpServer mail.corp.contoso.com `
	    -FromEmailAddress FSAdmin@corp.contoso.com

3. Maak een template gebaseerd op een reeds bestaande template

	    $myQuota = Get-FsrmQuotaTemplate -Name "Monitor 500 MB Share" //gebruik maken van template
	    $myQuota | New-FsrmQuotaTemplate -Name HomeFolders
	    Set-FsrmQuotaTemplate -Name HomeFolders -Threshold $myQuota.
	    Threshold

4. Voer quota toe
	    
	    New-FsrmAutoQuota -Path E:\Users -Template HomeFolders

5. Creeër quota:
	    
	    New-FsrmQuota -Path E:\Groups -Template HomeFolders

6. Genereer quota gebruik report dat naar mail wordt verzonden:

	    New-FsrmStorageReport -Name "Quota Report" -Namespace "E:\" `
	    -ReportTypeQuotaUsage -Interactive -MailTo fsadmin@corp.contoso.
	    com

## Managing Network Shares with PowerShell

**Aanmaken van shares**

1. Bekijk de huidige shares op de server

		Get-SmbShare

2. Maak de eerste basis bestand share aan

		New-Item -Path E:\Share1 -ItemType Directory
		New-SmbShare -Name Share1 -Path E:\share1

3. Maak een tweede share aan die iedereen read access geeft
	    
	    New-Item -Path E:\Share2 -ItemType Directory
	    New-SmbShare -Name Share2 -Path E:\share2 -ReadAccess Everyone `
	    -FullAccess Administrator -Description "Test share"

4. Lijst de share permissies op
	    
	    Get-SmbShare | Get-SmbShareAccess

5. Geef full control aan de eerste share aan Joe Smith

	    Grant-SmbShareAccess -Name Share1 -AccountName CORP\Joe.Smith `
	    -AccessRight Full -Confirm:$false

6. We kunnen ook de toegang blokken tot een gebruiker

		Block-SmbShareAccess -Name Share2 -AccountName CORP\joe.smith `
		-Confirm:$false

**Share mappen in de windows file explorer vanuit Powershell**

1. Use Get-ChildItem to view the contents of a share:
	    
	    Get-ChildItem \\server1\share2

2. Map the share as persistent:

	    New-PSDrive -Name S -Root \\server1\share1 -Persist -PSProvider
	    FileSystem

**NFS remote computer toegang geven tot een directory**

1. Installeer NFS server service
	    
	    Add-WindowsFeature FS-NFS-Service –IncludeManagementTools

2. Maak een NFS share aan
	    
	    New-Item C:\shares\NFS1 -ItemType Directory
	    New-NfsShare -Name NFS1 -Path C:\shares\NFS1

3. Geeft toegang aan remote computer
	    
	    Grant-NfsSharePermission -Name NFS1 -ClientName Server1 `
	    -ClientType host -Permission readwrite -AllowRootAccess $true

## Managing Windows Updates with PowerShell

**Installeren van WSUS (Windows Server Update Services)**

1. Installeer de UpdateServices feature:
	    
	    Install-WindowsFeature UpdateServices -IncludeManagementTools

2. Voer de basis config uit

	    New-Item E:\MyContent -ItemType Directory
	    & 'C:\Program Files\Update Services\Tools\WsusUtil.exe'
	    postinstall contentdir=e:\Mycontent

3. Bekijk de huidige synhronization instellingen:

	    $myWsus = Get-WsuscServer
	    $myWsus.GetSubscription()

4. Indien internet via proxy is en niet rechtstreeks kunnen we configureren om onze proxy te gebruiken (optioneel)

	    $wuConfig = $myWsus.GetConfiguration()
	    $wuConfig.ProxyName = "proxy.corp.contoso.com"
	    $wuConfig.ProxyServerPort = 8080
	    $wuConfig.UseProxy = $true
	    $wuConfig.Save()

5. Voer de initiële synchronisatie uit
	    
	    $mySubs = $myWsus.GetSubscription()
	    $mySubs.StartSynchronizationForCategoryOnly()

**Instellen wat we juist willen updaten adhv WSUS**

1. Instellen welke producten je wilt updaten

	    $myProducts = Get-WsusProduct | `
	    Where-Object {$_.Product.Title -in ('Forefront Client Security', `
	    'SQL Server 2008 R2', 'Office', 'Windows')}
	    $myProducts | Set-WsusProduct

2. Instellen welke soort van updates je wilt toepassen

	    $myClass = Get-WsusClassification | `
	    Where-Object { $_.Classification.Title -in ('Update Rollups', `
	    'Security Updates', 'Critical Updates', 'Definition Updates', `
	    'Service Packs', 'Updates')}
	    $myClass | Set-WsusClassification

3. Start de synchronisatie

		$mySubs = $myWsus.GetSubscription()
		$mySubs.StartSynchronization()

4. Configureer voor automatische synchronisatie (hier één per dag)

	    $mysubs = $myWsus.GetSubscription()
	    $mysubs.SynchronizeAutomatically = $true
	    $mysubs.NumberOfSynchronizationsPerDay = 1
	    $mysubs.Save()

**Overzicht van beschikbare updates krijgen**

1. Maak het object aan om te zoeken

	    $searcher = New-Object -ComObject Microsoft.Update.Searcher
	    $searcher.Online = $true
	    $searcher.ServerSelection = 1

2. Definieer op basis van wat je naar updates wil zoeken
	    
	    $results = $searcher.Search('IsInstalled=0')

3. Toon de resultaten

	    $results.Updates | `
	    Select-Object @{Name="UpdateID"; `
	    Expression={$_.Identity.UpdateID}}, Title

**Installen van updates**

1. Zet de nodige updates client objecten op:
    
    	$searcher = New-Object -ComObject Microsoft.Update.Searcher
    	$updateCollection = New-Object -ComObject Microsoft.Update.
    	UpdateColl
    	$session = New-Object -ComObject Microsoft.Update.Session
    	$installer = New-Object -ComObject Microsoft.Update.Installer

2. Zoek naar missende updates:

    	$searcher.online=$true
    	$searcher.ServerSelection=1
    	$results = $searcher.Search("IsInstalled=0")

3. Zoek naar toepasbare updates:
    
    	$updates=$results.Updates
    	ForEach($update in $updates){ $updateCollection.Add($update) }

4. Download de updates:

    	$downloader = $session.CreateUpdateDownloader()
    	$downloader.Updates = $updateCollection
    	$downloader.Download()

5. Installeer de updates:

    	$installer.Updates = $updateCollection
    	$installer.Install()

**Uninstallen van updates**

1. Lijst de geïnstalleerde updates op die aangeven welke update zal verdwijnen.

    	$searcher = New-Object -ComObject Microsoft.Update.Searcher
    	$searcher.Online = $true
    	$searcher.ServerSelection = 1
    	$results = $searcher.Search('IsInstalled=1')
    	$results.Updates | `
    	Select-Object @{Name="UpdateID"; `
    	Expression={$_.Identity.UpdateID}}, Title

2. Zet de nodige update client objecten op:

    	$updateCollection = New-Object -ComObject Microsoft.Update.
    	UpdateColl
    	$installer = New-Object -ComObject Microsoft.Update.Installer

3. Zorg voor de correcte update bij UpdateID:

    	$searcher.online = $true
    	$searcher.ServerSelection = 1
    	$results = $searcher.Search("UpdateID='70cd87ec-854f-4cdd-8acac272b6fe45f5'")

4. Maak een collectie van toepasbare updates:

    	$updates = $results.Updates
    	ForEach($update in $updates){ $updateCollection.Add($update) }

5. Uninstall de updates:

    	$installer.Updates = $updateCollection
    	$installer.UnInstall()

## Managing Printers with PowerShell

**Een gedeelde printer opzetten** (in dit geval een HP LasterJet met IP van 10.0.0.200.)

1. Installeer de print server
	    
	    Add-WindowsFeature Print-Server –IncludeManagementTools

2. Creeër de poort voor de printer

		Add-PrinterPort -Name Accounting_HP -PrinterHostAddress
		"10.0.0.200"

3. Voeg de driver van de printer toe

		Add-PrinterDriver -Name "HP LaserJet 9000 PCL6 Class Driver"

4. Voeg de printer toe met add-printer

		Add-Printer -Name "Accounting HP" -DriverName "HP LaserJet 9000
		PCL6 Class Driver" -PortName Accounting_HP

5. Deel de printer zodat alle gebruikers (of wie hem nodig heeft) hem kunnen gebruiken

		Set-Printer -Name "Accounting HP" -Shared $true -Published $true

6. Check de werking

		Get-Printer | Format-Table -AutoSize	


**Verkrijgen van informatie over het gebruik van een printer** dit kan misschien helpen te detecteren dat er overbodige printers zijn of dat er bepaalde printers te vaak worden gebruikt met als gevolg dat we extra printers moeten toevoegen.

1. Maak logging op de print server mogelijk adhv de wevutil.exe

	    wevtutil.exe sl "Microsoft-Windows-PrintService/Operational" /
	    enabled:true

2. Maak een query voor succesvolle print opdrachten

		Get-WinEvent -LogName Microsoft-Windows-PrintService/Operational |
		`
		Where-Object ID -eq 307

3. Vervolgens kunnen we nog nagaan wie wanneer iets heeft geprint en hoeveel en op welke printer adhv volgende commando

		Get-WinEvent -LogName Microsoft-Windows-PrintService/Operational |
		`
		Where-Object ID -eq 307 | `
		Select-Object TimeCreated, `
		@{Name="User";Expression={$_.Properties[2].Value}}, `
		@{Name="Source";Expression={$_.Properties[3].Value}}, `
		@{Name="Printer";Expression={$_.Properties[4].Value}}, `
		@{Name="Pages";Expression={$_.Properties[7].Value}}

## Troubleshooting Servers with PowerShell

**Testing if a server is responding**

Ping naar een host.

    Test-Connection -ComputerName corpdc1

Ping naar meerdere hosts.

    Workflow Ping-Host ([string[]] $targets)
    {
    	ForEach -Parallel ($target in $targets)
    	{
    		If (Test-Connection -ComputerName $target -Count 2 -Quiet)
    		{
    			"$target is alive"
    		} Else {
    			"$target is down"
    		}
    	}
    }
    Ping-Host 10.10.10.10, 10.10.10.11

## Inventorying Servers with PowerShell

**Bijhouden van hardware met PowerShell**

Verkrijg schijfinformatie.

    $TargetSystem="."
    $myCim = New-CimSession -ComputerName $TargetSystem
    Get-Disk -CimSession $myCim

Logische schijf

    Get-Disk -CimSession $myCim

Fysieke schijf

    Get-PhysicalDisk -CimSession $myCim

Netwerk adapters

    Get-NetAdapter -CimSession $myCim

System enclosure

    Get-WmiObject -ComputerName $TargetSystem `
    -Class Win32_SystemEnclosure

Computer systeem

    Get-WmiObject -ComputerName $TargetSystem `
    -Class Win32_ComputerSystemProduct

Processor

    Get-WmiObject -ComputerName $TargetSystem -Class Win32_Processor

Fysiek geheugen

    Get-WmiObject -ComputerName $TargetSystem -Class Win32_
    PhysicalMemory

CD-Rom

    Get-WmiObject -ComputerName $TargetSystem -Class Win32_CDromDrive

Sound card

    Get-WmiObject -ComputerName $TargetSystem -Class Win32_SoundDevice

Video card

    Get-WmiObject -ComputerName $TargetSystem `
    -Class Win32_VideoController

BIOS

    Get-WmiObject -ComputerName $TargetSystem -Class Win32_BIOS

**Bijhouden van geïnstalleerde software**

Verkrijg de geïnstalleerde features.

    Get-WindowsFeature | Where-Object Install`State -EQ "Installed"

**Bijhouden van de systeem configuratie**

Verkrijg de netwerk configuratie.

    $TargetSystem="."
    $myCim = New-CimSession -ComputerName $TargetSystem
    Get-NetIPAddress -CimSession $myCim
    Get-NetRoute -CimSession $myCim

Toon de local users en groepen.

    Get-WmiObject -ComputerName $TargetSystem `
    -Query "Select * from Win32_Group where Domain='$TargetSystem'"
    Get-WmiObject -ComputerName $TargetSystemt `
    -Query "Select * from Win32_UserAccount where
    Domain='$TargetSystem'"

Som de services op.
    
    Get-Service
    Get-WmiObject -Class Win32_Service | `
    Select-Object Name, Caption, StartMode, State

Lijst van de lopende proccessen. 

    Get-Process

Toon de shares en printers.

    Get-SmbShare
    Get-Printer
    
Toon de start-up informatie.

    Get-WmiObject -ComputerName $TargetSystem -Class Win32_
    StartupCommand

Toon de tijdzone.

    Get-WmiObject -ComputerName $TargetSystem -Class Win32_TimeZone

Toon registratie informatie.

    Get-WmiObject -ComputerName $TargetSystem -Class Win32_Registry

## Server Backup

**Configuring backup policies**

Installeerd Windows Server Backup feature:

    Add-WindowsFeature Windows-Server-Backup -IncludeManagementTools

Maak een backup policy:

    $myPol = New-WBPolicy

Voeg de backup bronnen toe.

    $myPol | Add-WBBareMetalRecovery
    $myPol | Add-WBSystemState
    $sourceVol = Get-WBVolume C:
    Add-WBVolume -Policy $myPol -Volume $sourceVol

Definieer het backup target.

    $targetVol = New-WBBackupTarget -Volume (Get-WBVolume E:)
    Add-WBBackupTarget -Policy $myPol -Target $targetVol

Definieer de planning.

    Set-WBSchedule -Policy $myPol -Schedule ("12/17/2012 9:00:00 PM")

Slaag de policy op.

    Set-WBPolicy -Policy $myPol

**Initiating backups manually**

Initieer de default backup policy.

    Get-WBPolicy | Start-WBBackup

Monitor van backup.:

    Get-WBSummay
Maak één backup van C:\InetPub:

    $myPol = New-WBPolicy
    $mySpec = New-WBFileSpec -FileSpec "C:\InetPub"
    Add-WBFileSpec -Policy $myPol -FileSpec $mySpec
    $targetVol = New-WBBackupTarget -Volume (Get-WBVolume E:)
    Add-WBBackupTarget -Policy $myPol -Target $targetVol
    Start-WBBackup -Policy $myPol

**Restoring files**

Identifieer de backup set.

    Get-WBBackupSet
    
    $myBackup = Get-WBBackupSet | `
    Where-Object VersionId -eq 03/03/2013-19:31

Voer de recovery uit op één file.

    Start-WBFileRecovery -BackupSet $myBackup `
    -SourcePath c:\temp\perfcounter.csv

Herstellen van een folder.

    Start-WBFileRecovery -BackupSet $myBackup `
    -SourcePath c:\inetpub\ -Recursive -TargetPath c:\

**Restoring Windows system state**

Identifiëren van de backup set.

    Get-WBBackupSet
    $myBackup = Get-WBBackupSet | `
    Where-Object VersionId -eq 03/03/2013-19:31

Initieer de system state restore.

    Start-WBSystemStateRecovery -BackupSet $myBackup

Select Y om het systeem te rebooten.












































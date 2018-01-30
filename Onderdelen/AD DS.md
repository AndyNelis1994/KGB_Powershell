#AD DS Nicolai
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

#AD DS Andy

###Configureer het ip address van de server

**Geef de adapter alias en index weer.**

```
Get-NetAdapter | Get-Member
```

####Configureer een statisch ip address

Om een statisch ip address te gebruiken moeten we eerst DHCP uitzetten. Dan pas kunnen we de ipv4 en ipv6 adressen toekennen. In dit voorbeeld gebruiken we 192.168.10.0/24 als het ipv4 subnet, en 2001:db8:0:10::/64 als ipv6 subnet.

**Om DHCP uit te schakellen op netwerkadapter 10, gebruik we het volgende commando.**

```
Set-NetIPInterface -InterfaceAlias "10 Network" -DHCP Disabled -PassThru
```

Door de -PassThru parameter mee te geven geeft powershell een status terug van de ip interface. Standaard wordt die niet weer gegeven dus weten we ook niet of de aanpassing succesvol was.

**Nu kunenn we het ipv4 address 192.168.10.2 instellen met het volgende commando.**

```
New-NetIPAddress -AddressFamily IPv4 -InterfaceAlias "10 Network" -IPAddress 192.168.10.2 -PrefixLength 24 -DefaultGateway 192.168.10.1
```
     
**Nu kunenn we het ipv6 address 2001:db8:0:10::2 instellen met het volgende commando.**

```
New-NetIPAddress -AddressFamily IPv6 -InterfaceAlias "10 Network" -IPAddress 2001:db8:0:10::2 -PrefixLength 64 -DefaultGateway 2001:db8:0:10::1
```
**Stel het DNS ip address in met volgend commando**

```
Set-DnsClientServerAddress -InterfaceAlias "10 Network" -ServerAddresses 192.168.10.2,2001:db8:0:10::2
```

**Bekijk de veranderingen aan network adapter 10.**

```
Get-NetIPAddress -InterfaceAlias "10 Network
```

**Pas de servernaam aan.**

```
Rename-Computer -NewName servernaam -Restart -Force -PassThru
```

###Installeer Active Directory Domain Services

Voor we de server tot domein controller kunnen promoten moeten we de Active Directory Domain Services role op de server installeren. Het volgende commando zal de AD DS role installeren, inclusief the management tools.

```
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```
###Maak het forest aan (dcpromo)

####Update Windows PowerShell help

Voor je verder gaat is het best dat je de help files van Powershell update. De files worden namenlijk regelmatig aangepast.

Om de files te updaten heb je administrator rechten nodig. Gebruik het volgende commando om de help files up te daten.

```
Update-Help
```

####Test de serveromgeving 

Voor je het forest aanmaakt, test je best of je serveromgeving aan de eisen van een forest voldoet. het volgende script test dit. Sla dit script op als 'Test-myForestCreate.ps1'. Het script staat ook in het mapje scripts op github.

```
Import-Module ADDSDeployment 
Test-ADDSForestInstallation `
     -DomainName 'TreyResearch.net' `
     -DomainNetBiosName 'TREYRESEARCH' `
     -DomainMode 6 `
     -ForestMode 6 `
     -NoDnsOnNetwork `
     -NoRebootOnCompletion
```
Voer het script dan uit in powershell met volgend commando.

```
Test-myForestCreate.ps1
```
Bekijk de uitvoer en zo nodig de waarschuwingen die het script geeft.

####maak het eerste domein en forest aan

Nu staat alles klaar om het forrest aan te maken. Het commando om het forrest aan te maken is heel gelijkend op het testscript dat we net uitgevoerd hebben. Het commando is als volgt.

```
Install-ADDSForest `
     -DomainName 'domeinnaam.net' `
     -DomainNetBiosName 'DOMEINNAAM' `
     -DomainMode 6 `
     -ForestMode 6 `
     -NoDnsOnNetwork `
     -SkipPreChecks `
     -Force
```

Het `-Force` commando zorgt er voor dat tijdens het aanmaken van het forrest geen confirmatie prompts tevoorschijn komen. Desalniettemin zal je toch een prompt krijgen om het Directory Services Restore Mode (DSRM) password in te geven. Bij het aanmaken van veel verschillende forests is dit niet zo interessant en kunnen we deze prompt ook vermijden door de DSMR al mee te geven. Dit doen we met volgend stukje code.

```
$pwdSS = ConvertTo-SecureString -String 'P@ssw0rd!' -AsPlainText -Force
```
De waarde van parameters `DomainmMode` en `ForestMode` kunnen we met volgende tabel verklaren. 

| Functional level        | Nummer        | String   |
| -------------           |:-------------:| -----:   |
| Windows Server 2003     | 2             | win2003  |
| Windows Server 2008     | 3             | win2008  |
| Windows Server 2008 R2  | 4             | win2008R2|
| Windows Server 2012     | 5             | win2012  |
| Windows Server 2012 R2  | 6             | win2012R2|

Het forest en domein hebben dus als functional level Windows Server 2012 R2.

**Informatie bekijken domein en forest**

Na het aanmaken van het domein en de forest kan je met volgend script de Forest Mode, Domain Mode, en Schema Version bekijken. Sla het script op als Get-myADVersion.ps1. Het script staat ook in het script mapje op github.

```
<#
.Synopsis
Get the current Schema version and Forest and Domain Modes
.Description
The Get-myADVersion script queries the AD to discover the current AD schema version,
and the forest mode and domain mode. If run without parameters, it will query the
current AD context, or if a Domain Controller is specified, it will query against
that DC's context. Must be run as a user with sufficient privileges to query AD DS.
.Example
Get-myADVersion
Queries against the current AD context.
.Example
Get-myADVersion -DomainController Trey-DC-02
Gets the AD versions for the Domain Controller "Trey-DC-02"
.Parameter DomainController
Specifies the domain controller to query. This will change the response to match
the AD context of the DC.
.Inputs
[string]
.Notes
    Author: Charlie Russel
 Copyright: 2015 by Charlie Russel
          : Permission to use is granted but attribution is appreciated
   Initial: 3/7/2015 (cpr)
   ModHist:
          :
#>
[CmdletBinding()]
Param(
     [Parameter(Mandatory=$False,Position=0)]
     [string]
     $DomainController
     )

if ($DomainController) {
   $AD = Get-ADRootDSE -Server $DomainController
   Get-ADObject $AD.SchemaNamingContext -Server $DomainController `
                                        -Property ObjectVersion
} else {
   $AD = Get-ADRootDSE
   Get-ADObject $AD.SchemaNamingContext -Property ObjectVersion
}
$Forest = $AD.ForestFunctionality
$Domain = $AD.DomainFunctionality

# Use a Here-String to print out the result.
$VersionCodes = @"

Forest: $Forest
Domain: $Domain


Where the Schema version is:
72 = Windows Server Technical Preview Build 9841
69 = Windows Server 2012 R2
56 = Windows Server 2012
47 = Windows Server 2008 R2
44 = Windows Server 2008
31 = Windows Server 2003 R2
30 = Windows Server 2003
13 = Windows 2000
"@
$VersionCodes
```
Als je het script wilt uitvoeren in Powershell geef je het volgende commando in.

```
Get-myADVersion
```
##Meedere domain controllers
uitwerken
##Clone AD DS 

###Bestaande AD DS toevoegen aan Cloneable Domain Controllers security group.
Voor we de AD DS kunnen clonen moeten hem eerst een security group toevoegen zodat hij de rechten heeft om gecloned te worden. Na het clonen voltooid is moet de AD DS terug uit de groep verwijderd worden.


```
Add-ADGroupMember -Identity "Cloneable Domain Controllers" `
                  -Members (Get-ADComputer -Identity trey-dc-04).SAMAccountName `
                  -PassThru
```

Resultaat:

```
DistinguishedName : CN=Cloneable Domain Controllers,CN=Users,DC=TreyResearch,DC=net
GroupCategory     : Security
GroupScope        : Global
Name              : Cloneable Domain Controllers
ObjectClass       : group
ObjectGUID        : b12b23c1-499b-4dbe-8206-846a17cd2df2
SamAccountName    : Cloneable Domain Controllers
SID               : S-1-5-21-910751839-3601328731-670513855-522
```
Om de AD DS weer uit de groep te verwijderen gebruik je volgend commando.

```
Remove-ADGroupMember -Identity "Cloneable Domain Controllers" `
                     -Members (Get-ADComputer -Identity trey-dc-04).SAMAccountName `
                     -PassThru
```

###Problemen clonen vermijden
Er zijn een paar problemen die kunnen opduiken tijdens het clonen. Deze problemen kunnen we omzeilen door wat verandering door te voeren aan de AD DS.

***Programma's en services verwijderen***<br>
Als eerste kunnen geïnstalleerde programma's of services voor problemen zorgen.
Gebruik het volgende commando om te kijken of er programma's of services voor problemen kunnen zorgen.

```
Get-ADDCCloningExcludedApplicationList
```

Als een programma of service voor problemen zou kunnen zorgen, verwijderd deze dan.
Antivirus programma's of services zoals windows defender kunnen meestal voor problemen zorgen.

verwijder een programma of service met volgend commando. Als voorbeeld gebruiken we windows defender.

```
Uninstall-WindowsFeature -Name Windows-Defender
Restart-Computer -Wait 0
```

Eens de programma's of services verwijderd zijn en de server weer geboot is gaan we een .xml file maken van de exclusion list.
Dit doen we met volgend commando.

```
Get-ADDCCloningExcludedApplicationList -GenerateXML
The inclusion list was written to 'C:\Windows\NTDS\CustomDCCloneAllowList.xml'.
```

***Verwijder managed service accounts***<br>
Stand-alone managed service accounts worden niet ondersteund tijdens het clonen. Verwijder deze accounts dus en maak ze weer aan als het clonen voltooid is.

Bekijk met volgend commando of er zo een accounts op je server bevinden. Als dit het geval is verwijder ze dan.
```
Get-ADComputer -Identity trey-dc-04 | Get-ADComputerServiceAccount
```
Group managed service accounts zijn wel toegestaan bij cloning. Deze hoeven dus niet verwijderd te worden.

###Maak DCCloneConfig.xml aan
We hebben een .xml file nodig die het cloning procces controleert. Gebruik hiervoor het `New-ADDCCloneConfigFile` cmdlet.

```
New-ADDCCloneConfigFile -Static `
                        -CloneComputerName trey-dc-10 `
                        -IPv4Address 192.168.10.10 `
                        -IPv4SubnetMask 255.255.255.0 `
                        -IPv4DefaultGateway 192.168.10.1 `
                        -IPv4DNSResolver 192.168.10.2
```
Als dit commando geslaagd is op de bestaande AD DS, sluit deze dan af.

###Maak de cloned domain controller aan
Clone de AD DS met het volgende commando op de doel server. Pas aan waar nodig.

```
Copy-Item "D:\VMs\trey-dc-04\Virtual Hard Disks\trey-dc-04-system.vhdx" `
           "V:\trey-dc-10\Virtual Hard Disks\trey-dc-10-system.vhdx"
$ClonedDC=New-VM -Name trey-dc-10 `
           -MemoryStartupBytes 1024MB `
           -Generation 2 `
           -BootDevice VHD `
           -Path "V:\" `
           -VHDPath "V:\trey-dc-10\Virtual Hard Disks\trey-dc-10-system.vhdx" `
           -Switch "Local-10"
Set-VM -VM $ClonedDC -ProcessorCount 2 -DynamicMemory -PassThru
Start-VM $ClonedDC
```

###Beheer FSMO roles
Er zijn 5 flexible single master operations in een windows domein. Elke role speelt een belangrijke rol in het onderhoud van het domein. Als een nieuw domein of forest aangemaakt wordt, worden deze roles automatisch op de eerste AD DS geïnstalleerd.
In een klein domein is dit perfect haalbaar maar wanneer het domein groeit is het beter om deze roles op meerdere AD DS bij te houden.<br><br>
De 5 roles zijn: `SchemaMaster`,`DomainNamingMaster`,`PDCEmulator`,`RIDMaster`,`InfrastructureMaster`.

***Bekijk waar roles zich bevinden***<br>
Om te kijken waar de roles zich bevinden gebruik dan het commando.

```
Get-ADDomain -Identity treyresearch.net
```

***Roles verplaatsen***<br>
Om roles te verplaatsen gebruiken we volgend commando.

```
Move-ADDirectoryServerOperationMasterRole `
                         -OperationMaster PDCEmulator,RIDMaster,InfrastructureMaster `
                         -Identity trey-dc-04
```

Controleer dan met het volgend commando.

```
Move-ADDirectoryServerOperationMasterRole `
                         -Identity 'trey-dc-09' `
                         -OperationMasterRole SchemaMaster,DomainNamingMaster
```

***Roles overnemen***
De meest verkozen manier om roles te verplaatsen is door voorgaande manier te hanteren. Maar soms is dit niet mogelijk, bijvoorbeeld bij disaster recovery of domein migratie. Op zo een momenten kan het zijn dat je geen roles zomaar kan verplaatsen. Maar tot zo lang er 1 DC op het domein actief is kan je wel zijn roles overnemen. Dit gebeurd op dezelfde manier dan hierboven beschreven. Het enige verschil is dat we het `-Force` commando gebruiken.

```
Move-ADDirectoryServerOperationMasterRole `
                    -OperationMaster PDCEmulator,RIDMaster,InfrastructureMaster `
                    -Identity trey-dc-02 `
                    -Force
```

Hou er zeker rekening mee dat eens roles van een bepaalde DC overgenomen worden, deze DC NOOIT meer in gebruik kan worden genomen! DC `trey-dc-04` moet nu verwijderd worden omdat van hem roles overgenomen hebben. Gebruik het volgende commando om de DC te verwijderen.

```
Remove-VM -Name trey-dc-04 ; rm -r D:\VMs\trey-dc-04
```

##read-only domain controllers (RODCs)

###Domein voorbereiden


***Group policy voorbereiden***<br> 
Dit moet enkel als eerste keer RODC.

```
adprep /domainprep /gpprep
```

***Domein voorbereiden***

```
adprep /rodcprep
```

###RODC account voorbereiden

```
Add-ADDSReadOnlyDomainControllerAccount `
      -DomainControllerAccountName "trey-rodc-200" `
      -DomainName "TreyResearch.net" `
      -SiteName "Default-First-Site-Name" `
      -DelegatedAdministratorAccountName "TREYRESEARCH\Stanley" `
      -InstallDNS `
      -AllowPasswordReplicationAccountName "Dave","Alfie","Stanley"
```

###Doelserver voorbereiden

***Netwerkadapter aanpassen***<br>
Gebruik het script `Set-myIP.ps1` om de netwerkadapter correct te configureren. Pas natuurlijk aan waar nodig. Het script kan je vinden in het mapje scripts.

```
# Quick and dirty IP address setter

Param ($IP4,$IP6)

$Network = "192.168.10."
$Network6 = "2001:db8:0:10::"
$IPv4 = $Network + "$IP4"
$IPv6 = $Network6 + "$IP6"
$Gateway4 = $Network + "1"
$Gateway6 = $Network6 + "1"

$Nic = Get-NetAdapter -name Ethernet
$Nic | Set-NetIPInterface -DHCP Disabled
$Nic | New-NetIPAddress -AddressFamily IPv4 `
                        -IPAddress $IPv4 `
                        -PrefixLength 24 `
                        -type Unicast `
                        -DefaultGateway $Gateway4
Set-DnsClientServerAddress -InterfaceAlias $Nic.Name `
                           -ServerAddresses 192.168.10.2,2001:db8:0:10::2
$NIC |  New-NetIPAddress -AddressFamily IPv6 `
                         -IPAddress $IPv6 `
                         -PrefixLength 64 `
                         -type Unicast `
                         -DefaultGateway $Gateway6
ipconfig /all
```
Om het script uit te voeren geef je eerst de naam in gevolgd door 2 parameters. 
De eerste parameter stelt het laatste cijfer van het ipv4 adres voor. De tweede parameter het laatste hexadecimaal cijfer van het ipv6 adres.

Voorbeeld:

```
Set-myIP -IP4 200 -IP6 C8
```
IPv4 Address. . . . . . . . . . . : 192.168.10.200<br>
IPv6 Address. . . . . . . . . . . : 2001:db8:0:10::c8

***naam server aanpassen***<br>
Naam van de server moet aangepast worden naar de naam van de RODC.
```
Rename-Computer -NewName trey-rodc-200 -Restart -Force
```
###Maak de RODC target server aan
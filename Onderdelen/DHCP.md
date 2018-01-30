#DHCP Nicolai

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

5. Voeg DHCP reservations toe (reservaties zijn voor bepaalde servers bedoeld, hierdoor zullen ze altijd dezelfde IP-adres hebben):

	    Add-dhcpserverv4reservation –scopeid 10.10.10.0 –ipaddress
	    10.10.10.102 –name test2 –description "Test server" –clientid 12-
	    34-56-78-90-12
	    Get-dhcpserverv4reservation –scopeid 10.10.10.0

6. Voeg DHCP exclusions toe (exclusions zijn een reeks IP-adressen die niet toegekend worden aan werkstations):

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

##DHCP Andy

###DHCP intialiseren

***Role installeren***<br>
Om DHCP te kunnen gebruiken is een role nodig. Deze moet dus eerst geïnstalleerd worden.

``` 
Install-WindowsFeature -ComputerName trey-dns-03 `
                       -Name DHCP `
                       -IncludeAllSubFeature `
                       -IncludeManagementTools
```  

***User groups aanmaken***<br>
Eens de role geïnstalleerd is moeten er 2 lokale groepen aangemaakt worden op de DHCP server: DHCP Users en DHCP Administrators. We doen dit aan de hand van volgende commando's.

``` 
$connection = [ADSI]"WinNT://trey-dns-03"
$lGroup = $connection.Create("Group","DHCP Administrators")
$lGroup.SetInfo()
$lGroup = $connection.Create("Group","DHCP Users")
$lGroup.SetInfo()
``` 

***DHCP activeren in AD DS***<br>
De laatste stap die genomen moet worden voor de je de DHCP kan configureren is de DHCP te activeren in de AD DS als een geautoriseerde server. Dit doen we door gebruik te maken van het `Add-DhcpServerInDC` cmdlet. De `-DnsName` kan zowel met het ip adres als de DNS naam ingevuld worden.

``` 
Add-DhcpServerInDC -DnsName 'trey-dns-03' -PassThru
``` 

###DHCP configureren
Nu de role geïnstalleerd is en alles klaar gezet is kunnen we beginnen met het configureren van de DHCP server. 

***Ipv4 scope maken***<br>
Om een Ipv4 scope aan te maken gebruiken we het `Add-DhcpServerv4Scope` cmdlet.
Als voorbeeld maken we een scope aan van 200 ip adressen.

``` 
Add-DhcpServerv4Scope -Name "Trey-Default" `
                      -ComputerName "trey-dns-03" `
                      -Description "Default IPv4 Scope for Lab" `
                      -StartRange "192.168.10.1" `
                      -EndRange   "192.168.10.200" `
                      -SubNetMask "255.255.255.0" `
                      -State Active `
                      -Type DHCP `
                      -PassThru
``` 
Als het aanmaken van de scope goed verlopen is krijg je volgende uitvoer te zien.

``` 
ScopeId       SubnetMask    Name         State   StartRange    EndRange
LeaseDuration
-------       ----------    ----         -----   ----------    --------        ------
-------
192.168.10.0  255.255.255.0 Trey-Default Active  192.168.10.1  192.168.10.200
8.00:00:00
``` 
***Ipv4 exclusion range***<br>
Vervolgens maken we nu een exclusion range aan die we kunnen gebruiken voor servers op het netwerk die een statisch ip adres hebben. We maken in de voorbeeld een exclusion range aan voor 20 statische ip adressen. We gebruiken hiervoor het `Add-DhcpServerv4ExclusionRange` cmdlet.

``` 
Add-DhcpServerv4ExclusionRange -ScopeID "192.168.10.0" `
                               -ComputerName "trey-dns-03" `
                               -StartRange "192.168.10.1" `
                               -EndRange   "192.168.10.20" `
                               -PassThru
``` 
Als het aanmaken van de exclusion range goed verlopen is krijg je volgende uitvoer te zien.

```
ScopeId              StartRange           EndRange
-------              ----------           --------
192.168.10.0         192.168.10.1         192.168.10.20
```

***IpV4 scope opties***<br>
Als laatste moeten nog enkele opties op de scope aangepast worden. Dit doen we met het `Set-DhcpServerv4OptionValue` cmdlet.

``` 
Set-DhcpServerv4OptionValue -ScopeID 192.168.10.0 `
                            -ComputerName "trey-dns-03" `
                            -DnsDomain "TreyResearch.net" `
                            -DnsServer "192.168.10.2" `
                            -Router "192.168.10.1" `
                            -PassThru
``` 

***IpV6 configuratie***<br>
De IpV6 configuratie is volledig analoog aan de IpV4 configuratie. Het enige verschil is dat in de cmdlets de 4 wordt vervangen door een 6 en dat de Ipv4 adressen natuurlijk worden vervangen door IpV6 adressen.


***Meer configuratie DHCP***<br>
Er zijn nog meer dingen die je kan configureren aan je DHCP. Gebruik hiervoor de cmdlet set `Set-DhcpServer`. Gebruik de help functie om de verschillende cmdlets te bekijken.

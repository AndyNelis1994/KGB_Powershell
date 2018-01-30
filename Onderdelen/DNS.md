#DNS Nicolai

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

#DNS Andy

Het is onmogelijk een AD DS te draaien zonder het gebruik van een DNS. 
In het volgende hoofdstuk ga ik dieper in over hoe een DNS tot stand wordt gebracht met Powershell.

### Nieuwe primary forward lookup zone aanmaken
Voor het aanmaken van een nieuwe primary zone gebruiken we het Add-DnsServerPrimaryZone cmdlet. Vervang bij `Name` en `ComputerName` de waarden die van toepassing zijn.
 
```
Add-DnsServerPrimaryZone -Name 'dnsnaam.com' `
                         -ComputerName 'computernaam.treyresearch.net' `
                         -ReplicationScope 'Domain' `
                         -DynamicUpdate 'Secure' `
                         -PassThru
                         
```
### Nieuwe reverse lookup zone aanmaken
Een reverse lookup zone aanmaken is heel gelijkend op het aanmaken van een forward lookup zone. Het enige verschil is dat we idpv parameter `ComputerName` we `NetworkID` gebruiken. We maken ook weer gebruik van het `DnsServerPrimaryZone` cmdlet.

Pas de `NetworkID` aan naar de waarde die van toepassing is.

***Reverse lookup zone met ipv4***

```
Add-DnsServerPrimaryZone -NetworkID 192.168.10.0/24 `
                         -ReplicationScope 'Forest' `
                         -DynamicUpdate 'NonsecureAndSecure' `
                         -PassThru
```
***Reverse lookup zone met ipv6***

```
Add-DnsServerPrimaryZone -NetworkID 2001:db8:0:10::/64 `
                         -ReplicationScope 'Forest' `
                         -DynamicUpdate 'Secure' `
```
### Lookup zone aanpassen

```
Set-DnsServerPrimaryZone -Name 'TailspinToys.com' `
                         -Notify 'NotifyServers' `
                         -NotifyServers "192.168.10.201","192.168.10.202" `
                         -PassThru
```
### Primary lookup zone exporteren
Het is handig wanneer je een primary DNS zone als file hebt voor disaster recovery of voor gebruik in testomgevingen. Om dit te doen hanteren we de `Export-DnsServerZone` cmdlet.

***Ipv4 DNS zone exporteren***

***Ipv6 DNS zone exporteren***

```
Export-DnsServerZone -Name '0.1.0.0.0.0.0.0.8.b.d.0.1.0.0.2.ip6.arpa' `
                     -Filename '0.1.0.0.0.0.0.0.8.b.d 0.1.0.0.2.ip6.arpa.dns'
```
De file wordt opgeslagen in de  `%windir%\system32\dns` directory. Het bestand bevat alle informatie om de DNS zone te reacreëren.

### Beheer secondary zones

Secondary DNS zones are primarily used for providing distributed DNS resolution when you are using traditional file-based DNS zones. Secondary DNS zones are used for both forward lookup and reverse lookup zones. Een secondary DNS zone is read-only. 

***Secondary zone aanmaken aan de hand van geëxporteerde file***<br>
Om een secondary zone aan te maken gebruiken we het cmdlet `Add-DnsServerSecondaryZone`. In dit voorbeeld gebruiken we het geëxporteerd bestand in de `%windir%\system32\dns` directory. 

```
Add-DnsServerSecondaryZone –Name 0.1.0.0.0.0.0.0.8.b.d.0.1.0.0.2.ip6.arpa `
                           -ZoneFile "0.1.0.0.0.0.0.0.8.b.d.0.1.0.0.2.ip6.arpa.dns" `
                           -LoadExisting `
                           -MasterServers 192.168.10.2,2001:db8:0:10::2 `
                           -PassThru
```
***Secondary zone aanpassen***<br>
Hierbij gebruiken we het `Set-DnsServerSecondaryZone` cmdlet.

```
Set-DnsServerSecondaryZone -Name 0.1.0.0.0.0.0.0.8.b.d.0.1.0.0.2.ip6.arpa `
                           -MasterServers 192.168.10.3,2001:db8:0:10::3 `
                           -PassThru
```

###Stub zones
Stub DNS zones houden enkel de nodige records bij om een namesever te lokaliseren van van een zone. 
Hun functie is om bij te houden welke servers  gemachtigd zijn om een child zone te betreden, zonder volledige records bij te houden van die child zone. Stub zones kunnen voor zowel forward als reverse lookup zones gebruikt worden.

Stub zones worden vaak gebruikt bij secondary zones. Secondary zones houden namelijk alle records bij van een zone waardoor een indringer in het bezit kan geraken van deze belangrijke informatie. Door het gebruik van Stub Zone heeft een potentiële indringer enkel toegang tot ip adressen en de namen van hun DNS servers.

***Stub Zone aanmaken***<br>
Om een Stub zone aan te maken gebruiker we het `Add-DnsServerStubZone` cmdlet. Als voorbeeld heeft de master server ip 192.168.10.4.

```
Add-DnsServerStubZone -Name TailspinToys.com `
                      -MasterServers 192.168.10.4 `
                      -ReplicationScope Domain `
                      -PassThru
```
***Stub Zone aanpassen***<br>
Om een Stub Zone aan te passen gebruiken we het `Set-DnsServerStubZone` cmdlet. 

```
Set-DnsServerStubZone -Name TailspinToys.com `
                      -LocalMasters 192.168.10.201,192.168.10.202 `
                      -PassThru
```

###Conditional forwards configureren
Conditional forwards worden gebruikt om DNS request te forwarden naar specifieke DNS domeinen. 

Voorbeeld:<br>
Je hebt meerder interne DNS domeinen en 1 DNS krijgt een request dat niet voor hem bestemd is. De DNS gaat dan kijken in zijn cache of hij de bestemming van de request heeft. Als dit niet zo is gaat de DNS na of er een conditional forward is geconfigureerd voor de DNS request. Als dit het geval is zal de DNS de request forwarden naar de ingestelde conditional forward.

***Conditional forwarder aanmaken***<br>
Als voorbeeld gaan we een conditional forward configureren voor TreyResearch.net. We gebruiken het `Add-DnsServerConditionalForwarderZone` cmdlet.

```
Add-DnsServerConditionalForwarderZone -Name treyresearch.net `
                                      -MasterServers 192.168.10.2,2001:db8::10:2 `
                                      -ForwarderTimeout 5 `
                                      -ReplicationScope "Forest" `
                                      -Recursion $False `
                                      -PassThru
```

***Conditional forwarder aanpassen***<br>
Om een conditional forwarder aan te passen gebruik je het `Add-DnsServerConditionalForwarderZone` cmdlet.

```
Add-DnsServerConditionalForwarderZone -Name treyresearch.net `
                                      -MasterServers 192.168.10.3,2001:db8::10:3 `
                                      -PassThru
```

###Beheer zone delegation
Zone delegation wordt gebruikt om grote zones te verdelen in kleinere subzones om de performantie te verhogen.

***Zone delagation aanmaken***<br>
Om een zone delegation aan te maken gebruiken we het `Add-DnsServerZoneDelegation` cmdlet.

```
Add-DnsServerZoneDelegation -Name TreyResearch.net `
                            -ChildZoneName Engineering `
                            -IPAddress 192.168.10.12,2001:db8:0:10::c `
                            -NameServer trey-engdc-12.engineering.treyresearch.net `
                            -PassThru
```

***Zone delagation aanpassen***                            
Om een zone delegation aan te passen gebruiken we het `Set-DnsServerZoneDelegation` cmdlet.                            
 
```
Set-DnsServerZoneDelegation -Name TreyResearch.net `
                            -ChildZoneName Engineering `
                            -IPAddress 192.168.10.13,2001:db8:0:10::d `
                            -NameServer trey-engdc-13.engineering.treyresearch.net `
                            -PassThru
```                            

###Beheer DNS records
Een DNS server doet veel meer dan enkel een servernaam vertalen naar een ip-adres. De DNS houd bijvoorbeeld informaitie bij die services zoals mail servers nodig hebben om de server van bestemming te vinden. Deze informatie wordt bijgehouden in records. Er zijn veel verschillende soorten records en daarom ga ik ze niet allemaal overlopen. Om ze op te vragen kan je het volgende help commando gebruiken.

```   
Get-Help Add-DnsServerResourceRecord* | ft -auto Name,Synopsis
```   

Als uitvoer krijg je dan:

``` 
Name                              Synopsis
----                              --------
Add-DnsServerResourceRecord       Adds a resource record of a specified type to...
Add-DnsServerResourceRecordA      Adds a type A resource record to a DNS zone.
Add-DnsServerResourceRecordAAAA   Adds a type AAAA resource record to a DNS server.
Add-DnsServerResourceRecordCName  Adds a type CNAME resource record to a DNS  zone.
Add-DnsServerResourceRecordDnsKey Adds a type DNSKEY resource record to a DNS zone.
Add-DnsServerResourceRecordDS     Adds a type DS resource record to a DNS zone.
Add-DnsServerResourceRecordMX     Adds an MX resource record to a DNS server.
Add-DnsServerResourceRecordPtr    Adds a type PTR resource record to a DNS server.

``` 

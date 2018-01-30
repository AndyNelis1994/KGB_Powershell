#Algemeen
##Help
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

##Understanding PowerShell Scripting

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

    Start-Transcript -Path C:\Gebruikers/Nicolai/Documenten
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

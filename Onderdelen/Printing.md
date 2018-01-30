#Printing

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
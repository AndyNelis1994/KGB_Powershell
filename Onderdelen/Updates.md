#UPDATES

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

#Storage

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

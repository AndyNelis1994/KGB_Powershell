#Network shares

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
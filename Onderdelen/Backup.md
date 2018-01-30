#Backup 

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

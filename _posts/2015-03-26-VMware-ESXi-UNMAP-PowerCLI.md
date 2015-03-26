---
layout: post
title: Utiliser PowerCLI et la primitive UNMAP pour réclamer de l'espace libre sur des LUN thin
---

Nous avons vu dans un [article précédent](http://blog.okcomputer.io/2015/02/26/R%C3%A9clamer-espace-libre-lun-thin-ESXi-UNMAP/) comment réclamer l'espace libre en ligne de commande. Cette méthode, bien que fonctionnelle, nécessite de lancer manuellement la commande pour chaque datastore concerné.

Afin d'automatiser le process, je vous propose d'utiliser PowerCLI:

```
Param (
  [Parameter(Mandatory=$true)]
  [string]$VMHost,
  [string]$Datastore="*",
  [int]$asyncUnmapFilePourcentage=50
)

# Récupere les informations sur le ou les datastore(s)
$Datastores = Get-Datastore -Name $Datastore

# Réclamation de l'espace libre sur chaque datastore
foreach ($DS in $Datastores) {

  #Calcul de la taille du fichier asyncUnmapFile en fonction du pourcentage passé en argument. Par défaut 50%
  [int]$FreeSpace = $DS.FreeSpaceMB
  $asyncUnmapFileSize = [math]::Round($FreeSpace / $asyncUnmapFilePourcentage)

  # Esxcli
  $esxcli = get-vmhost -Name $VMHost | Get-EsxCli

  Write-Host "Reclamation de l'espace libre sur $($DS.name) par bloc de $asyncUnmapFileSize Mo et depuis le serveur $VMHost"

  #Execution de la commande avec gestion des erreurs
  Try {
    $result = $esxcli.storage.vmfs.unmap($asyncUnmapFileSize, $DS.name, $null)
  }
  Catch [VMware.VimAutomation.Sdk.Types.V1.ErrorHandling.VimException.ViError] {
    Write-Host "Erreur lors de la réclamation de l'espace libre sur $($DS.name)"
    Write-Host $_.Exception.Message
  }
}
```
L'idée est de réclamer l'espace libre sur tous les datastores disponibles depuis le serveur ESXi passé en argument *-VMHost*. La taille du fichier asyncUnmapFileSize est définie en fonction de l'espace libre restant sur le datastore et un pourcentage qui par défaut est de 50% mais qui peut aussi être passé en argument *-asyncUnmapFilePourcentage*. Il est possible de préciser sur quel datastore on veut exécuter le script grâce à l'argument *-Datastore*.

Vous trouverez le script complet sur [GitHub](https://github.com/okcomputerpro/vmware-powercli/blob/master/UNMAP/ReclaimUnusedSpace.ps1).

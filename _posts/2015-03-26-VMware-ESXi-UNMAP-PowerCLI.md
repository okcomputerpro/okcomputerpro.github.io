---
layout: post
title: Utiliser PowerCLI et la primitive UNMAP pour réclamer de l'espace libre sur des LUN thin
---

Nous avons vu dans un [article précédent](http://blog.okcomputer.io/2015/02/26/R%C3%A9clamer-espace-libre-lun-thin-ESXi-UNMAP/) comment réclamer l'espace libre en ligne de commande. Cette méthode, bien que fonctionnelle, nécessite de lancer manuellement la commande pour chaque datastore concerné.

Afin d'automatiser le process, je vous propose d'utiliser PowerCLI:

{% gist 5262190e218b8ce69a58 %}

L'idée est de réclamer l'espace libre sur tous les datastores disponibles depuis le serveur ESXi passé en argument *-VMHost*. La taille du fichier asyncUnmapFileSize est définie en fonction de l'espace libre restant sur le datastore et un pourcentage qui par défaut est de 50% mais qui peut aussi être passé en argument *-asyncUnmapFilePourcentage*. Il est possible de préciser sur quel datastore on veut exécuter le script grâce à l'argument *-Datastore*.

Vous trouverez le script complet sur [GitHub](https://github.com/okcomputerpro/vmware-powercli/blob/master/UNMAP/ReclaimUnusedSpace.ps1).

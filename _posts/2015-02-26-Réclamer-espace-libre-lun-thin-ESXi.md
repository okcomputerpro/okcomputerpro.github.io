---
layout: post
title: Récupérer de l'espace libre sur des LUN thin avec VMware ESXi
---

VMware a introduit avec la version 5.0 une nouvelle primitive VAAI (UNMAP) qui permet, dans le cas de LUN en thin provisionning, de réaffecter les blocs qui ne sont plus utilisé à l'espace libre. Le but étant de conserver les bénéfices du thin provisionning dans le temps. 

En version 5.0, l'invocation de cette primitive était gérée automatiquement par les ESXi mais des problèmes de performance ont poussé VMware à la désactiver. Elle a été de nouveau disponible dans la version 5.0 U1 cependant l'exécution est maintenant manuelle. L'efficacité de la primitive a été améliorée dans la version 5.5.

Le fonctionnement de la primitive est plutôt simple, elle va écrire des 0 sur les blocs qui ne sont plus utilisé par le datastore. Ensuite ce sera à la charge de la baie de stockage de réaffecter ces blocs à l'espace libre.

###Ma baie est-elle compatible avec la primitive ?

Outre le fait que la baie doit être compatible VAAI (ce qui est le cas de toutes les baies SAN récentes) il faut en plus qu'elle supporte la primitive UNMAP. Il est possible de le vérifier pour chaque datastore avec les commandes esxcli suivantes:

```
esxcli storage core device list | more (noter le uid naa.)
esxcli storage core device vaai status get -d <naa ID> (Vérifier la présence de ligne Zero Status: Supported)
```

###Quand et pourquoi utiliser cette fonctionnalité ?

Et bien ça dépend... Sur des datastores avec des VM de production il est rare que la volumétrie soit amenée à beaucoup varier. Réclamer l'espace libre sur ce type d'environnement sera certainement une perte de temps. Là ou on pourra y voir un plus grand intérêt c'est dans le cas des environnements de test/dev ou plus généralement quand le contenu des datastores évolue régulièrement.

###VMware ESXi 5.0 U1 et 5.1:

Avec un ESXi en version 5.0 U1 ou 5.1 il faut utiliser la commande vmkfstools:

Se connecter en SSH sur l'ESXi et se positionner sur le datastore concerné

```
cd /vmfs/volumes/nomdudatastore
```

Exécuter la primitive

```
vmkfstools -y 60
```

####Remarques:

* La commande vmkfstools crée un fichier temporaire (.vmfsBalloon+suffix) dans lequel tous les blocks récupérés seront stockés avant d'êtres remis à zéro, le paramètre -y 60 indique que 60% des blocks libres seront remis à zéro, cela implique que le datastore devra disposer de suffisamment d'espace disque libre pour effectuer cette opération. En cas de doute, relancer la commande plusieurs fois avec un % inférieur.
* L'exécution de la primitive peut générer beaucoup d'I/O et de l'utilisation CPU, VMware recommande de réaliser ces opérations hors production.

###VMware ESXi 5.5

La commande vmkfstools n'est plus supportée depuis la version 5.5, elle a été remplacée par une commande esxcli:

Se connecter en SSH sur l'ESXi et exécuter la primitive

```
esxcli storage vmfs unmap -l nomdudatastore -n nombredeblocks
```

####Remarques:

* La commande crée un fichier temporaire (.asyncUnmapFile) mais, à la différence des versions précédentes, la taille du fichier est définie par la valeur du paramètre -n (en Mo)
* Une seule exécution de la commande est nécessaire pour libérer l'ensemble des blocks non utilisés
* HP recommande de spécifier une valeur importante pour le paramètre -n afin de réduire la durée d'exécution de la primitive
* VMware indique que la primitive peut être utilisée pendant les heures de production, il est quand même nécessaire de réaliser des tests au préalable.

###Sources:

####VMware ESXi 5.0 U1 et 5.1:

[KB VMware](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2014849)

[Jason Boche - Starting thin and staying thin with VAAI Unmap](http://www.boche.net/blog/index.php/2012/06/28/storage-starting-thin-and-staying-thin-with-vaai-unmap/)

####VMware ESXi 5.5:
[Jason Boche - vSphere 5.5 UNMAP Deep dive](http://www.boche.net/blog/index.php/2013/09/13/vsphere-5-5-unmap-deep-dive/)

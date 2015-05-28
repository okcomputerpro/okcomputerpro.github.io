---
layout: post
title: Configurer une baie EMC VNX en ligne de commande
---

Pour faire suite à [l'article](http://blog.okcomputer.io/2015/02/27/Initialisation-VNX-CLI/) expliquant comment initialiser une baie EMC VNX en CLI avec l'utilitaire Naviseccli, voici quelques commandes supplémentaires permettant d'aller jusqu'à la présentation des LUN aux hôtes.

Dans un premier temps je vous conseil d'enregistrer dans un fichier de sécurité les identifiants du compte Unisphere que vous utiliserez pour configurer la baie. Cela vous évitera de retaper à chaque commande les informations d'identification. Attention, par défaut, le fichier de sécurité est stocké dans votre profil utilisateur.

Créer un fichier de sécurité pour l'utilisateur sysadmin:

```
naviseccli -AddUserSecurity -Scope 0 -user sysadmin -password sysadmin
```

Une fois fait, vous pourrez executer vos commandes naviseccli sans fournir d'identifiants.

Passons maintenant à la configuration de la baie. Les différentes étapes seront les suivantes:
- Modification des noms des SP (facultatif...)
- Synchronisation du temps avec des serveurs NTP
- Configuration des serveurs DNS et du domaine
- Activation des statistiques (facultatif sauf si l'on souhaite utiliser EMC Monitoring & Reporting)
- Configuration du FastCache (si vous avez la licence...)
- Création d'un Storage Pool
- Création de LUN

> Les différentes options des commandes seront bien sur à modifier en fonction de la configuration de votre baie (emplacement des disques) et des IP des différents serveurs de votre infrastructure (NTP, DNS).

#### Modifier le nom des SP:

```
naviseccli -h 192.168.1.100 networkadmin -set -name vnx01spa
naviseccli -h 192.168.1.101 networkadmin -set -name vnx01spb
```  
> Attention cette commande implique un redémarrage des SP, il est préférable de modifier le nom d'un controleur à la fois.

#### Configuration NTP:

```
naviseccli -h 192.168.1.100 ntp -set -start yes -servers 192.168.1.3 192.168.1.4
```

#### Configuration DNS:

```
naviseccli -h 192.168.1.100 networkadmin -dns -set -domain domain.local -nameserver 192.168.1.1 192.168.1.2
```

#### Activation des Statistics Logging:

```
naviseccli -h 192.168.1.100 setstats -on
```

#### Configuration du FastCache:

La commande va activer le FastCache sur les disques indiqués via le paramètre `-disks` (ici 1_0_0 et 1_0_1) le mode du cache est défini par le paramètre `-mode` (ici en lecture/écriture) et le type de RAID est défini par le paramètre `-rtype` (ici RAID 1).

```
naviseccli -h 192.168.1.100 cache -fast -create -disks 1_0_0 1_0_1 -mode rw -rtype r_1
```

#### Création d'un storage pool:

La commande va créer un storage pool sur l'ensemble des disques indiqués via le paramètre `-disks`, le type de RAID est défini par le paramètre `-rtype` (ici RAID 5) et le nombre de disques contenu dans chaque grappe RAID est défini par le paramètre `-rdrivecount` (ici 5 ce qui équivaut à 4+1).

```
naviseccli -h 192.168.1.100 storagepool -create -disks 0_0_4 0_0_5 0_0_6 0_0_7 0_0_8 0_0_9 0_0_10 0_0_11 0_0_12 0_0_13 0_0_14 0_0_15 0_0_16 0_0_17 0_0_18 0_0_19 0_0_20 0_0_21 0_0_22 0_0_23 -rtype r_5 -rdrivecount 5 -name Pool01 -autoTiering scheduled -fastcache on
```

#### Création d'un LUN:

```
naviseccli -h 192.168.1.100 lun -create -type nonThin -poolName Pool01  -capacity 1024 -sq gb -name LUN01 -l 0
```

#### Création d'un Storage Group:

```
naviseccli -h 192.168.1.100 storagegroup -create -gname SG01
```

#### Ajout d'un host à un Storage Group:

```
naviseccli -h 192.168.1.100 storagegroup -connecthost -host esxi01.domain.local -gname SG01
```
> Les hosts doivent êtres vues (zoning configuré, Unisphere Host Agent installé et configuré) par la baie avant d'être associés à un storage group

#### Ajout d'un LUN à un Storage Group:

```
naviseccli -h 192.168.1.100 storagegroup -addhlu -gname SG01 -hlu 0 -alu 0
```

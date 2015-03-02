---
layout: post
title: Initialiser une baie EMC VNX en ligne de commande
---

Tous ceux qui ont déjà installé une baie EMC VNX savent que l'outil d'initialisation fournie par EMC (EMC Storage System Initialization) est plutôt capricieux. Réussir à lui faire découvrir la baie sans avoir recours à un switch dédié peut parfois s'avérer compliqué.

Pour contourner ce problème je préfère maintenant initialiser la baie en ligne de commande avec l'outil NaviSecCLI (téléchargeable sur [le site support d'EMC](https://support.emc.com)). Les 2 contrôleurs ont une IP par défaut qui peut être utilisée pour configurer la baie:

- SPA: 1.1.1.1/24
- SPB: 1.1.1.2/24
- Utilisateur: sysadmin
- Mot de passe: sysadmin

La marche à suivre est plutôt simple, il est d'abord nécessaire de mettre son poste de travail dans le même sous réseau que la baie (prendre par exemple l'IP 1.1.1.3) puis de suivre la procédure suivante:

Vérifier que la baie est bien joignable sur les 2 IP

```
naviseccli -h 1.1.1.1 -user sysadmin -password sysadmin -scope 0 networkadmin -get
naviseccli -h 1.1.1.2 -user sysadmin -password sysadmin -scope 0 networkadmin -get
```

Vérifier que la baie ne remonte aucune erreur sur les 2 contrôleurs

```
naviseccli -h 1.1.1.1 -user sysadmin -password sysadmin -scope 0 faults -list
naviseccli -h 1.1.1.2 -user sysadmin -password sysadmin -scope 0 faults -list
```

Vérifier qu'aucun domaine n'a déjà été configuré

```
naviseccli -h 1.1.1.1 -user sysadmin -password sysadmin -scope 0 domain -list
naviseccli -h 1.1.1.2 -user sysadmin -password sysadmin -scope 0 domain -list
```

Configurer l'adresse IP cible du SPA

```
naviseccli -h 1.1.1.1 -user sysadmin -password sysadmin -scope 0 networkadmin -set -ipv4 -address 192.168.1.1 -subnetmask 255.255.255.0  -gateway 192.168.1.254
```

Configurer l'adresse IP cible du SPB

```
naviseccli -h 1.1.1.2 -user sysadmin -password sysadmin -scope 0 networkadmin -set -ipv4 -address 192.168.1.2 -subnetmask 255.255.255.0  -gateway 192.168.1.254
```

Une fois toutes ces opérations réalisées avec succès, vous devriez être en mesure de vous connecter à la baie avec Unisphere. Il ne vous restera plus qu'à finaliser la configuration en spécifant par exemple les serveurs DNS et NTP.

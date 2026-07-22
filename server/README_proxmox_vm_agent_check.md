# Proxmox VM Zabbix agent check

Template complémentaire à **Proxmox VE by HTTP**. Elle lève une alerte d'information lorsqu'une VM QEMU existe dans Proxmox mais n'existe **pas** comme host dans Zabbix, autrement dit une VM déployée mais jamais ajoutée à la supervision (donc sans agent Zabbix).

## Principe

L'approche est autonome et découplée de la structure interne de la template officielle :

1. Une règle LLD de type HTTP agent interroge directement l'API Proxmox `/cluster/resources` pour lister les VMs QEMU.
2. Les entrées sont filtrées sur le type `qemu`, et les noms correspondant à la macro d'exclusion sont écartés (serveur et proxys Zabbix par exemple).
3. Pour chaque VM découverte, un item de type Script interroge l'API Zabbix (`host.get`) et vérifie si un host du même nom existe.
4. Un trigger d'information se déclenche lorsqu'aucun host correspondant n'existe.

La correspondance repose sur l'égalité exacte entre le nom de la VM dans Proxmox et le nom technique du host dans Zabbix.

## Prérequis

- La template doit être liée au **même host** que *Proxmox VE by HTTP*, dont elle réutilise les macros d'accès à l'API Proxmox (URL, port, token).
- Un **token API Zabbix en lecture seule** (utilisateur avec droits de lecture sur les hosts à vérifier).
- La macro globale `{$ZABBIX.URL}` doit être définie (URL de base du frontend). Le script y ajoute automatiquement `/api_jsonrpc.php`.

## Ce que détecte réellement la template

Le trigger vérifie l'**existence d'un host Zabbix** du même nom que la VM, pas l'état technique de l'agent. Concrètement :

- Si le host existe dans Zabbix, la VM est considérée comme supervisée, même si son agent est momentanément injoignable.
- Le trigger ne se déclenche que pour les VMs sans host correspondant.

Pour surveiller en plus la disponibilité effective de l'agent sur les hosts existants, utiliser l'item interne `zabbix[host,agent,available]` sur ces hosts (hors périmètre de cette template).

## Macros

| Macro | Valeur par défaut | Description |
|-------|-------------------|-------------|
| `{$PVE.URL.HOST}` | | Nom d'hôte ou IP de l'API Proxmox. Réutilisée depuis la template officielle. |
| `{$PVE.URL.PORT}` | `8006` | Port de l'API Proxmox. Réutilisée depuis la template officielle. |
| `{$PVE.TOKEN.ID}` | `USER@REALM!TOKENID` | ID du token API Proxmox. Réutilisée depuis la template officielle. |
| `{$PVE.TOKEN.SECRET}` | *(secret)* | Secret du token API Proxmox. Réutilisée depuis la template officielle. |
| `{$ZABBIX.URL}` | | URL de base du frontend Zabbix (macro globale existante). |
| `{$ZBX.API.TOKEN}` | *(secret)* | Token API Zabbix en lecture seule. |
| `{$ZBX.VM.IGNORE.MATCHES}` | `^$` | Regex des noms de VM/hosts à exclure du contrôle. `^$` n'exclut rien. |

### Exemple d'exclusion

Pour ignorer les VMs qui hébergent le serveur et les proxys Zabbix eux-mêmes :

```
^(zabbix-srv|zabbix-proxy01|zabbix-proxy02)$
```

Ajouter le préfixe `(?i)` pour rendre la correspondance insensible à la casse.

## Découverte

| Élément | Type | Détail |
|---------|------|--------|
| VM discovery (Zabbix agent check) | HTTP agent (LLD) | Interroge `/cluster/resources`, filtre `qemu`, applique l'exclusion. |
| VM [...]: Zabbix host present | Script (item prototype) | Renvoie `1` si un host Zabbix du même nom existe, `0` sinon. |
| VM [...] présente dans Proxmox mais absente de Zabbix | Trigger (Info) | Se déclenche sur la valeur `0`. Tag `class: vm_no_zabbix_host`. |

## Dashboard

Un dashboard « VMs sans host Zabbix » présente deux widgets Problems filtrés sur le tag `class: vm_no_zabbix_host` : les VMs actuellement en alerte, et celles récemment résolues. La liste s'adapte automatiquement aux VMs découvertes.

## Réseau et proxys

Le script s'exécute sur le composant qui supervise le host Proxmox (serveur ou proxy Zabbix). Ce composant doit pouvoir joindre l'URL contenue dans `{$ZABBIX.URL}`. Pour un Proxmox supervisé via un proxy sans accès à cette URL, surcharger `{$ZABBIX.URL}` au niveau du host concerné.

## Vérification après installation

1. Renseigner `{$ZBX.API.TOKEN}` et `{$ZBX.VM.IGNORE.MATCHES}` sur le host.
2. Forcer la découverte (*Discovery rules → Execute now*).
3. Dans *Monitoring → Latest data*, contrôler l'item « Zabbix host present » : une valeur `0` ou `1` par VM confirme le fonctionnement. Une erreur explicite indique quelle macro ou quel droit ajuster.

# PowerWalker UPS by SNMP

Template de supervision pour les onduleurs **PowerWalker** (BlueWalker) équipés d'une carte réseau (Network Management Card), exposant la MIB standard **RFC 1628 (UPS-MIB)** sur la branche `1.3.6.1.2.1.33`.

## Testé sur

- PowerWalker VFI 500 VA
- Network Management Card G2
- Firmware `2.2.0-r2`
- Zabbix 7.4

## Prérequis

- SNMP activé sur la carte réseau de l'onduleur (v2c ou v3).
- Une interface SNMP configurée sur l'hôte Zabbix, port 161.
- La macro `{$SNMP_COMMUNITY}` renseignée (au niveau hôte ou global) pour du SNMPv2c, ou les identifiants SNMPv3.

> **Recommandation** : privilégier SNMPv3 (auth + priv) en production. Le SNMPv2c transmet la communauté en clair sur le réseau.

## Contenu

### Items UPS (RFC 1628)

Identité (fabricant, modèle, firmware), état et charge batterie, autonomie restante, tension batterie, tension d'entrée, source d'alimentation de sortie, tension / courant / puissance / charge de sortie.

### Items SNMP standard (SNMPv2-MIB)

Nom système, emplacement, contact, description, object ID, uptime, et disponibilité de l'interface SNMP.

### Triggers

| Trigger | Sévérité |
|---------|----------|
| Pas de données SNMP depuis 10 minutes | High |
| La carte réseau a redémarré (uptime < 10 min) | Info |
| Fonctionne sur batterie (coupure secteur) | High |
| Batterie basse | High |
| Batterie épuisée | Disaster |
| Charge batterie faible | Warning |
| Autonomie restante faible | Warning |
| Charge de sortie élevée | Warning |
| Charge de sortie critique | High |

### Dashboard

Un dashboard « PowerWalker UPS » sur deux pages : une vue d'ensemble (états, jauges de charge, valeurs électriques, problèmes actifs) et une page de graphiques (tensions, charges, autonomie).

## Macros

| Macro | Valeur par défaut | Description |
|-------|-------------------|-------------|
| `{$UPS.BATTERY.CHARGE.WARN}` | `50` | Seuil bas de charge batterie restante (%). |
| `{$UPS.BATTERY.RUNTIME.WARN}` | `10` | Seuil bas d'autonomie restante (minutes). |
| `{$UPS.OUTPUT.LOAD.WARN}` | `80` | Seuil haut de charge de sortie (%), avertissement. |
| `{$UPS.OUTPUT.LOAD.HIGH}` | `95` | Seuil haut de charge de sortie (%), critique. |

## Notes sur les unités

Certaines valeurs sont remontées par la MIB en **dixièmes** d'unité. Un preprocessing avec facteur `0.1` est appliqué sur :

- **Tension batterie** : la MIB renvoie par exemple `823` pour 82,3 V.
- **Courant de sortie** : la MIB renvoie par exemple `12` pour 1,2 A.

Ces facteurs peuvent varier selon le firmware. Après import, vérifier dans *Latest data* que ces valeurs correspondent à l'affichage de l'onduleur, et ajuster ou retirer le preprocessing si un facteur 10 apparaît.

## Vérification SNMP préalable

Pour confirmer que la carte répond bien en RFC 1628 avant d'attacher le template :

```
snmpwalk -v2c -c <communaute> <IP_onduleur> 1.3.6.1.2.1.33
```

Si des valeurs remontent sous cette branche, le template est compatible. Sinon, la carte utilise probablement une MIB propriétaire et le template devra être adapté.

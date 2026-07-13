# Templates Zabbix custom

Ce dépôt regroupe les templates Zabbix développés et maintenus en interne. Chaque template est autonome (monolithique), au format YAML, et prêt à être importé dans Zabbix.

## Compatibilité

Sauf mention contraire dans la fiche du template, les templates de ce dépôt ciblent **Zabbix 7.4**. Les templates sont exportés avec `version: '7.4'`. Un template exporté depuis une version donnée s'importe dans la même version ou une version plus récente, mais pas dans une version antérieure.

## Catalogue

| Template | Type de collecte | Équipement cible | Fichier |
|----------|------------------|------------------|---------|
| PowerWalker UPS by SNMP | SNMP (RFC 1628) | Onduleurs PowerWalker avec carte réseau (NMC G2) | [`power/powerwalker_ups_snmp/`](power/powerwalker_ups_snmp/) |

## Structure du dépôt

```
.
├── README.md
├── power/
│   └── powerwalker_ups_snmp/
│       ├── template_powerwalker_ups_snmp.yaml
│       └── README.md
├── network/
├── server/
└── ...
```

L'arborescence est organisée par catégorie d'équipement (`power`, `network`, `server`, etc.). Chaque template vit dans son propre sous-dossier contenant le fichier YAML et, idéalement, une fiche `README.md` décrivant ses spécificités.

## Importer un template

1. Dans Zabbix, aller dans *Data collection → Templates*.
2. Cliquer sur *Import* en haut à droite.
3. Sélectionner le fichier `.yaml` du template.
4. Cocher au minimum *Templates*, *Value maps*, *Triggers* et *Dashboards* dans les règles d'import.
5. Pour une mise à jour d'un template existant, cocher aussi *Update existing*. Les entités sont comparées par UUID, celles qui correspondent sont mises à jour.
6. Cliquer sur *Import*.

## Attacher un template à un hôte

1. Créer ou éditer l'hôte avec l'adresse IP de l'équipement.
2. Ajouter l'interface adaptée au mode de collecte (interface **SNMP** pour les templates SNMP, port 161).
3. Renseigner les macros nécessaires (par exemple `{$SNMP_COMMUNITY}`, ou les identifiants SNMPv3). Cette macro peut être définie au niveau global de Zabbix, elle est alors héritée automatiquement.
4. Dans l'onglet *Templates* de l'hôte, lier le template voulu.

## Conventions

Quelques règles à respecter pour garder un dépôt cohérent :

- **UUID** : chaque entité (template, item, trigger, value map, dashboard) possède un UUID au format v4. Ne jamais modifier ou régénérer un UUID existant lors d'une mise à jour, sous peine de créer un doublon au lieu de mettre à jour l'entité. Un nouvel élément reçoit un nouvel UUID v4.
- **Nommage des items** : préfixer par le composant fonctionnel (par exemple `UPS:`, `SNMP:`) pour un regroupement lisible.
- **Seuils** : externaliser les seuils dans des macros (`{$...}`) plutôt que de les coder en dur dans les expressions de trigger, afin de permettre un ajustement au niveau de l'hôte.
- **Preprocessing** : documenter dans la description de l'item toute transformation appliquée (facteur multiplicateur pour les unités en dixièmes, par exemple).
- **Groupe de templates** : ranger les templates dans un groupe cohérent (`Templates/Power`, `Templates/Network`, etc.).

## Contribuer

1. Créer une branche à partir de `main`.
2. Ajouter le template dans le sous-dossier de catégorie adapté, avec sa fiche `README.md`.
3. Valider la syntaxe YAML et tester l'import sur une instance Zabbix avant de proposer la modification.
4. Mettre à jour le tableau du catalogue dans ce README.
5. Ouvrir une pull request.

## Licence

Ce projet est distribué sous licence **MIT**. Vous êtes libre de l'utiliser, le modifier et le redistribuer, y compris dans un cadre commercial, à condition de conserver la mention de copyright. Voir le fichier [`LICENSE`](LICENSE) pour le texte complet.

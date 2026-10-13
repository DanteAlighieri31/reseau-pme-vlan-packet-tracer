# 1. Cahier des charges

## Contexte

**Arvex** est une PME fictive de 40 salariés, installée sur un seul site de deux étages. Son réseau actuel est « à plat » : tous les postes, l'imprimante et le serveur partagent le même réseau et le même domaine de diffusion. Il n'y a ni cloisonnement, ni supervision, et les équipements s'administrent en Telnet avec un mot de passe partagé.

Un audit a relevé trois risques majeurs :

1. **N'importe quel poste peut joindre les données de la comptabilité** (paie, factures, coordonnées bancaires), ce qui pose un problème de confidentialité et de conformité au RGPD.
2. **Les équipements réseau sont administrables depuis n'importe quel poste**, en Telnet, donc avec des identifiants qui circulent en clair.
3. **Un seul domaine de diffusion** : une tempête de broadcast, une boucle ou un faux serveur DHCP branché par erreur peut paralyser toute l'entreprise.

## Besoins exprimés

| N° | Besoin | Priorité |
|----|--------|----------|
| B1 | Séparer les services dans des réseaux distincts (Direction, Comptabilité, Commercial, Informatique, Serveurs) | Haute |
| B2 | Permettre la communication entre services, sauf exceptions de sécurité | Haute |
| B3 | Interdire au service Commercial l'accès au réseau Comptabilité | Haute |
| B4 | Réserver l'administration des équipements au seul service Informatique, via un protocole chiffré | Haute |
| B5 | Attribuer automatiquement les adresses IP des postes | Haute |
| B6 | Donner accès à Internet à tous les services | Haute |
| B7 | Héberger un intranet et un DNS internes | Moyenne |
| B8 | Centraliser les journaux des équipements et synchroniser leur heure | Moyenne |
| B9 | Protéger le réseau contre les erreurs de branchement (boucles, faux DHCP, postes non autorisés) | Moyenne |
| B10 | Prévoir l'évolution : ajout de postes et de switches sans refonte | Moyenne |

## Contraintes

- Matériel Cisco (standard de l'entreprise) : routeur 2911, switches 2960 et 3560.
- La comptabilité est répartie sur les **deux étages** : son VLAN doit donc traverser les deux switches d'accès.
- Une seule adresse IP publique fournie par le FAI.
- Le projet est réalisé et validé dans **Cisco Packet Tracer** avant tout déploiement.

## Livrables

| Livrable | Emplacement |
|----------|-------------|
| Schéma d'architecture et choix techniques | [02-architecture.md](02-architecture.md) |
| Plan d'adressage IP et plan des VLANs | [03-plan-adressage.md](03-plan-adressage.md) |
| Guide de construction de la maquette | [04-guide-construction.md](04-guide-construction.md) |
| Politique de sécurité appliquée | [05-securite.md](05-securite.md) |
| Cahier de recette (tests de validation) | [06-recette.md](06-recette.md) |
| Guide de dépannage | [07-depannage.md](07-depannage.md) |
| Fiches techniques des équipements | [fiches-techniques/](fiches-techniques/) |
| Configurations complètes | [../configs/](../configs/) |

## Correspondance besoins / solutions

| Besoin | Solution retenue |
|--------|------------------|
| B1 | 5 VLANs métier + 1 VLAN d'administration |
| B2 | Routage inter-VLAN par un switch de niveau 3 (SVI) |
| B3 | ACL étendue en entrée du VLAN 30 |
| B4 | SSH v2, comptes locaux, `access-class` limitée au VLAN 50, VLAN 99 dédié et filtré |
| B5 | Serveur DHCP sur R1-EDGE + relais DHCP (`ip helper-address`) sur SW-CORE |
| B6 | NAT/PAT sur R1-EDGE, route par défaut diffusée en OSPF |
| B7 | Serveur SRV-INTRA (DNS + HTTP) dans le VLAN 40 |
| B8 | Syslog et NTP sur SRV-INTRA |
| B9 | Rapid PVST+ avec racine fixée, PortFast + BPDU Guard, DHCP Snooping, Port Security |
| B10 | Architecture cœur / accès, OSPF, trunks avec liste de VLANs explicite |

# 3. Plan d'adressage

## Principe

Tout le réseau interne utilise le bloc privé **10.10.0.0/16**. Le troisième octet correspond au numéro du VLAN : l'adresse `10.10.30.25` est immédiatement reconnaissable comme un poste du VLAN 30. Dans chaque réseau :

- `.1` = passerelle (SVI de SW-CORE) ;
- `.2` à `.20` = réservé aux équipements à adresse fixe (imprimantes, serveurs, switches) ;
- `.21` à `.254` = distribué par DHCP.

Les adresses « publiques » utilisent les plages réservées à la documentation (RFC 5737), pour ne jamais entrer en conflit avec un vrai réseau.

## Plan des VLANs

| VLAN | Nom | Réseau | Passerelle | Plage DHCP | Rôle |
|------|-----|--------|------------|------------|------|
| 10 | DIRECTION | 10.10.10.0/24 | 10.10.10.1 | .21 – .254 | Direction générale |
| 20 | COMPTA | 10.10.20.0/24 | 10.10.20.1 | .21 – .254 | Comptabilité (présente aux deux étages) |
| 30 | COMMERCIAL | 10.10.30.0/24 | 10.10.30.1 | .21 – .254 | Service commercial |
| 40 | SERVEURS | 10.10.40.0/24 | 10.10.40.1 | — (fixe) | Serveurs internes |
| 50 | INFORMATIQUE | 10.10.50.0/24 | 10.10.50.1 | .21 – .254 | Service informatique, seul autorisé à administrer |
| 99 | ADMIN | 10.10.99.0/24 | 10.10.99.1 | — (fixe) | Management des équipements réseau |
| 998 | PARKING | — | — | — | Ports inutilisés (désactivés) |
| 999 | NATIVE | — | — | — | VLAN natif des trunks, aucun port |

## Réseaux d'interconnexion

| Réseau | Lien | Adresses |
|--------|------|----------|
| 10.10.255.0/30 | R1-EDGE ↔ SW-CORE | R1-EDGE `.1` · SW-CORE `.2` |
| 203.0.113.0/30 | R1-EDGE ↔ ISP (WAN) | ISP `.1` · R1-EDGE `.2` (adresse publique de l'entreprise) |
| 198.51.100.0/24 | ISP ↔ Internet simulé | ISP `.1` · WEB-EXTERNE `.10` |

## Adresses fixes

| Équipement | Interface | Adresse | VLAN / réseau |
|------------|-----------|---------|---------------|
| R1-EDGE | Gi0/0 | 10.10.255.1/30 | Lien vers le cœur |
| R1-EDGE | Gi0/1 | 203.0.113.2/30 | WAN |
| SW-CORE | Gi0/1 (routé) | 10.10.255.2/30 | Lien vers R1 |
| SW-CORE | Vlan10 … Vlan50 | 10.10.x.1/24 | Passerelles |
| SW-CORE | Vlan99 | 10.10.99.1/24 | Management + passerelle ADMIN |
| SW-ACC1 | Vlan99 | 10.10.99.11/24 | Management |
| SW-ACC2 | Vlan99 | 10.10.99.12/24 | Management |
| SRV-INTRA | Fa0 | 10.10.40.10/24 | VLAN 40 |
| ISP | Gi0/0 | 203.0.113.1/30 | WAN |
| ISP | Gi0/1 | 198.51.100.1/24 | Internet simulé |
| WEB-EXTERNE | Fa0 | 198.51.100.10/24 | Internet simulé |

## Identifiants de routeur OSPF

| Équipement | Router ID | Aire |
|------------|-----------|------|
| R1-EDGE | 1.1.1.1 | 0 |
| SW-CORE | 2.2.2.2 | 0 |

## Brassage (affectation des ports)

### SW-CORE (Cisco 3560-24PS)

| Port | Mode | VLAN | Raccordé à |
|------|------|------|------------|
| Gi0/1 | Routé | — | R1-EDGE Gi0/0 |
| Fa0/1 | Trunk | 10, 20, 99 (natif 999) | SW-ACC1 Gi0/1 |
| Fa0/2 | Trunk | 20, 30, 50, 99 (natif 999) | SW-ACC2 Gi0/1 |
| Fa0/10 | Accès | 40 | SRV-INTRA |
| Fa0/3-9, Fa0/11-24, Gi0/2 | Désactivé | 998 | — |

### SW-ACC1 (Cisco 2960-24TT), étage 1

| Port | Mode | VLAN | Raccordé à |
|------|------|------|------------|
| Gi0/1 | Trunk | 10, 20, 99 (natif 999) | SW-CORE Fa0/1 |
| Fa0/1-4 | Accès | 10 DIRECTION | PC-DIR-01 sur Fa0/1 |
| Fa0/5-12 | Accès | 20 COMPTA | PC-COMPTA-01 sur Fa0/5 |
| Fa0/13-24, Gi0/2 | Désactivé | 998 | — |

### SW-ACC2 (Cisco 2960-24TT), étage 2

| Port | Mode | VLAN | Raccordé à |
|------|------|------|------------|
| Gi0/1 | Trunk | 20, 30, 50, 99 (natif 999) | SW-CORE Fa0/2 |
| Fa0/1-10 | Accès | 30 COMMERCIAL | PC-COM-01 sur Fa0/1, PC-COM-02 sur Fa0/2 |
| Fa0/11-14 | Accès | 50 INFORMATIQUE | PC-INFO-01 sur Fa0/11 |
| Fa0/15-18 | Accès | 20 COMPTA | PC-COMPTA-02 sur Fa0/15 |
| Fa0/19-24, Gi0/2 | Désactivé | 998 | — |

## Matrice des flux autorisés entre VLANs

Lecture : la ligne est la **source**, la colonne la **destination**.

| Source ↓ / Destination → | 10 DIR | 20 COMPTA | 30 COM | 40 SRV | 50 INFO | 99 ADMIN | Internet |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| **10 DIRECTION** | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ |
| **20 COMPTA** | ✅ | ✅ | ❌¹ | ✅ | ✅ | ❌ | ✅ |
| **30 COMMERCIAL** | ✅ | ❌ | ✅ | ✅ | ✅ | ❌ | ✅ |
| **50 INFORMATIQUE** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

¹ Les ACL sont *sans état* : la COMPTA peut envoyer une requête vers le COMMERCIAL, mais la réponse est bloquée par l'ACL du VLAN 30. L'isolement est donc effectif dans les deux sens. Voir [05-securite.md](05-securite.md).

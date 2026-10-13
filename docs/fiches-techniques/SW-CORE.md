# Fiche technique : SW-CORE

| Rubrique | Valeur |
|----------|--------|
| **Modèle** | Cisco Catalyst 3560-24PS (switch de niveau 3) |
| **Rôle** | Cœur de réseau : routage inter-VLAN, relais DHCP, filtrage, racine Spanning Tree |
| **Emplacement** | Baie principale, rez-de-chaussée |
| **Configuration** | [configs/SW-CORE.txt](../../configs/SW-CORE.txt) |
| **Administration** | SSH v2 sur `10.10.99.1` (ou toute SVI), depuis le VLAN 50 uniquement |

## Ports

| Port | Mode | VLAN(s) | Raccordement | Sécurité |
|------|------|---------|--------------|----------|
| Gi0/1 | Routé | — | R1-EDGE Gi0/0 (`10.10.255.2/30`) | — |
| Fa0/1 | Trunk 802.1Q | 10, 20, 99 · natif 999 | SW-ACC1 Gi0/1 | DTP désactivé |
| Fa0/2 | Trunk 802.1Q | 20, 30, 50, 99 · natif 999 | SW-ACC2 Gi0/1 | DTP désactivé |
| Fa0/10 | Accès | 40 | SRV-INTRA | Port Security (1 MAC), PortFast, BPDU Guard |
| Fa0/3-9, Fa0/11-24, Gi0/2 | Accès | 998 | — | Désactivés |

## Interfaces virtuelles (passerelles)

| SVI | Adresse | Relais DHCP | ACL en entrée | OSPF |
|-----|---------|-------------|---------------|------|
| Vlan10 | 10.10.10.1/24 | 10.10.255.1 | `ACL-DIRECTION-IN` | passif |
| Vlan20 | 10.10.20.1/24 | 10.10.255.1 | `ACL-COMPTA-IN` | passif |
| Vlan30 | 10.10.30.1/24 | 10.10.255.1 | `ACL-COMMERCIAL-IN` | passif |
| Vlan40 | 10.10.40.1/24 | — | — | passif |
| Vlan50 | 10.10.50.1/24 | 10.10.255.1 | `ACL-INFORMATIQUE-IN` | passif |
| Vlan99 | 10.10.99.1/24 | — | — | passif |
| Vlan1 | — | — | — | Désactivée |

## Fonctions

| Fonction | Paramètres |
|----------|------------|
| **VTP** | Mode transparent |
| **Spanning Tree** | Rapid PVST+, priorité 4096 sur les VLANs 10, 20, 30, 40, 50, 99 (pont racine) |
| **Routage** | `ip routing`, OSPF processus 1, router-id `2.2.2.2`, aire 0, route par défaut apprise de R1 |
| **Syslog / NTP** | Serveur `10.10.40.10` |

## Vérification

```
show vlan brief
show interfaces trunk
show spanning-tree root
show ip interface brief
show ip route
show ip ospf neighbor
show access-lists
show port-security
```

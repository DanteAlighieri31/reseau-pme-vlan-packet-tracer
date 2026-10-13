# Fiche technique : SW-ACC1

| Rubrique | Valeur |
|----------|--------|
| **Modèle** | Cisco Catalyst 2960-24TT (switch de niveau 2) |
| **Rôle** | Switch d'accès de l'étage 1 : Direction et Comptabilité |
| **Emplacement** | Local technique, étage 1 |
| **Configuration** | [configs/SW-ACC1.txt](../../configs/SW-ACC1.txt) |
| **Administration** | SSH v2 sur `10.10.99.11` (VLAN 99), passerelle `10.10.99.1`, depuis le VLAN 50 uniquement |

## Ports

| Port | Mode | VLAN(s) | Usage | Sécurité |
|------|------|---------|-------|----------|
| Gi0/1 | Trunk 802.1Q | 10, 20, 99 · natif 999 | Liaison montante vers SW-CORE Fa0/1 | DHCP Snooping *trust*, DTP désactivé |
| Fa0/1-4 | Accès | 10 DIRECTION | Postes de la direction (PC-DIR-01 sur Fa0/1) | Port Security, PortFast, BPDU Guard, limite DHCP |
| Fa0/5-12 | Accès | 20 COMPTA | Postes de la comptabilité (PC-COMPTA-01 sur Fa0/5) | Port Security, PortFast, BPDU Guard, limite DHCP |
| Fa0/13-24, Gi0/2 | Accès | 998 PARKING | Réserve | Désactivés |

## Fonctions et sécurité

| Fonction | Paramètres |
|----------|------------|
| **VTP** | Mode transparent (VLANs 10, 20, 30, 40, 50, 99, 998, 999 créés localement) |
| **Spanning Tree** | Rapid PVST+, priorité par défaut (n'est jamais racine) |
| **Port Security** | 2 MAC maximum par port, apprentissage *sticky*, violation *restrict* |
| **DHCP Snooping** | VLANs 10 et 20, option 82 désactivée, 15 paquets DHCP/s maximum par port d'accès |
| **Syslog / NTP** | Serveur `10.10.40.10` |

## Vérification

```
show vlan brief
show interfaces trunk
show interfaces status
show port-security
show ip dhcp snooping
show spanning-tree vlan 10
```

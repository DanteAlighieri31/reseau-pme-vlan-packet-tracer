# Fiche technique : SW-ACC2

| Rubrique | Valeur |
|----------|--------|
| **Modèle** | Cisco Catalyst 2960-24TT (switch de niveau 2) |
| **Rôle** | Switch d'accès de l'étage 2 : Commercial, Informatique et annexe Comptabilité |
| **Emplacement** | Local technique, étage 2 |
| **Configuration** | [configs/SW-ACC2.txt](../../configs/SW-ACC2.txt) |
| **Administration** | SSH v2 sur `10.10.99.12` (VLAN 99), passerelle `10.10.99.1`, depuis le VLAN 50 uniquement |

## Ports

| Port | Mode | VLAN(s) | Usage | Sécurité |
|------|------|---------|-------|----------|
| Gi0/1 | Trunk 802.1Q | 20, 30, 50, 99 · natif 999 | Liaison montante vers SW-CORE Fa0/2 | DHCP Snooping *trust*, DTP désactivé |
| Fa0/1-10 | Accès | 30 COMMERCIAL | PC-COM-01 sur Fa0/1, PC-COM-02 sur Fa0/2 | Port Security, PortFast, BPDU Guard, limite DHCP |
| Fa0/11-14 | Accès | 50 INFORMATIQUE | PC-INFO-01 sur Fa0/11 | Port Security, PortFast, BPDU Guard, limite DHCP |
| Fa0/15-18 | Accès | 20 COMPTA | PC-COMPTA-02 sur Fa0/15 | Port Security, PortFast, BPDU Guard, limite DHCP |
| Fa0/19-24, Gi0/2 | Accès | 998 PARKING | Réserve | Désactivés |

## Fonctions et sécurité

| Fonction | Paramètres |
|----------|------------|
| **VTP** | Mode transparent (VLANs 10, 20, 30, 40, 50, 99, 998, 999 créés localement) |
| **Spanning Tree** | Rapid PVST+, priorité par défaut (n'est jamais racine) |
| **Port Security** | 2 MAC maximum par port, apprentissage *sticky*, violation *restrict* |
| **DHCP Snooping** | VLANs 20, 30 et 50, option 82 désactivée, 15 paquets DHCP/s maximum par port d'accès |
| **Syslog / NTP** | Serveur `10.10.40.10` |

## Vérification

```
show vlan brief
show interfaces trunk
show interfaces status
show port-security
show ip dhcp snooping
show spanning-tree vlan 30
```

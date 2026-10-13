# Fiche technique : R1-EDGE

| Rubrique | Valeur |
|----------|--------|
| **Modèle** | Cisco ISR 2911 |
| **Rôle** | Routeur de bordure : sortie Internet, serveur DHCP |
| **Emplacement** | Baie principale, rez-de-chaussée |
| **Configuration** | [configs/R1-EDGE.txt](../../configs/R1-EDGE.txt) |
| **Administration** | SSH v2 sur `10.10.255.1`, depuis le VLAN 50 uniquement |

## Interfaces

| Interface | Description | Adresse | NAT | ACL | État |
|-----------|-------------|---------|-----|-----|------|
| Gi0/0 | Lien routé vers SW-CORE Gi0/1 | 10.10.255.1/30 | inside | — | Actif |
| Gi0/1 | WAN vers ISP Gi0/0 | 203.0.113.2/30 | outside | `ACL-WAN-IN` (entrée) | Actif, CDP désactivé |
| Gi0/2 | Non utilisée | — | — | — | Désactivée |

## Fonctions

| Fonction | Paramètres |
|----------|------------|
| **Routage statique** | `0.0.0.0/0` via `203.0.113.1` (FAI) |
| **OSPF** | Processus 1, router-id `1.1.1.1`, aire 0, réseau `10.10.255.0/30`, diffuse la route par défaut (`default-information originate`) |
| **NAT/PAT** | Source : ACL 1 (`10.10.0.0/16`), traduite en adresse de Gi0/1 (*overload*) |
| **Serveur DHCP** | 4 pools : VLAN10-DIRECTION, VLAN20-COMPTA, VLAN30-COMMERCIAL, VLAN50-INFORMATIQUE. Adresses `.1`–`.20` exclues. DNS distribué : `10.10.40.10` |
| **Syslog / NTP** | Serveur `10.10.40.10` |

## Sécurité

- `ACL-WAN-IN` : rejet des sources RFC 1918 et `127.0.0.0/8` venant d'Internet
- SSH v2 seul, clé RSA 2048 bits, `access-class 10` (VLAN 50), 3 essais, délai 60 s
- Compte local `admin` (privilège 15, mot de passe haché), `enable secret`
- `exec-timeout` 5 min (console) / 10 min (VTY), bannière légale

## Vérification

```
show ip interface brief
show ip route
show ip ospf neighbor
show ip nat translations
show ip dhcp binding
show access-lists
show ip ssh
```

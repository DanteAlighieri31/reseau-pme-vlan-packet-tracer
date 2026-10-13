# Fiche technique : serveurs

## SRV-INTRA

| Rubrique | Valeur |
|----------|--------|
| **Modèle** | Server-PT (Packet Tracer) |
| **Rôle** | Serveur interne : DNS, intranet, collecte des journaux, référence horaire |
| **Raccordement** | SW-CORE Fa0/10, VLAN 40 SERVEURS |
| **Adressage** | `10.10.40.10/24`, passerelle `10.10.40.1`, DNS `10.10.40.10` (fixe) |
| **Configuration** | [configs/serveurs-et-postes.md](../../configs/serveurs-et-postes.md) |

| Service | Port | État | Détail |
|---------|------|------|--------|
| DNS | UDP 53 | Actif | `intranet.arvex.lan` → 10.10.40.10 · `web.example.com` → 198.51.100.10 |
| HTTP / HTTPS | TCP 80 / 443 | Actif | Page « Intranet Arvex » |
| Syslog | UDP 514 | Actif | Reçoit les journaux de R1-EDGE, SW-CORE, SW-ACC1, SW-ACC2 |
| NTP | UDP 123 | Actif | Source d'heure de tous les équipements Cisco |
| DHCP, FTP, TFTP, EMAIL, AAA | — | Désactivés | Réduction de la surface d'attaque |

**Sécurité :** port du switch limité à une seule adresse MAC (Port Security), BPDU Guard.

## WEB-EXTERNE (hors périmètre de l'entreprise)

| Rubrique | Valeur |
|----------|--------|
| **Modèle** | Server-PT (Packet Tracer) |
| **Rôle** | Simule un site web sur Internet, pour tester le NAT |
| **Raccordement** | ISP Gi0/1 |
| **Adressage** | `198.51.100.10/24`, passerelle `198.51.100.1` |
| **Services** | HTTP uniquement |

## ISP (hors périmètre de l'entreprise)

| Rubrique | Valeur |
|----------|--------|
| **Modèle** | Cisco ISR 2911 |
| **Rôle** | Simule le routeur du fournisseur d'accès |
| **Interfaces** | Gi0/0 `203.0.113.1/30` (vers R1-EDGE) · Gi0/1 `198.51.100.1/24` (vers WEB-EXTERNE) |
| **Routage** | Aucune route vers le réseau interne : le NAT de R1-EDGE rend les adresses `10.10.x.x` invisibles |
| **Configuration** | [configs/ISP.txt](../../configs/ISP.txt) |

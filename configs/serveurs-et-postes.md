# Configuration des serveurs et des postes

Les serveurs et les PC de Packet Tracer se configurent avec l'interface graphique, pas en ligne de commande. Voici les réglages exacts à appliquer.

## SRV-INTRA (Server-PT) : serveur interne

**Onglet Desktop → IP Configuration → Static**

| Champ | Valeur |
|-------|--------|
| IPv4 Address | `10.10.40.10` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `10.10.40.1` |
| DNS Server | `10.10.40.10` |

**Onglet Services :**

| Service | État | Réglages |
|---------|------|----------|
| **DNS** | ON | Enregistrements de type **A Record** : `intranet.arvex.lan` → `10.10.40.10` et `web.example.com` → `198.51.100.10` |
| **HTTP** | ON | Remplacer le contenu de `index.html` par la page ci-dessous |
| **HTTPS** | ON | Laisser par défaut |
| **SYSLOG** | ON | Les journaux des 4 équipements Cisco arrivent ici |
| **NTP** | ON | Laisser par défaut (le serveur donne l'heure aux équipements) |
| **DHCP** | **OFF** | Le DHCP est assuré par R1-EDGE |
| FTP, TFTP, EMAIL, AAA, IoT | **OFF** | Services inutiles : on réduit la surface d'attaque |

Page `index.html` de l'intranet :

```html
<html><body>
<h1>Intranet Arvex</h1>
<p>Serveur SRV-INTRA - VLAN 40 SERVEURS - 10.10.40.10</p>
</body></html>
```

> Pour simplifier, SRV-INTRA résout aussi le nom du site externe (`web.example.com`). Dans la réalité, il transmettrait ces requêtes aux serveurs DNS du FAI.

## WEB-EXTERNE (Server-PT) : site Internet simulé

**Desktop → IP Configuration → Static**

| Champ | Valeur |
|-------|--------|
| IPv4 Address | `198.51.100.10` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `198.51.100.1` |

**Services :** HTTP **ON**, avec cette page `index.html` :

```html
<html><body>
<h1>Site Internet externe</h1>
<p>Si cette page s'affiche, le NAT de R1-EDGE fonctionne.</p>
</body></html>
```

Désactivez les autres services (DNS, DHCP, FTP, TFTP, EMAIL, AAA).

## Postes de travail (PC-PT)

Tous les postes sont en **DHCP** : **Desktop → IP Configuration → DHCP**. Ils doivent recevoir une adresse dans la plage de leur VLAN, à partir de `.21` (les adresses `.1` à `.20` sont réservées).

| Poste | Switch | Port | VLAN | Adresse attendue |
|-------|--------|------|------|------------------|
| PC-DIR-01 | SW-ACC1 | Fa0/1 | 10 DIRECTION | `10.10.10.21` à `.254` |
| PC-COMPTA-01 | SW-ACC1 | Fa0/5 | 20 COMPTA | `10.10.20.21` à `.254` |
| PC-COM-01 | SW-ACC2 | Fa0/1 | 30 COMMERCIAL | `10.10.30.21` à `.254` |
| PC-COM-02 | SW-ACC2 | Fa0/2 | 30 COMMERCIAL | `10.10.30.21` à `.254` |
| PC-INFO-01 | SW-ACC2 | Fa0/11 | 50 INFORMATIQUE | `10.10.50.21` à `.254` |
| PC-COMPTA-02 | SW-ACC2 | Fa0/15 | 20 COMPTA | `10.10.20.21` à `.254` |

Si un PC affiche « DHCP request failed », consultez [docs/07-depannage.md](../docs/07-depannage.md).

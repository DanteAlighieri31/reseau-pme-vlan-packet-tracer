# 6. Cahier de recette

Chaque test vérifie un besoin du [cahier des charges](01-cahier-des-charges.md). Les commandes `ping`, `ssh` et `ipconfig` se tapent sur un PC dans **Desktop → Command Prompt** ; les commandes `show` dans l'onglet **CLI** de l'équipement (après `enable`).

> 💡 Un premier `ping` vers une nouvelle destination peut perdre le premier paquet (le temps de la résolution ARP) : c'est normal. Relancez le test si besoin.

Remplissez la colonne **Résultat** avec ✅ ou ❌, puis faites des captures des tests marqués 📸 pour le dossier `docs/img/`.

## A. Commutation et VLANs

| N° | Test | Où | Commande / action | Résultat attendu | Résultat |
|----|------|----|-------------------|------------------|:--------:|
| T01 📸 | Les VLANs existent | SW-CORE, SW-ACC1, SW-ACC2 | `show vlan brief` | VLANs 10, 20, 30, 40, 50, 99, 998, 999 présents, ports affectés comme dans le plan | ✅ |
| T02 📸 | Les trunks sont actifs | SW-CORE | `show interfaces trunk` | Fa0/1 et Fa0/2 en mode `on`, natif 999, VLANs autorisés conformes | ✅ |
| T03 | SW-CORE est racine STP | SW-ACC1 | `show spanning-tree vlan 10` | `Root ID` = adresse de SW-CORE, priorité 4106 (4096 + 10) | ✅ |
| T04 | Un VLAN traverse deux switches | PC-COMPTA-01 | `ping <IP de PC-COMPTA-02>` | Réponses reçues (même VLAN 20 sur deux étages) | ✅ |
## B. Adressage et routage

| N° | Test | Où | Commande / action | Résultat attendu | Résultat |
|----|------|----|-------------------|------------------|:--------:|
| T05 📸 | DHCP fonctionne | Chaque PC | `ipconfig` | Adresse en `10.10.<VLAN>.21` ou plus, passerelle `10.10.<VLAN>.1`, DNS `10.10.40.10` | ✅ |
| T06 | Baux DHCP visibles | R1-EDGE | `show ip dhcp binding` | Une ligne par PC | ✅ |
| T07 📸 | Voisinage OSPF établi | R1-EDGE | `show ip ospf neighbor` | Voisin `2.2.2.2` en état `FULL` | ✅ |
| T08 | Route par défaut diffusée | SW-CORE | `show ip route` | Ligne `O*E2 0.0.0.0/0 via 10.10.255.1` | ✅ |
| T09 | R1 connaît les VLANs | R1-EDGE | `show ip route ospf` | Les réseaux 10.10.10.0 à 10.10.99.0 appris en `O` | ✅ |
| T10 | Routage inter-VLAN | PC-DIR-01 | `ping <IP de PC-COM-01>` | Réponses reçues | ✅ |
## C. Services

| N° | Test | Où | Commande / action | Résultat attendu | Résultat |
|----|------|----|-------------------|------------------|:--------:|
| T11 📸 | Intranet par son nom | PC-COM-01 | Desktop → Web Browser → `http://intranet.arvex.lan` | Page « Intranet Arvex » | ✅ |
| T12 📸 | Accès Internet (NAT) | PC-DIR-01 | Web Browser → `http://web.example.com` | Page « Site Internet externe » | ✅ |
| T13 | Traductions NAT | R1-EDGE | `show ip nat translations` (juste après T12) | Adresse interne `10.10.10.x` traduite en `203.0.113.2` | ✅ |
| T14 | Logs centralisés | SRV-INTRA | Services → SYSLOG | Des messages des équipements Cisco apparaissent (relancer `write memory` ou désactiver puis réactiver un port pour en générer) | ✅ |
| T15 | Heure synchronisée | SW-CORE | `show ntp status` | `Clock is synchronized` (peut demander quelques minutes de simulation) | ✅ |
## D. Sécurité : cloisonnement

| N° | Test | Où | Commande / action | Résultat attendu | Résultat |
|----|------|----|-------------------|------------------|:--------:|
| T16 📸 | DIRECTION → COMPTA autorisé | PC-DIR-01 | `ping <IP de PC-COMPTA-01>` | Réponses reçues | ✅ |
| T17 📸 | COMMERCIAL → COMPTA interdit | PC-COM-01 | `ping <IP de PC-COMPTA-01>` | `Destination host unreachable` ou `Request timed out` | ✅ |
| T18 | DIRECTION → ADMIN interdit | PC-DIR-01 | `ping 10.10.99.11` | Échec | ✅ |
| T19 | INFORMATIQUE → ADMIN autorisé | PC-INFO-01 | `ping 10.10.99.11` | Réponses reçues | ✅ |
| T20 | Compteurs des ACL | SW-CORE | `show access-lists` | Les lignes `deny` de `ACL-COMMERCIAL-IN` affichent des correspondances (*matches*) après T17 | ✅ |
## E. Sécurité : administration

| N° | Test | Où | Commande / action | Résultat attendu | Résultat |
|----|------|----|-------------------|------------------|:--------:|
| T21 📸 | SSH depuis l'informatique | PC-INFO-01 | `ssh -l admin 10.10.99.11` (mot de passe `Arvex-Admin-2026`) | Bannière puis invite `SW-ACC1#` | ✅ |
| T22 | SSH vers le routeur | PC-INFO-01 | `ssh -l admin 10.10.255.1` | Invite `R1-EDGE#` | ✅ |
| T23 📸 | SSH refusé depuis un autre service | PC-DIR-01 | `ssh -l admin 10.10.10.1` | Connexion refusée (`access-class`) | ✅ |
| T24 | Telnet refusé | PC-INFO-01 | `telnet 10.10.99.11` | Connexion refusée | ✅ |
## F. Sécurité : niveau 2

| N° | Test | Où | Commande / action | Résultat attendu | Résultat |
|----|------|----|-------------------|------------------|:--------:|
| T25 | Port Security active | SW-ACC1 | `show port-security interface fa0/1` | `Port Security: Enabled`, 1 adresse MAC apprise (sticky) | ✅ |
| T26 📸 | Violation détectée | SW-ACC1 | Sur Fa0/1 : `switchport port-security maximum 1`, puis débrancher PC-DIR-01 et brancher un nouveau PC sur Fa0/1 et lui faire envoyer un ping | `show port-security interface fa0/1` : compteur `Security Violation Count` > 0. Remettre `maximum 2` ensuite. | ✅ |
| T27 | Port inutilisé inactif | SW-ACC1 | Brancher un PC sur Fa0/20 | Le lien reste rouge (port `administratively down`) | ✅ |
| T28 | DHCP Snooping actif | SW-ACC2 | `show ip dhcp snooping` | Activé sur les VLANs 20, 30, 50 ; Gi0/1 `trusted` | ✅ |
| T29 📸 | Faux serveur DHCP bloqué | SW-ACC2 | Brancher un nouveau Server-PT sur Fa0/3 (VLAN 30), activer son service DHCP avec un pool `192.168.1.0`, puis sur PC-COM-02 : repasser en Static puis DHCP | PC-COM-02 reçoit toujours une adresse `10.10.30.x` du vrai serveur | ✅ |
| T30 📸 | Visualiser l'étiquette 802.1Q | Mode **Simulation** | Filtre ICMP, ping de PC-DIR-01 vers SRV-INTRA, cliquer sur le paquet sur le trunk | Couche 2 : en-tête `802.1Q` avec `VLAN: 10` | ✅ |
## Synthèse

| Catégorie | Tests | Réussis |
|-----------|:-----:|:-------:|
| A. Commutation et VLANs | 4 | 4/4 |
| B. Adressage et routage | 6 | 6/6 |
| C. Services | 5 | 5/5 |
| D. Cloisonnement | 5 | 5/5 |
| E. Administration | 4 | 4/4 |
| F. Niveau 2 | 6 | 6/6 |
| **Total** | **30** | **30/30** |

Date de la recette : 13/10/2026 · Version de Packet Tracer : 9.0.1.0858

## Résultats observés

Valeurs relevées lors de la recette du 13/10/2026 (Packet Tracer 9.0.1.0858).

| N° | Résultat obtenu |
|----|-----------------|
| T01 | VLANs 10, 20, 30, 40, 50, 99, 998, 999 présents sur les trois switches ; ports affectés conformément au plan de brassage |
| T02 | Fa0/1 et Fa0/2 : mode `on`, `802.1q`, `trunking`, natif 999 ; autorisés `10,20,99,999` et `20,30,50,99,999` |
| T03 | Root ID priorité 4106, adresse `000C.CF3D.312D` = SW-CORE (« This bridge is the root » sur SW-CORE) |
| T04 | PC-COMPTA-01 → PC-COMPTA-02 (10.10.20.22) : 4/4 réponses |
| T05 | PC-DIR-01 10.10.10.21 · PC-COMPTA-01 10.10.20.21 · PC-COM-01 10.10.30.21 · PC-COM-02 10.10.30.22 · PC-INFO-01 10.10.50.21 · PC-COMPTA-02 10.10.20.22 ; passerelle `.1`, DNS 10.10.40.10, serveur DHCP 10.10.255.1 |
| T06 | 6 baux, un par PC (adresses MAC conformes) |
| T07 | Voisin 2.2.2.2 en `FULL/BDR` sur Gi0/0 |
| T08 | `O*E2 0.0.0.0/0 [110/1] via 10.10.255.1` |
| T09 | 10.10.10.0, .20, .30, .40, .50 et .99 appris en `O` via 10.10.255.2 |
| T10 | PC-DIR-01 → PC-COM-01 : 3/4 réponses (premier paquet perdu pendant la résolution ARP) |
| T11 | `http://intranet.arvex.lan` depuis PC-COM-01 : page « Intranet Arvex » |
| T12 | `http://web.example.com` depuis PC-DIR-01 : page « Site Internet externe » |
| T13 | `tcp 203.0.113.2:1025 ← 10.10.10.21:1025 → 198.51.100.10:80` |
| T14 | Messages `%SYS-5-CONFIG_I` reçus de R1-EDGE (10.10.255.1) et SW-CORE (10.10.40.1) |
| T15 | `Clock is synchronized, stratum 2, reference is 10.10.40.10` (après quelques minutes de simulation ; R1-EDGE également synchronisé) |
| T16 | PC-DIR-01 → PC-COMPTA-01 : 3/4 réponses (premier paquet perdu pendant la résolution ARP) |
| T17 | `Reply from 10.10.30.1: Destination host unreachable` ×4 |
| T18 | `Reply from 10.10.10.1: Destination host unreachable` ×4 |
| T19 | PC-INFO-01 → 10.10.99.11 : 4/4 réponses |
| T20 | `ACL-COMMERCIAL-IN` : `deny ip any 10.10.20.0 0.0.0.255 (4 match(es))` ; `ACL-DIRECTION-IN` : `deny ip any 10.10.99.0 0.0.0.255 (4 match(es))` |
| T21 | Bannière puis invite `SW-ACC1#` |
| T22 | Bannière puis invite `R1-EDGE#` |
| T23 | `% Connection refused by remote host` |
| T24 | `Trying 10.10.99.11 ...Open` puis immédiatement `[Connection to 10.10.99.11 closed by foreign host]`, sans invite de connexion : Packet Tracer ouvre la session TCP mais la ferme aussitôt (`transport input ssh`) |
| T25 | `Port Security: Enabled`, `Secure-up`, maximum 2, 1 adresse sticky (`00D0.BCC4.E307`, PC-DIR-01), 0 violation |
| T26 | Avec `maximum 1` et un PC inconnu (`00D0.BAEB.31DA`) sur Fa0/1 : 100 % de perte, `Security Violation Count: 5`. `maximum 2` remis ensuite |
| T27 | PC branché sur Fa0/20 : `FastEthernet0/20 is administratively down, line protocol is down (disabled)` |
| T28 | Snooping actif sur `20,30,50`, Gi0/1 `trusted`, ports d'accès non fiables limités à 15 pps |
| T29 | Faux serveur (pool 192.168.1.0/24) sur Fa0/3 : son DHCP Offer est rejeté par SW-ACC2 à l'entrée de Fa0/3 (vérifié en mode Simulation) ; PC-COM-02 reste en 10.10.30.22, serveur DHCP 10.10.255.1 |
| T30 | Ping PC-DIR-01 → SRV-INTRA : trame non étiquetée entre le PC et SW-ACC1, puis en-tête `802.1Q` (TPID 0x8100) `VLAN 10` sur le trunk SW-ACC1 → SW-CORE |

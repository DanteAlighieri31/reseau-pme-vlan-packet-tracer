# 5. Politique de sécurité

La sécurité est appliquée en **défense en profondeur** : chaque niveau (port physique, VLAN, routage, administration) a ses propres protections, pour qu'une seule erreur de configuration ne suffise pas à compromettre le réseau.

## Niveau 2 : les switches d'accès

| Mesure | Commande | Menace traitée |
|--------|----------|----------------|
| **Port Security** | `switchport port-security maximum 2` + `mac-address sticky` + `violation restrict` | Branchement d'un équipement non autorisé ou d'un petit switch pirate. Le switch apprend les adresses MAC légitimes ; au-delà de 2, les trames sont rejetées et un compteur de violations est incrémenté. Le mode *restrict* garde le port actif pour le poste légitime (le mode *shutdown* couperait aussi l'utilisateur). |
| **DHCP Snooping** | `ip dhcp snooping` + `ip dhcp snooping trust` sur le trunk uniquement | Faux serveur DHCP (une box branchée par erreur, ou un attaquant qui se fait passer pour la passerelle afin d'intercepter le trafic). Seul le port montant est autorisé à envoyer des réponses DHCP. |
| **Limite de débit DHCP** | `ip dhcp snooping limit rate 15` | Épuisement du pool DHCP (*DHCP starvation*) par des milliers de fausses demandes. |
| **PortFast + BPDU Guard** | `spanning-tree portfast` + `spanning-tree bpduguard enable` | PortFast met le poste en service immédiatement. BPDU Guard coupe le port si un switch y est branché : impossible de créer une boucle ou de prendre le rôle de racine STP. |
| **Ports inutilisés désactivés** | `shutdown` + VLAN 998 PARKING | Une prise réseau libre dans une salle de réunion ne donne accès à rien. Même réactivé par erreur, le port tombe dans un VLAN sans passerelle. |
| **VLAN natif dédié** | `switchport trunk native vlan 999` | Attaque par double étiquetage 802.1Q (*VLAN hopping*). |
| **DTP désactivé** | `switchport mode access` / `switchport nonegotiate` | Un PC qui négocie un trunk pour accéder à tous les VLANs (*switch spoofing*). |
| **VLAN 1 désactivé** | `interface Vlan1` → `shutdown` | Le VLAN 1 est le VLAN par défaut, connu de tous les attaquants. |

## Niveau 3 : le cloisonnement entre VLANs

Les ACL étendues sont appliquées **en entrée** des SVI de SW-CORE, au plus près de la source : un paquet interdit est rejeté avant même d'être routé.

| ACL | Appliquée sur | Règles |
|-----|---------------|--------|
| `ACL-DIRECTION-IN` | Vlan10 | Refuse vers ADMIN ; anti-usurpation |
| `ACL-COMPTA-IN` | Vlan20 | Refuse vers ADMIN ; anti-usurpation |
| `ACL-COMMERCIAL-IN` | Vlan30 | Refuse vers ADMIN et vers COMPTA ; anti-usurpation |
| `ACL-INFORMATIQUE-IN` | Vlan50 | Accès complet ; anti-usurpation |

**Anti-usurpation :** chaque ACL n'accepte que les adresses sources de son propre VLAN (`permit ip 10.10.X.0 0.0.0.255 any` puis `deny ip any any`). Un poste qui modifie son adresse IP pour se faire passer pour un autre service est bloqué.

**Pourquoi `permit udp any any eq 67` en premier ?** Un poste qui démarre n'a pas encore d'adresse : sa demande DHCP part de `0.0.0.0`. Sans cette ligne, l'anti-usurpation bloquerait le DHCP.

**Limite connue : ACL sans état.** Une ACL standard examine chaque paquet isolément, sans savoir s'il s'agit d'une réponse. Conséquence : la COMPTA peut envoyer un ping au COMMERCIAL, mais la réponse est bloquée par `ACL-COMMERCIAL-IN`. L'isolement fonctionne donc dans les deux sens, ce qui est acceptable ici. Avec un vrai pare-feu à états (ou le *Zone-Based Firewall* de Cisco), on pourrait autoriser finement les réponses : c'est une piste d'amélioration.

## Niveau 3 : la bordure Internet

| Mesure | Rôle |
|--------|------|
| **NAT/PAT** | Les adresses internes ne sont jamais visibles depuis Internet. Aucune connexion ne peut être initiée de l'extérieur vers un poste interne. |
| **`ACL-WAN-IN`** | Rejette les paquets venant d'Internet avec une adresse source privée ou de boucle locale (RFC 1918) : ce sont forcément des paquets usurpés. |
| **`no cdp enable` côté WAN** | CDP divulgue le modèle, la version d'IOS et les adresses de l'équipement. On ne l'envoie pas au FAI. |
| **OSPF passif sur les SVI** | Aucun paquet OSPF n'est envoyé vers les postes : un poste ne peut pas injecter de fausses routes. |

## Administration des équipements

| Mesure | Détail |
|--------|--------|
| **SSH v2 uniquement** | `transport input ssh` : Telnet, qui transmet les mots de passe en clair, est refusé. Clé RSA de 2048 bits. |
| **Comptes nominatifs locaux** | `username admin privilege 15 secret …` + `login local`. Le mot de passe est stocké haché (`secret`). |
| **Filtrage des sources** | `access-class 10 in` : seules les adresses du VLAN 50 INFORMATIQUE peuvent ouvrir une session SSH, quelle que soit l'adresse de l'équipement visée. |
| **VLAN de management dédié** | Les switches d'accès ne sont joignables que sur le VLAN 99, lui-même interdit aux services non informatiques par les ACL. |
| **Limites de session** | `exec-timeout` (5 min en console, 10 min en SSH), `ip ssh time-out 60`, 3 tentatives d'authentification maximum. |
| **Bannière légale** | Avertissement affiché avant connexion, nécessaire pour pouvoir poursuivre un accès non autorisé. |
| **`service password-encryption`** | Masque les mots de passe restants dans l'affichage de la configuration. |

## Traçabilité

- **Syslog** : les quatre équipements Cisco envoient leurs journaux à SRV-INTRA (`logging host 10.10.40.10`) : connexions, changements d'état des ports, violations de sécurité.
- **NTP** : tous les équipements prennent l'heure sur SRV-INTRA. Sans heure commune, impossible de reconstituer la chronologie d'un incident à partir des journaux de plusieurs équipements.
- **Horodatage à la milliseconde** : `service timestamps log datetime msec`.

## Mots de passe de la maquette

Les mots de passe (`Arvex-Enable-2026`, `Arvex-Admin-2026`) sont volontairement publiés **parce qu'il s'agit d'une maquette pédagogique**. En production :
- les mots de passe ne sont **jamais** publiés dans un dépôt Git ;
- chaque administrateur a son propre compte ;
- l'authentification est centralisée sur un serveur RADIUS ou TACACS+ (AAA).

## Correspondance avec les référentiels

| Recommandation | Mise en œuvre |
|----------------|---------------|
| ANSSI — *Recommandations pour la sécurisation des réseaux* : cloisonner les réseaux selon la sensibilité | VLANs par service + ACL |
| ANSSI — administrer depuis un réseau dédié | VLAN 99 + `access-class` |
| CIS Cisco IOS Benchmark — désactiver Telnet, utiliser SSH v2 | `transport input ssh`, `ip ssh version 2` |
| CIS — journalisation distante et NTP | `logging host`, `ntp server` |
| Bonnes pratiques Cisco (*Layer 2 security*) | Port Security, DHCP Snooping, BPDU Guard, VLAN natif inutilisé |

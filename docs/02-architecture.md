# 2. Architecture

## Schéma logique

```mermaid
flowchart TB
    WEB["🌐 WEB-EXTERNE<br/>198.51.100.10"]
    ISP["ISP · FAI simulé<br/>Cisco 2911"]
    R1["R1-EDGE · Cisco 2911<br/>NAT/PAT · DHCP · OSPF · ACL WAN"]
    CORE["SW-CORE · Cisco 3560 (niveau 3)<br/>Routage inter-VLAN · Relais DHCP · ACL<br/>Racine STP · OSPF"]
    SRV["SRV-INTRA · VLAN 40<br/>DNS · Web · Syslog · NTP<br/>10.10.40.10"]
    ACC1["SW-ACC1 · Cisco 2960<br/>Étage 1"]
    ACC2["SW-ACC2 · Cisco 2960<br/>Étage 2"]

    WEB --- ISP
    ISP ---|"WAN 203.0.113.0/30"| R1
    R1 ---|"Lien routé 10.10.255.0/30<br/>OSPF aire 0"| CORE
    CORE ---|"Fa0/10 · accès VLAN 40"| SRV
    CORE ===|"Trunk 802.1Q<br/>VLANs 10,20,99"| ACC1
    CORE ===|"Trunk 802.1Q<br/>VLANs 20,30,50,99"| ACC2

    ACC1 --- DIR["PC-DIR-01<br/>VLAN 10"]
    ACC1 --- CPT1["PC-COMPTA-01<br/>VLAN 20"]
    ACC2 --- COM1["PC-COM-01<br/>VLAN 30"]
    ACC2 --- COM2["PC-COM-02<br/>VLAN 30"]
    ACC2 --- INFO["PC-INFO-01<br/>VLAN 50"]
    ACC2 --- CPT2["PC-COMPTA-02<br/>VLAN 20"]
```

## Modèle en deux niveaux (cœur réduit / accès)

| Niveau | Équipement | Rôle |
|--------|------------|------|
| **Bordure** | R1-EDGE | Seule porte vers Internet : NAT, filtrage des entrées, DHCP |
| **Cœur / distribution** | SW-CORE | Passerelle de tous les VLANs, routage inter-VLAN, filtrage interne, racine Spanning Tree |
| **Accès** | SW-ACC1, SW-ACC2 | Raccordement des postes, sécurité de niveau 2 au plus près de l'utilisateur |

Pour une PME d'un seul site, fusionner le cœur et la distribution dans un seul switch (*collapsed core*) est le compromis classique entre coût et performance. Ajouter un étage revient à brancher un nouveau switch d'accès en trunk sur SW-CORE.

## Choix techniques justifiés

### Routage inter-VLAN par switch de niveau 3 plutôt que « router-on-a-stick »

Avec un *router-on-a-stick*, tout le trafic entre VLANs remonte au routeur par un seul câble, qui devient un goulot d'étranglement. Le 3560 route en matériel à la vitesse de ses ports et garde le routeur pour son vrai rôle : la sortie Internet. Les passerelles sont les interfaces virtuelles (SVI) `Vlan10`, `Vlan20`…

### Lien routé (`no switchport`) entre SW-CORE et R1-EDGE

Le lien entre le cœur et le routeur est un réseau /30 dédié et non un trunk. Le routeur ne participe ainsi à aucun VLAN : un problème de niveau 2 (boucle, tempête de broadcast) ne peut pas l'atteindre.

### OSPF plutôt que des routes statiques

Avec seulement deux routeurs, des routes statiques suffiraient. OSPF a été choisi parce que :
- R1-EDGE apprend automatiquement tout nouveau VLAN créé sur SW-CORE (pas besoin de toucher au routeur) ;
- la route par défaut est diffusée au cœur avec `default-information originate` ;
- c'est la base d'une évolution vers un deuxième site ou un lien redondant.

Les SVI sont en `passive-interface` : leurs réseaux sont annoncés, mais aucun paquet OSPF n'est envoyé vers les postes, ce qui empêche un poste malveillant de s'annoncer comme routeur.

### DHCP centralisé sur R1-EDGE avec relais

Les pools DHCP sont sur le routeur, et SW-CORE relaie les demandes avec `ip helper-address`. On centralise ainsi la gestion des adresses sur un équipement, et ce mécanisme de relais est celui utilisé en entreprise avec un serveur DHCP Windows ou Linux.

### VTP en mode transparent

VTP en mode serveur/client peut effacer la base VLAN de tout le réseau si un switch avec un numéro de révision plus élevé est branché. En mode transparent, chaque switch gère ses propres VLANs : un peu plus de configuration, mais aucun risque d'effacement accidentel.

### Trunks : liste de VLANs explicite et VLAN natif dédié

- Chaque trunk ne transporte que les VLANs utiles de l'étage (*pruning* manuel) : moins de broadcast, et un VLAN absent d'un étage ne peut pas y être atteint.
- Le VLAN natif est le VLAN 999, qui n'est utilisé par aucun port : cela neutralise l'attaque par double étiquetage (*VLAN hopping*).
- `switchport nonegotiate` désactive DTP : un PC ne peut pas négocier un trunk.

### Spanning Tree : Rapid PVST+ et racine forcée

SW-CORE reçoit la priorité 4096 sur tous les VLANs pour être **toujours** le pont racine. Sans ça, l'élection se ferait sur la plus petite adresse MAC, et un vieux switch pourrait devenir racine et faire passer le trafic par un chemin absurde. Rapid PVST+ converge en quelques secondes au lieu de 30 à 50 secondes.

## Flux principaux

| Flux | Chemin |
|------|--------|
| PC-DIR-01 → Internet | ACC1 → trunk → SVI Vlan10 (ACL) → routage → R1-EDGE (NAT) → ISP → WEB-EXTERNE |
| PC-COMPTA-01 → PC-COMPTA-02 | Même VLAN : ACC1 → trunk → SW-CORE (commutation niveau 2) → trunk → ACC2, sans routage |
| PC-COM-01 → intranet | ACC2 → SVI Vlan30 (ACL) → routage → SVI Vlan40 → SRV-INTRA |
| Demande DHCP | Broadcast du PC → SVI (relais `ip helper-address`) → unicast vers R1-EDGE → réponse via OSPF |
| PC-INFO-01 → SSH SW-ACC1 | ACC2 → SVI Vlan50 → SVI Vlan99 → trunk → SW-ACC1 (`access-class 10`) |

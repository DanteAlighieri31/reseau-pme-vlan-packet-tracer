# 4. Guide de construction de la maquette

Ce guide permet de reconstruire la maquette de zéro dans **Cisco Packet Tracer 9.0 ou plus récent** (environ 1 h 30 la première fois).

## Étape 1 : placer les équipements

Dans la barre en bas de Packet Tracer, faites glisser ces équipements sur l'espace de travail :

| Catégorie Packet Tracer | Modèle | Quantité | Noms à donner |
|-------------------------|--------|----------|---------------|
| Network Devices → Routers | **2911** | 2 | R1-EDGE, ISP |
| Network Devices → Switches | **3560-24PS** | 1 | SW-CORE |
| Network Devices → Switches | **2960** (2960-24TT) | 2 | SW-ACC1, SW-ACC2 |
| End Devices → End Devices | **Server** | 2 | SRV-INTRA, WEB-EXTERNE |
| End Devices → End Devices | **PC** | 6 | PC-DIR-01, PC-COMPTA-01, PC-COM-01, PC-COM-02, PC-INFO-01, PC-COMPTA-02 |

Pour renommer un équipement : cliquez dessus → onglet **Config** → champ **Display Name**. (Le nom réel, `hostname`, sera défini par les configurations.)

> 💡 Organisez l'espace comme sur le [schéma d'architecture](02-architecture.md) : Internet en haut, les postes en bas. Ajoutez des zones de texte « Étage 1 » / « Étage 2 » et des rectangles de couleur par VLAN : c'est ce qui rendra vos captures d'écran lisibles.

## Étape 2 : câbler

Utilisez l'outil **Connections** (l'éclair), puis choisissez le type de câble et **cliquez sur le bon port** de chaque côté. Respecter les ports est indispensable, car les configurations y font référence.

| Équipement A | Port A | Équipement B | Port B | Câble |
|--------------|--------|--------------|--------|-------|
| R1-EDGE | GigabitEthernet0/0 | SW-CORE | GigabitEthernet0/1 | Droit (Copper Straight-Through) |
| R1-EDGE | GigabitEthernet0/1 | ISP | GigabitEthernet0/0 | Croisé (Copper Cross-Over) |
| ISP | GigabitEthernet0/1 | WEB-EXTERNE | FastEthernet0 | Droit |
| SW-CORE | FastEthernet0/1 | SW-ACC1 | GigabitEthernet0/1 | Croisé |
| SW-CORE | FastEthernet0/2 | SW-ACC2 | GigabitEthernet0/1 | Croisé |
| SW-CORE | FastEthernet0/10 | SRV-INTRA | FastEthernet0 | Droit |
| SW-ACC1 | FastEthernet0/1 | PC-DIR-01 | FastEthernet0 | Droit |
| SW-ACC1 | FastEthernet0/5 | PC-COMPTA-01 | FastEthernet0 | Droit |
| SW-ACC2 | FastEthernet0/1 | PC-COM-01 | FastEthernet0 | Droit |
| SW-ACC2 | FastEthernet0/2 | PC-COM-02 | FastEthernet0 | Droit |
| SW-ACC2 | FastEthernet0/11 | PC-INFO-01 | FastEthernet0 | Droit |
| SW-ACC2 | FastEthernet0/15 | PC-COMPTA-02 | FastEthernet0 | Droit |

Règle à retenir : câble **croisé** entre deux équipements de même type (switch-switch, routeur-routeur), câble **droit** sinon.

## Étape 3 : appliquer les configurations Cisco

**Ordre recommandé** : ISP → R1-EDGE → SW-CORE → SW-ACC1 → SW-ACC2. Le réseau se construit ainsi de l'extérieur vers l'intérieur, et chaque étape peut être testée.

Pour chaque équipement :

1. Cliquez dessus → onglet **CLI**.
2. S'il demande `Would you like to enter the initial configuration dialog? [yes/no]:`, tapez `no` puis Entrée.
3. Ouvrez le fichier correspondant dans [`configs/`](../configs/) et **copiez-collez un bloc à la fois** (d'un en-tête `! ----` au suivant). Coller tout le fichier d'un coup fonctionne souvent, mais Packet Tracer perd parfois des lignes quand le texte est long.
4. Vérifiez qu'aucune ligne n'affiche `% Invalid input` ou `% Incomplete command`. Si c'est le cas, voir [07-depannage.md](07-depannage.md).
5. La dernière ligne, `write memory`, sauvegarde la configuration dans la mémoire de démarrage.

> ⏱️ La génération de la clé RSA (`crypto key generate rsa`) peut prendre quelques secondes : attendez le message `[OK]` avant de coller la suite.

> ⚠️ Si la configuration de SW-CORE produit beaucoup de `% Invalid input` (sur `ip routing`, `no switchport` ou `switchport trunk encapsulation`), le switch placé n'est probablement pas un **3560** : le 2960 ne sait pas router. Vérifiez le modèle choisi à l'étape 1.

## Étape 4 : configurer les serveurs et les postes

Suivez [configs/serveurs-et-postes.md](../configs/serveurs-et-postes.md) : adresses fixes et services de SRV-INTRA et WEB-EXTERNE, puis passage de chaque PC en DHCP.

## Étape 5 : laisser converger

Après la configuration, attendez environ 30 secondes (ou cliquez plusieurs fois sur **Fast Forward Time** en bas à droite) : les ports doivent passer au vert, OSPF doit établir son voisinage, et les PC obtenir leur adresse.

## Étape 6 : valider

Déroulez le [cahier de recette](06-recette.md) et cochez chaque test. Faites des captures d'écran des tests importants pour le dossier `docs/img/`.

## Étape 7 : enregistrer

**File → Save As** → `topologie/arvex-reseau.pkt`. Ce fichier sera publié sur GitHub avec le reste du projet : n'importe qui pourra l'ouvrir et voir la maquette fonctionner.

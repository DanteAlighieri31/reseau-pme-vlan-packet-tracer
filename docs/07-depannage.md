# 7. Dépannage

La méthode : partir du bas (câble, port) et remonter les couches une par une. La plupart des pannes se trouvent dans les couches 1 et 2.

## Commandes de diagnostic essentielles

| Commande | Ce qu'elle montre |
|----------|-------------------|
| `show ip interface brief` | État et adresse de chaque interface (`up/up` attendu) |
| `show interfaces status` | (switch) État, VLAN et vitesse de chaque port |
| `show vlan brief` | VLANs et ports associés |
| `show interfaces trunk` | Trunks actifs, VLAN natif, VLANs autorisés |
| `show spanning-tree vlan 10` | Racine STP et état des ports |
| `show ip route` | Table de routage |
| `show ip ospf neighbor` | Voisins OSPF |
| `show ip dhcp binding` / `show ip dhcp pool` | (R1) Adresses distribuées |
| `show ip dhcp snooping` | (switch d'accès) Ports de confiance |
| `show access-lists` | Règles des ACL et nombre de correspondances |
| `show port-security` | Ports protégés et violations |
| `show ip nat translations` | (R1) Traductions en cours |
| `show running-config` | Configuration active complète |

## Un PC n'obtient pas d'adresse (« DHCP request failed »)

Vérifiez dans l'ordre :

1. **Le lien est vert** entre le PC et le switch. S'il est orange, attendez quelques secondes ou utilisez *Fast Forward Time*.
2. **Le port est dans le bon VLAN** : `show vlan brief` sur le switch d'accès.
3. **Le VLAN passe dans le trunk** : `show interfaces trunk` sur le switch d'accès **et** sur SW-CORE. Le VLAN doit apparaître dans la ligne *Vlans in spanning tree forwarding state*.
4. **La SVI est active** : `show ip interface brief` sur SW-CORE, `Vlan20` doit être `up/up`. Une SVI reste `down` tant qu'aucun port actif n'appartient au VLAN.
5. **Le relais est configuré** : `show running-config interface vlan 20` doit contenir `ip helper-address 10.10.255.1`.
6. **R1 peut répondre** : `show ip route` sur R1-EDGE doit contenir le réseau du VLAN appris par OSPF. Sinon, voir la section OSPF.
7. **DHCP Snooping** : si tout le reste est correct, désactivez temporairement l'option 82 qui pose souvent problème : `no ip dhcp snooping information option` (déjà présent dans les configurations), puis en dernier recours `no ip dhcp snooping` pour confirmer que c'est la cause.
8. Sur le PC, repassez en **Static** puis en **DHCP** pour relancer une demande.

## Le voisinage OSPF ne s'établit pas

- `show ip interface brief` : Gi0/0 de R1 et Gi0/1 de SW-CORE doivent être `up/up`.
- `ping 10.10.255.2` depuis R1 doit répondre.
- Sur SW-CORE, Gi0/1 doit être un port routé : `show running-config interface gi0/1` doit afficher `no switchport`.
- Vérifiez que `ip routing` est bien présent sur SW-CORE.
- Les deux routeurs doivent annoncer `10.10.255.0 0.0.0.3` dans l'**aire 0**.

## Pas d'accès à Internet

1. Depuis R1-EDGE : `ping 198.51.100.10` doit répondre (sinon, problème entre R1 et ISP).
2. Sur SW-CORE : `show ip route` doit afficher la route par défaut `O*E2`. Sinon, vérifiez `default-information originate` et la route statique sur R1.
3. Sur R1 : `ip nat inside` sur Gi0/0 et `ip nat outside` sur Gi0/1 ; `show ip nat translations` pendant un ping.
4. Pour le nom `web.example.com` : testez d'abord `ping 198.51.100.10` depuis le PC. Si l'IP répond mais pas le nom, c'est le DNS de SRV-INTRA (enregistrement A manquant ou service DNS sur OFF).

## Un ping entre VLANs échoue alors qu'il devrait passer

- `show access-lists` sur SW-CORE : quelle ligne voit son compteur augmenter ? Une ligne `deny ip any any` qui augmente signifie souvent que le PC a une **adresse fixe hors de son VLAN** (bloquée par l'anti-usurpation).
- Rappel : la COMPTA ne peut pas recevoir de réponse du COMMERCIAL (ACL sans état), c'est le comportement attendu.

## La connexion SSH échoue

| Symptôme | Cause probable |
|----------|----------------|
| `Connection refused` depuis le VLAN 50 | Clé RSA non générée : `show ip ssh` doit afficher `SSH Enabled - version 2.0`. Relancez `crypto key generate rsa general-keys modulus 2048`. |
| `Connection refused` depuis un autre VLAN | Normal : `access-class 10` (test T23). |
| `Request timed out` vers 10.10.99.11 | Vérifiez `ip default-gateway 10.10.99.1` sur le switch d'accès et que le VLAN 99 passe dans le trunk. |
| Mot de passe refusé | Le nom d'utilisateur est `admin`, le mot de passe `Arvex-Admin-2026` (sensible à la casse). |

## Un port est coupé ou rejette le trafic

- `show port-security interface fa0/X` : si `Security Violation Count` augmente, une adresse MAC non autorisée est branchée. Pour réinitialiser l'apprentissage : `clear port-security sticky interface fa0/X`.
- Port en `err-disabled` : BPDU Guard a détecté un switch. Débranchez-le, puis `shutdown` et `no shutdown` sur le port.

## Une commande est refusée (`% Invalid input detected`)

Packet Tracer ne gère pas toutes les commandes de l'IOS réel, et cela varie selon les versions.

| Commande | Alternative si elle est refusée |
|----------|-------------------------------|
| `crypto key generate rsa general-keys modulus 2048` | Taper `crypto key generate rsa` puis répondre `2048` (ou `1024`) à la question |
| `ip dhcp snooping limit rate 15` | Peut être ignorée, le reste du DHCP Snooping fonctionne |
| `no ip dhcp snooping information option` | Si absente, mettre `ip dhcp relay information trust-all` sur SW-CORE |
| `remark …` dans une ACL | Peut être ignorée : ce n'est qu'un commentaire |
| `ntp server` sur un 2960 | Peut être ignorée sur les switches d'accès |
| `interface range … , …` avec virgules | Configurer chaque plage séparément |

Notez toute commande refusée et sa solution dans une section « Écarts » du README : c'est exactement le type de retour d'expérience qui intéresse un recruteur.

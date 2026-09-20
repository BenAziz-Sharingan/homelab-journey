######### Résultats des tests de validation — Lab 01-vlan-Segmentation #########

#########  1. Connectivité de base (avant ACL) #############

| Test | Résultat |
|---|---|
| Ping intra-VLAN (PC1 ↔ PC2, Direction) | ✅ OK |
| Ping PC1: 172.16.10.5 (Direction) → 172.16.20.5 (RH-Finance) | ✅ OK |
| Ping PC1 (Direction) → 172.16.99.10 (Management) | ✅ OK |
| Ping PC1 (Direction) → 172.16.30.10 (IT-Dev) | ✅ OK |
| `show ip route` sur R1 | ✅ Les 6 réseaux VLAN apparaissent, routes confirmées indirectement par les pings pour VLAN 99 (coupure d'affichage terminal) |

Capture Wireshark sur le port d'accès e0/1 (VLAN 10) : confirme que le trafic hôte (ARP,
ICMP) sort bien non tagué (comportement normal d'un port access), et que les BPDU
Spanning-Tree PVST+ sont tagués VLAN 10 (comportement normal de Cisco PVST+, pas une
anomalie).


############  2. Validation de la matrice de flux (après ACL) #################

| Test | Source          | Destination           | Résultat attendu | Résultat observé |
|---   |---|--- |---|--- |                       |
| 1    | Direction (PC1) | Invités (172.16.66.5) | Bloqué           | ✅ `ICMP type:3, code:13 (Communication administratively prohibited)` |
| 2    | RH-Finance      | Invités               | Bloqué           | ✅ Timeout confirmé après ajout de la règle manquante dans ACL-RHFINANCE-IN |

Les autres combinaisons de la matrice (Direction↔Prod-Serveurs, Management↔IT-Dev,
RH-Finance↔Prod-Serveurs, etc.) restent à documenter avec captures individuelles au fur et
à mesure des tests — utiliser le tableau de la matrice de flux (`docs/matrice-de-flux.md`)
comme checklist.

###########  3. Vérification de la configuration des ACLs ########################

**** Verification avec la commande : << show ip interface e0/0.20 | include access list >>  → confirmé :
```
Inbound access list is ACL-RHFINANCE-IN
```

**** Verification avec la commande : `` show access-lists`` sur R1 confirme la présence des 6 ACLs nommées avec leurs règles dans
le bon ordre (règles spécifiques `permit` avant les `deny` génériques, `permit ip ... any`
en fin de liste).

###########   4. Anomalie détectée et corrigée   #############################

Un ping réussi entre le VLAN Invités et RH-Finance a révélé l'absence d'une règle `deny`
vers 172.16.66.0/24 dans `ACL-RHFINANCE-IN`. Corrigé par l'ajout de la ligne :
```
25 deny ip 172.16.20.0 0.0.0.255 172.16.66.0 0.0.0.255
```
Retest : timeout confirmé après correction.

###########  5. Limite connue  #########################################

Le flux TCP/443 (RH-Finance → IT-Dev) n'a pas pu être testé avec un vrai client TCP, VPCS
ne générant que de l'ICMP/ARP/DHCP. Validation indirecte via les compteurs
`show access-lists` sur la ligne `permit tcp ... eq 443`.

## À compléter

- [ ] Captures d'écran de chaque test du tableau matrice de flux, à ajouter dans ce dossier
- [ ] Capture `show access-lists` complète avec compteurs après une session de test complète
- [ ] (Optionnel) Test réel du port 443 avec une VM Linux + netcat

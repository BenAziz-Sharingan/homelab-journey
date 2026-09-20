# Matrice de flux — Lab 01

## Tableau récapitulatif

| Source \ Destination  | Direction | RH-Finance | IT-Dev    | Prod-Serveurs | Invités | Management |
|-----------------------|:---:|:---:|:---:|:---: |:---:|:---:|
| Direction             |  —        |  ✅        |  ✅       |  ✅           |  ❌     |  ✅        |
| RH-Finance            |  ✅       |  —         | ⚠️ TCP/443|  ✅           |  ❌     |  ❌        |
| IT-Dev                |  ✅       | ⚠️ TCP/443 |  —        |  ✅           |  ❌     |  ✅        |
| Prod-Serveurs         |  ✅       |  ✅        |  ✅       |  —            |  ❌     |  ❌        |
| Invités               |  ❌       |  ❌        |  ❌       |  ❌  	     |  —      |  ❌         |
| Management            |  ✅       |  ❌        |  ✅       |  ❌           |  ❌     |  —         |

Légende : ✅ autorisé · ❌ bloqué · ⚠️ autorisé uniquement sur le port applicatif TCP/443

############### Justification de chaque règle #############################

*************** Direction → tout trafic autoriser , sauf Invités ******************

Profil de confiance élevé, a besoin d'accéder à tous les pôles de l'entreprise pour ses
fonctions de pilotage. Seule exception : le VLAN Invités, pour éviter qu'un poste visiteur
compromis serve de rebond vers la Direction.

************* RH-Finance ↔ IT-Dev : TCP/443 uniquement ********************

RH-Finance a besoin d'accéder à une application de paie hébergée côté IT-Dev, mais rien
d'autre. Principe de moindre privilège : un accès applicatif précis remplace un accès
réseau complet.

*********** RH-Finance → Direction, Prod-Serveurs : Trafic autorisé ************************

Flux métier légitimes (reporting financier, accès aux données de production pour la
comptabilité).

*********** RH-Finance → Management, Invités : Trafic bloqué *******************************

Aucun besoin métier identifié ; réduit la surface d'attaque du VLAN Management et isole
totalement les invités.

*********** IT-Dev → Direction, Prod-Serveurs, Management :Trafic autorisé ******************

IT-Dev administre l'infrastructure (serveurs, équipements réseau), a donc besoin d'un accès
large à ces VLANs.

********** IT-Dev → Invités : Trafic bloqué ******************************************************

Aucune raison pour l'équipe technique d'interagir directement avec le réseau visiteur.

********** Prod-Serveurs → Direction, RH-Finance, IT-Dev : Trafic autorisé ************************

Les serveurs doivent répondre aux applications utilisées par ces trois pôles.

********** Prod-Serveurs → Management, Invités : Trafic bloqué ************************************

Le Management n'a pas besoin d'un accès direct aux serveurs (l'administration se fait par
IT-Dev), et les invités ne doivent jamais atteindre la production.

*********** Invités → tout : bloqué ****************************************************************

Isolation totale. Le réseau visiteur est une zone non fiable par définition, sans aucun
accès au réseau interne.

*********** Management → Direction, IT-Dev : autorisé *********************************************

Les deux profils légitimes pour effectuer de l'administration réseau (accès SSH/Telnet aux
équipements, supervision).

*********  Management → RH-Finance, Prod-Serveurs, Invités : Trafic bloqué *************************

Une compromission du VLAN Management ne doit pas donner un accès direct aux données
métier ni aux serveurs de production.

###################  Implémentation technique ###############################################

Chaque règle est appliquée via une ACL extended nommée, en entrée (`in`) sur la
sous-interface R1 correspondante :

| VLAN | ACL appliquée       | Interface   |
|------|---------------------|-------------|
| 10   | ACL-DIRECTION-IN    | Et0/0.10    |
| 20   | ACL-RHFINANCE-IN    | Et0/0.20    |
| 30   | ACL-ITDEV-IN        | Et0/0.30    |
| 40   | ACL-PRODSERV-IN     | Et0/0.40    |
| 66   | ACL-INVITED-IN      | Et0/0.66    |
| 99   | ACL-MGMT-IN         | Et0/0.99    |

Le contenu complet de chaque ACL est visible dans `configs/routeur-r1.cfg` et dans les
captures `show access-lists` du dossier `tests/`.

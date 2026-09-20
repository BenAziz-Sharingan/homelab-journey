# Plan d'adressage — Lab 01

## Vue d'ensemble

Bloc d'adressage global : `172.16.0.0/16`, découpé en sous-réseaux `/24` par VLAN.

| VLAN | Nom                 | Réseau              | Passerelle (sous-interface R1) | Plage hôtes utilisable |
|------|---------------------|---------------------|----------------------------------|-------------------------|
| 10   | DIRECTION           | 172.16.10.0/24      | 172.16.10.1 (Et0/0.10)           | 172.16.10.2 – .254      |
| 20   | RH-FINANCE          | 172.16.20.0/24      | 172.16.20.1 (Et0/0.20)           | 172.16.20.2 – .254      |
| 30   | IT-DEV              | 172.16.30.0/24      | 172.16.30.1 (Et0/0.30)           | 172.16.30.2 – .254      |
| 40   | PRODUCTION-SERVEURS | 172.16.40.0/24      | 172.16.40.1 (Et0/0.40)           | 172.16.40.2 – .254      |
| 66   | INVITES             | 172.16.66.0/24      | 172.16.66.1 (Et0/0.66)           | 172.16.66.2 – .254      |
| 99   | MANAGEMENT          | 172.16.99.0/24      | 172.16.99.1 (Et0/0.99)           | 172.16.99.2 – .254      |

## Adressage des hôtes de test (VPCS)

| Hôte | VLAN | Adresse IP       | Passerelle    |
|------|------|------------------|---------------|
| PC1  | 10   | 172.16.10.5/24   | 172.16.10.1   |
| PC2  | 10   | 172.16.10.10/24  | 172.16.10.1   |
| PC3  | 20   | 172.16.20.5/24   | 172.16.20.1   |
| PC4  | 20   | 172.16.20.10/24  | 172.16.20.1   |
| PC5  | 40   | 172.16.40.5/24   | 172.16.40.1   |
| PC6  | 30   | 172.16.30.5/24   | 172.16.30.1   |
| PC7  | 66   | 172.16.66.5/24   | 172.16.66.1   |
| PC8  | 99   | 172.16.99.5/24   | 172.16.99.1   |

## Affectation des ports sur SW1

| Port SW1 | VLAN | Rôle                     |
|----------|------|--------------------------|
| Et0/0    | trunk (10,20,30,40,66,99) | Vers R1 (router-on-a-stick) |
| Et0/1    | 10   | DIRECTION (PC1)          |
| Et0/2    | 10   | DIRECTION (PC2)          |
| Et0/3    | 20   | RH-FINANCE (PC3)         |
| Et1/0    | 20   | RH-FINANCE (PC4)         |
| Et1/1    | 30   | IT-DEV (PC6)             |
| Et1/2    | 40   | PRODUCTION-SERVEURS (PC5)|
| Et1/3    | 66   | INVITES (PC7)            |
| Et2/0    | 99   | MANAGEMENT (PC8)         |

## Choix de conception

- Bloc `/16` volontairement large pour laisser de la place à une croissance future de
  NovaTech sans avoir à replanifier tout l'adressage.
- Chaque VLAN reste un /24 classique : simple à retenir, suffisant pour largement plus
  de 60 employés par pôle.
- L'ID de VLAN reprend le dernier octet du réseau (VLAN 10 → .10.0/24) pour faciliter la
  lecture et le dépannage.

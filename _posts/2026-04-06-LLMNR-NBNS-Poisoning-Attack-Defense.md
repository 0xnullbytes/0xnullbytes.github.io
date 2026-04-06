---
layout: post
title: LLMNR & NBT-NS Poisoning - Un Guide Pratique
date: 2026-04-06
image: /assets/img/posts/responder.png
categories:
  - Active Directory
tags:
  - mitm
  - spoofing
  - windows
  - french
---
**LLMNR/NBT-NS poisoning**  est un type d'attaque que l'on retrouve très souvent dans un envrionnement Active Directory. Il exploite l'hiérarchie de résolution de nom afin de voler les informations d'authentification d'un compte du domaine. 

Cette information se présente sous forme de **Hash NTLMv2 ou NTLMv1** pouvant être brute-forcé en local à l'aide d'outils tels que **Hashcat** ou **John The Ripper** ou relayé vers d'autres services ou protocoles.

## Hiérarchie de résolution de nom

Au sein d'un **AD**, le **DNS** (Domain Name System) joue un rôle crucial. Parmi ces rôles, on peut citer:
- **Résolution de noms** : transforme les noms de machines en adresses IP.
- **Localisation des services AD** : via les enregistrements SRV pour trouver les contrôleurs de domaine, LDAP, Kerberos, etc.
- ...

Si le **DNS** ne parvient pas à résoudre un nom d'hôte (par exemple en cas de faute de frappe ou de nom inexistant), alors le protocole **LLMNR (Link Local Mulitcast Name Resolution)** prend le relais. 

Et si ce dernier échoue, un autre protocole plus ancien, **NBT-NS (NetBIOS Name Service)** essaie à son tour de résoudre le nom d'hôte.

```
DNS → LLMNR → NetBIOS Name Service (NBT-NS)
```

## Présentation de LLMNR (Link Local Multicast Name Resolution)

## Présentation de NBT-NS (NetBIOS Name Service)

## Fonctionnement de l’attaque

## Capture des hashes NTLM

## Exploitation des hashes

## Démonstration pratique avec Responder

## Contre-mesures

## Conclusion





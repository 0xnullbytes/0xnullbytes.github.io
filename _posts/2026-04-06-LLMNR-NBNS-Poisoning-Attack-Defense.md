---
layout: post
title: LLMNR & NBT-NS Poisoning Explained - A Practical Attack Guide 
date: 2026-04-06
categories:
    - Active Directory
    - MITM
    - Attack
    - Defense
tags:
    - llmnr
    - nbt-ns
    - responder 
    - windows 
---
# Introduction 

![LLMNR/NBT-NS Poisoning](assets/img/posts/llmnr-nbtns-poisoning.png)

**LLMNR/NBT-NS** poisoning est un type d'attaque que l'on retrouve très souvent dans un envrionnement Active Directory. Il exploite l'hiérarchie de résolution de nom afin de voler les informations d'authentification d'un compte du domaine. 

Cette information se présente sous forme de **Hash NTLMv2 ou NTLMv1** pouvant être brute-forcé en local à l'aide d'outils tels que **Hashcat** ou **John The Ripper** ou relayé vers d'autres services ou protocoles.

## Hiérarchie de résolution de nom

Au sein d'un **AD**, le **DNS** (Domain Name System) joue un rôle crucial. Il permet aux clients qui s'authentifient de localiser facilement les contrôleurs de domaines ainsi que les différents serveurs grâce à leur noms de domaine.

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





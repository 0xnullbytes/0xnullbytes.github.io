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
**LLMNR/NBT-NS poisoning**  est un type d'attaque que l'on retrouve très souvent dans un environnement Active Directory. Il exploite l'hiérarchie de résolution de nom afin de voler les informations d'authentification d'un compte de domaine. 

Ces informations se présentent sous forme de **`hash NTLMv2`** ou **`NTLMv1`**. 
Ce type de **`hash`** peut être soumis à une attaque par brute-force en local à l'aide d'outils tels que **`Hashcat`** ou **`John The Ripper`** afin de retrouver le mot de passe en clair.
Ce **`hash`** peut également être relayé vers d'autres services ou protocoles pour s'authentifier sans pour autant connaitre le mot de passe.

## Hiérarchie de résolution de nom

Au sein d'un **Active Directory**, le **DNS** (Domain Name System) joue un rôle crucial. 
Il permet, entre autres:

- **la résolution de noms** : transforme les noms de machines en adresses IP.
- **Localisation des services AD** : via les enregistrements **`SRV`** pour trouver les contrôleurs de domaine, `LDAP`, `Kerberos`, etc.

Lorsqu’un nom d’hôte doit être résolu, Windows tente d’abord une résolution via DNS.  
Si celle-ci échoue, le système utilise ensuite **`LLMNR`**, puis **`NetBIOS Name Service (NBT-NS)`**. 

`LLMNR` (défini par la RFC 4795) et `NBT-NS` (protocole Microsoft) sont des mécanismes de résolution de noms opérant sur le réseau local via multicast et broadcast. Contrairement au DNS, ils ne nécessitent aucune infrastructure centralisée. N'importe quel hôte du segment réseau peut y répondre.

>[!Note]
Avant d’interroger le DNS, le système vérifie également le cache local et le fichier hosts.

Ce qui nous donne le schéma suivant:

```
Cache local → fichier hosts → DNS → LLMNR → NBT-NS
```

## LLMNR (Link Local Multicast Name Resolution)

Défini par la [RFC 4795](https://datatracker.ietf.org/doc/html/rfc4795#section-2), **`LLMNR (Link Local Multicast Name Resolution)`** permet la résolution de nom d'hôtes sur un réseau local en cas d'échec du DNS. Il utilise par défaut le port `UDP 5355` et opère sous le principe de `Sender / Responder`. 
Le `sender` étant celui qui envoie la requête et le `responder` celui qui répond à à cette requête. 
Les requêtes sont envoyées aux adresses multicasts :
- `IPv4 224.0.0.252`
- `IPv6 FF02::1:3` 
Tout hôte sur le même segment réseau peut répondre aux requêtes `LLMNR` (via unicast), ce qui le rend vulnérable aux attaque de type `poisoning`.
## NBT-NS (NetBIOS Name Service)


## Fonctionnement de l’attaque

## Capture des hashes NTLM

## Exploitation des hashes

## Démonstration pratique avec Responder

## Contre-mesures

## Conclusion





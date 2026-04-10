---
layout: post
title: LLMNR, NBT-NS & MDNS Poisoning - Un Guide Pratique
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
**LLMNR, NBT-NS & MDNS poisoning**  est un type d'attaque que l'on retrouve très souvent dans un environnement Active Directory. Il exploite l'hiérarchie de résolution de nom afin de voler les informations d'authentification d'un compte de domaine. 

Ces attaques ciblent principalement les protocoles de résolution locaux tels que :  
  
- **LLMNR (Link-Local Multicast Name Resolution)**  
- **NBT-NS (NetBIOS Name Service)**  
- **mDNS (Multicast DNS)**  
  
Ces protocoles sont utilisés lorsque la résolution DNS échoue ou dans certains mécanismes de découverte réseau locale.

Ces informations se présentent sous forme de **`hash NTLMv2`** ou **`NTLMv1`**. 
Ce type de **`hash`** peut être soumis à une attaque par brute-force en local à l'aide d'outils tels que **`Hashcat`** ou **`John The Ripper`** afin de retrouver le mot de passe en clair.
Ce **`hash`** peut également être relayé vers d'autres services ou protocoles pour s'authentifier sans pour autant connaitre le mot de passe.

## Hiérarchie de résolution de nom

Au sein d'un **Active Directory**, le **DNS** (Domain Name System) joue un rôle crucial. 
Il permet, entre autres:

- **la résolution de noms** : transforme les noms de machines en adresses IP.
- **Localisation des services AD** : via les enregistrements **`SRV`** pour trouver les contrôleurs de domaine, `LDAP`, `Kerberos`, etc.

Windows tente d’abord une résolution via DNS.  
Si celle-ci échoue, le système peut utiliser des mécanismes de résolution locaux tels que **LLMNR** et **NBT-NS**, qui reposent respectivement sur multicast et broadcast.

`LLMNR` (défini par la RFC 4795) et `NBT-NS` (protocole Microsoft) sont des mécanismes de résolution de noms opérant sur le réseau local via multicast et broadcast. 
Contrairement au DNS, ils ne nécessitent aucune infrastructure centralisée. N'importe quel hôte du segment réseau peut y répondre.

> Avant d’interroger le DNS, le système vérifie également le cache local et le fichier hosts.

Ce qui nous donne le schéma suivant:

```
               +--------+
               | Cache  |
               +--------+
                    |
                    v
               +--------+
               | hosts  |
               +--------+
                    |
                    v
               +--------+
               |  DNS   |
               +--------+
                    |
          +---------+---------+
          |                   |
          v                   v
      +--------+         +--------+
      | LLMNR  |         | mDNS   |
      +--------+         +--------+
          |
          v
      +--------+
      | NBT-NS |
      +--------+
```

## LLMNR (Link Local Multicast Name Resolution)

Défini par la [RFC 4795](https://datatracker.ietf.org/doc/html/rfc4795#section-2), **`LLMNR (Link Local Multicast Name Resolution)`** permet la résolution de nom d'hôtes sur un réseau local en cas d'échec du DNS. Il utilise par défaut le port `UDP 5355` et opère sous le principe de `Sender / Responder`. 
Le `Sender` étant celui qui envoie la requête et le `Responder` celui qui répond à à cette requête. 
Les requêtes sont envoyées aux adresses multicasts :
- `IPv4 224.0.0.252`
- `IPv6 FF02::1:3` 

Tout hôte sur le même segment réseau peut répondre aux requêtes `LLMNR` (via unicast), ce qui le rend vulnérable aux attaques de type `poisoning`.

## NBT-NS (NetBIOS Name Service)

**`NTB-NS (NetBIOS Name Service)`** permet de résoudre les **`nom NetBIOS`** (15 caractères max.) en adresses IP. Il utilise le port **`UDP 137`** et envoie des requêtes broadcast sur le segment réseau. 

C'est précisément ce mécanisme qui constitue la faille. N'importe quelle machine peut y répondre sans aucune forme d'authentification. 
## mDNS (Multicast DNS)

**`mDNS (Multicast DNS)`** est un protocole de résolution de noms local défini par la **RFC 6762**.  
Il permet aux machines de résoudre des noms d'hôtes sans utiliser de serveur DNS centralisé.

Contrairement à `LLMNR` et `NBT-NS`, `mDNS` est largement utilisé dans les environnements modernes.

Il utilise :

- **Port :** `UDP 5353`
- **Adresse multicast IPv4 :** `224.0.0.251`
- **Adresse multicast IPv6 :** `FF02::FB`

Les noms mDNS utilisent généralement le domaine : `.local`

Comme `LLMNR`, **n'importe quel hôte du réseau peut répondre à une requête mDNS**, ce qui rend ce protocole vulnérable aux attaques de type **poisoning**.

Un attaquant peut écouter les requêtes multicast et envoyer une réponse frauduleuse avant la machine légitime.

## Fonctionnement de l’attaque

## Capture des hashes NTLM

## Exploitation des hashes

## Démonstration pratique avec Responder

## Contre-mesures

## Conclusion





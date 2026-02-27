# TP5 : pfSense -- Bases d'un pare-feu

## Etudiant :
**Nom :** Giordano\
**Prénom :** Hugo\
**Classe :** B1 - Informatique\
**Date :** 20/02/2026

------------------------------------------------------------------------

# Objectifs

-   Mise en place d'un lab avec plusieurs VMs\
-   Installation et configuration d'un pfSense\
-   Donner accès à Internet à une VM Ubuntu via pfSense\
-   Sécuriser l'administration du pare-feu\
-   Comprendre la logique des règles\
-   Manipuler NAT, DHCP et DNS\
-   Mettre en place un filtrage ciblé

------------------------------------------------------------------------

# Partie 1 -- Prise en main et sécurisation

## 1. Accès à l'interface

### Adresse IP du LAN

**Réponse :**\
\> 192.168.156.2/24

### Adresse IP du WAN

**Réponse :**\
\> 10.0.2.15/24

![Interface IP](image-2.png)

### Pourquoi utilise-t-on HTTPS ?

**Réponse :**\
\> Le protocole HTTPS chiffre les données entre le client et l'interface de pfSense, protégeant ainsi les identifiants et les données sensibles contre les interceptions.

### Pourquoi faut-il changer les identifiants par défaut ?

**Réponse :**\
\> Les identifiants sont par défaut pour tous les utilisateurs donc n'importe qui les connait ce qui en fait une grande faille de sécurité 

------------------------------------------------------------------------

## 2. Sécurisation de l'accès administrateur

### Où se gèrent les utilisateurs ?

**Réponse :**\
\> Les utilisateurs se gèrent dans la section "System" -> "User Manager" sur l'interface de pfSense.
![](image-1.png)

### Définition d'un mot de passe robuste

**Réponse :**\
\> Un mot de passe robuste est un mot de passe qui ne doit pas être facilement devinable même par soi-même, contenant plus de 12 caractères dont des caractères spéciaux ou des chiffres et ne doit donc pas avoir des suites comme "azerty" ou des mots tels qu'un nom.

### Pourquoi sécuriser l'accès admin en priorité ?

**Réponse :**\
\> L'accès admin a tous les privilèges ce qui pourrait être critique si une personne malveillante en aurait les accès.

------------------------------------------------------------------------

# Partie 2 -- Interfaces réseau

## 3. Vérification WAN / LAN

### Interface donnant accès à Internet

**Réponse :**\
\> L'interface WAN (em0) est celle qui donne accès à Internet.

### Interface correspondant au réseau interne

**Réponse :**\
\> L'interface LAN (em1) est celle qui correspond au réseau interne.

### Conséquence d'une inversion des interfaces

**Réponse :**\
\> Si les interfaces WAN et LAN sont inversées, le réseau interne pourrait être exposé directement à Internet, compromettant ainsi la sécurité du réseau.

------------------------------------------------------------------------

# Partie 3 -- Services réseau

## 4. DHCP

### Pourquoi utiliser DHCP ?

**Réponse :**\
\> Le DHCP perrmet d'attribuer automatiquement des adresses IP aux appareils du réseau, ce qui permet une meilleure gestion des adresses et éviter des conflits entre les adresses IP.

### Plage d'adresses choisie

**Réponse :**\
\> La plage d'adresses choisie est de 192.168.156.49 à 192.168.156.199 

### Adresses exclues de la plage

**Réponse :**\
\> Les adresses exclues de la plage sont de 192.168.156.1 à 192.168.156.48 et de 192.168.156.200 à 192.168.156.254 
![DHCP Exclusions](image.png)
### Vérification : Ubuntu obtient-elle une IP automatiquement ?

**Observation :**\
\> Oui, le client obtient une IP automatiquement via le serveur DHCP de pfSense.
\> l'ip obtenue est 192.168.156.49/24
![IP VM](image-3.png)

------------------------------------------------------------------------

## 5. DNS

### Pourquoi pfSense peut-il jouer le rôle de DNS ?

**Réponse :**\
\> pfSense peut jouer le rôle de DNS en utlisant le service DNS Resolver intégré, qui permet de résoudre les noms de domaine en adresses IP pour les clients du réseau.

### Explication si ping 8.8.8.8 fonctionne mais pas les noms de domaine

**Réponse :**\
\> Si le ping vers 8.8.8.8 fonctionne mais pas les noms de domaine, cela indique un problème de résolution DNS. Le client peut atteindre Internet mais ne peut pas résoudre les noms de domaine en adresses IP.

------------------------------------------------------------------------

# Partie 4 -- Accès Internet

## 6. Règles de pare-feu

### Configuration réalisée (description)

**Description :**\
\> J'ai créé une règles de pare-feu sur l'interface LAN :\
-   La première règle autorise tout le trafic LAN Net vers n'importe quelle destination (Any) pour les protocoles TCP, UDP et ICMP.

### Source de la règle

**Réponse :**\
\> LAN Net 

### Destination de la règle

**Réponse :**\
\> Any

### Protocoles autorisés

**Réponse :**\
\> TCP, UDP et ICMP

![firewall-rule](image-4.png)

### Tests réalisés

  Test          |Résultat   |Observation
  ------------- |---------- |----------------------
  Ping pfSense  | ✅       | Le ping vers l'interface LAN de pfSense fonctionne correctement. 
  Ping 8.8.8.8  | ✅       | Le ping vers 8.8.8.8 fonctionne correctement.
  Test DNS      | ✅       | La résolution DNS fonctionne correctement.
  Accès Web     | ✅       | L'accès Web fonctionne correctement.

------------------------------------------------------------------------

## 7. NAT

### Pourquoi le NAT est-il nécessaire avec une interface WAN en NAT ?

**Réponse :**\
\> Le NAT est nécessaire pour permettre aux appareils du réseau interne d'accéder à Internet en utilisant une adresse IP publique partagée, car l'interface WAN est configurée en mode NAT.

en masquant les adresses IP privées du réseau interne.

### Différence NAT automatique / manuel ?

**Réponse :**\
\> Le NAT manuel permet de configurer des règles de NAT spécifiques, alors que le NAT automatique gère automatiquement avec des règles par défaut.

### Vérification qu'une traduction d'adresse a lieu ?

**Méthode utilisée :**\
\> J'ai vérifié la traduction d'adresse en faisant une capture de packet sur l'interface WAN de pfSense pendant que je faisais un ping vers une adresse externe (8.8.8.8) et j'ai observé que l'adresse source du paquet était bien traduite en l'adresse IP publique de l'interface WAN de pfSense. 

------------------------------------------------------------------------

# Partie 5 -- Filtrage

## 8. Blocage d'un site spécifique

### Méthode choisie (IP / Domaine)

**Réponse :**\
\> J'ai choisi de bloquer le site en utilisant le DNS Resolver pour bloquer le domaine "www.facebook.com" en créant une entrée de type "Host Override" qui redirige ce domaine vers une adresse IP non routable (0.0.0.0), ce qui empêche donc les clients du réseau de résoudre le nom de domaine et d'accéder au site.
![facebook DNS Resolver block](image-5.png)

### Impact du HTTPS

**Réponse :**\
\> HTTPS ne change rien dans le blocage du site, car le blocage utilisé se passe au niveau DNS, empêchant la résolution du nom de domaine

### Pourquoi le blocage IP peut être contourné

**Réponse :**\
\> Un site utilise plusieurs IP et peut en changer. 
   Bloquer une seule IP ne suffit pas.

### Analyse des logs

**Observation :**\
\> ``nslookup facebook.com`` retourne ``0.0.0.0``, ce qui confirme le blocage par redirection DNS.
![nslookup facebook](image-6.png)

------------------------------------------------------------------------

## 9. Blocage d'une catégorie (jeux d'argent)

### Méthode propre et maintenable

**Réponse :**\
\> J'ai utilisé une règle du firewall en utilisant un alias de table d'hosts contenant les adresses des sites d'argents en ligne, ce qui permet de maintenir facilement la liste des sites à bloquer en ajoutant ou supprimant des entrées dans l'alias.

### Emplacement de création des alias

**Réponse :**\
\> Les alias sont créés dans la section "Firewall" -> "Aliases" de l'interface pfSense.
![aliases](image-7.png)

### Emplacement de création de la règle

**Réponse :**\
\> La règle de blocage est crée dans la section "Firewall" -> "Rules", sur l'interface LAN, en ajoutant le nom de la règle des alias.
![Rules Alias](image-8.png)

### Vérification du blocage

**Réponse :**\
\> J'ai vérifié le blocage en essayant de ping un site de jeux d'argent (par exemple "betclic.com") depuis la VM cliente, et j'ai constaté que le ping échoue, confirmant que le blocage fonctionne correctement.

------------------------------------------------------------------------

# Partie 6 -- Approfondissement

## 10. Blocage réseaux sociaux

### Alias créé

**Détails :**\
\> J'ai créé un alias de type "Host" nommé "SocialMedia" qui contient les adresses de réseaux sociaux (par exemple, Facebook, X (ex Twitter), Instagram).

### Règle implémentée

**Description :**\
\> J'ai créé une règle de pare-feu sur l'interface LAN qui bloque le trafic vers l'alias "SocialMedia".
![SocialMedia](image-9.png)

### Analyse des logs

**Observation :**\
\> En analysant les logs du firewall, j'ai constaté que les tentatives d'accès aux sites des réseaux sociaux sont bloquées, de l'adresse IP du client vers les adresses IP des sites de réseaux sociaux, confirmant que la règle fonctionne correctement.
![log](image-10.png)

### Effet si placé sous "Pass Any"

**Réponse :**\
\> Si la règle de blocage de réseaux sociaux est placée sous une règle "Pass Any" qui va donc autorisé tout le trafic, alors la règle de blocage ne sera jamais pris en compte car les règles sont utilisés dans l'ordre.

------------------------------------------------------------------------

## 11. Règles horaires

### Horaire créé

**Détails :**\
\> J'ai créé un horaire nommé "WorkHours" qui est actif du lundi au vendredi de 9h à 18h. Que j'ai par la suite appliqué sur mes 2 règles crées au préalable (SocialMedia et JEUX_ARGENT)

![horaire](image-11.png)

### Intérêt en entreprise

**Réponse :**\
\> Cela permet de restreindre l'accès pendant les heures de travail au contenu non productif, améliorant ainsi la productivité des employés.

------------------------------------------------------------------------

## 12. Serveur web local

### Configuration du serveur

**Description :**\
\> 

### Filtrage par IP source

**Réponse :**\
\>

### Filtrage par port

**Réponse :**\
\>

### Pourquoi le pare-feu protège le LAN

**Réponse :**\
\>

------------------------------------------------------------------------

## 13. Logs et analyse

### Différence paquet bloqué / autorisé

**Réponse :**\
\> Un paquet bloqué est un paquet qui ne peut pas arriver jusqu'à sa destination car il est bloqué par une règle de pare-feu, alors qu'un paquet autorisé est un paquet qui peut arriver à sa destination car rien ne le bloque.

### Identification de la règle déclenchée

**Réponse :**\
\> 

### Sens du trafic

**Réponse :**\
\>

------------------------------------------------------------------------

## 15. Filtrage MAC

### Est-ce réellement sécurisé ?

**Réponse :**\
\>

### Pourquoi facilement contournable

**Réponse :**\
\>

------------------------------------------------------------------------

## 16. Portail captif

### Contextes d'utilisation

**Réponse :**\
\>

### Avantages par rapport à une simple règle

**Réponse :**\
\>

------------------------------------------------------------------------

## 17. Sauvegarde / restauration

### Procédure réalisée

**Description :**\
\>

### Importance des sauvegardes régulières

**Réponse :**\
\>

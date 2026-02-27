# TP -- Mise en place d'un serveur OpenVPN sur Ubuntu Server

**Nom :** Giordano\
**Prénom :** Hugo\
**Classe :** B1 - Informatique\
**Date :** 26/02/2026

------------------------------------------------------------------------

# Objectifs

-   Installer et configurer OpenVPN sur Ubuntu Server\
-   Mettre en place une infrastructure de certificats (PKI)\
-   Comprendre le rôle du NAT et du routage IP\
-   Générer un profil client fonctionnel\
-   Diagnostiquer un service système en échec

------------------------------------------------------------------------

# Mise en place

## Environnement

-   VM Ubuntu Server LTS\
-   Réseau NAT\
-   Accès SSH

------------------------------------------------------------------------

# Partie 1 -- Comprendre la PKI

## Questions théoriques

### Rôle d'une autorité de certification (CA)

**Réponse :**\
\> L'autorité de certification (CA) est une entité de confiance qui permet la vérification et la validation des certificats aux sites web, professionnels ou particuliers.

### Différence entre clé privée et certificat

**Réponse :**\
\> Une clé privée doit resté confidentielle car elle permet de prouver que tu es bien le propriétaire. Le certificat est un document qui contient la clé publique, l'identité du propriétaire et la signature CA, il sert a prouver l'authenticité de la clé publique et donc avoir une relation de confiance. 

### Pourquoi un serveur VPN a besoin de certificats

**Réponse :**\
\> Un serveur VPN a besoin de certificats pour authentifier les clients et sécuriser les communications. Les certificats permettent d'établir une connexion chiffrée et de vérifier l'identité du client et du serveur, assurant ainsi la confidentialité des données transmises.

------------------------------------------------------------------------

## Mise en place de l’infrastructure PKI (Easy-RSA)

### Création de l’environnement

**Description des commandes :**\
\> Installation et préparation de l’espace PKI :
```bash
sudo apt install easy-rsa
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
./easyrsa init-pki
```
![](img/TP6_creaPKI.png)

### Génération des éléments de sécurité

**Autorité de Certification (CA) :**\
\> Crée la clé privée et le certificat de la CA.
```bash
./easyrsa build-ca
```
![](img/TP6_build-ca.png)

**Certificat serveur :**\
\> Permet d’identifier le serveur VPN.
```bash
./easyrsa build-server-full server nopass
```
![](img/TP6_build-server.png)

**Certificat client :**\
\> Permet d'authentifier les clients VPN.
```bash
./easyrsa build-client-full hugo nopass
```
![](img/TP6_build-client.png)

**Paramètres Diffie-Hellman :**
\> Utilisés pour l’échange sécurisé des clés.
```bash
./easyrsa gen-dh
```
![](img/TP6_gen-dh.png)

**Clé TLS supplémentaire :**
\> Ajoute une protection supplémentaire contre certaines attaques.
```bash
openvpn --genkey secret ta.key
```


------------------------------------------------------------------------

## Questions

### Où Easy-RSA crée-t-il ses fichiers ?

**Réponse :**\
\> Easy-RSA crée ses fichiers dans le dossier `openvpn-ca/pki`.

### Que contient le dossier pki/ ?

**Réponse :**\
\> Le dossier pki/ contient les éléments de la PKI, notamment les clés privées, les certificats, les demandes de signature et les fichiers de configuration lié au certificats.

### Différence entre gen-req et sign-req

**Réponse :**\
\> gen-req génère une demande de certificat qui contient la clé publique et les informations d'identification du demandeur, tandis que sign-req est utilisé pour signer cette demande avec la clé privée de la CA, créant ainsi un certificat valide.

### Que se passe-t-il si un certificat n'est pas signé ?

**Réponse :**\
\> Si un certificat n'est pas signé, il n'est pas reconnu comme valide donc l'authentification sera impossible.

------------------------------------------------------------------------

# Partie 2 -- Configuration du serveur OpenVPN

## Fichier de configuration

Chemin : `/etc/openvpn/server/server.conf`

### Configuration minimale attendue :

```conf
port 1194
proto udp
dev tun

server 10.8.0.0 255.255.255.0

ca /etc/openvpn/server/ca.crt
cert /etc/openvpn/server/server.crt
key /etc/openvpn/server/server.key
dh /etc/openvpn/server/dh.pem

tls-auth /etc/openvpn/server/ta.key 0
```

------------------------------------------------------------------------

## Questions configuration

### Que signifie dev tun ?

**Réponse :**\
\> `dev tun` signifie que OpenVPN utilisera une interface de type TUN (Tunnel). Ce qui permet de créer un tunnel IP qui est utile pour transmettre des paquets IP entre deux machines.

### Différence UDP / TCP pour un VPN

**Réponse :**\
\> UDP → sera plus rapide car il n'attendra pas de ACK pour chaque paquet
\> TCP → garantit un transfert des données plus fiable mais peut introduire de la latence.

### Plage IP choisie pour le VPN et justification

**Réponse :**\
\> Exemple : `10.8.0.0/24`

On choisit une plage :

- Privée
- Différente du réseau local
- Qui n’entre pas en conflit avec d’autres réseaux

------------------------------------------------------------------------

# Routage et NAT

## Activation du forwarding IP

**Méthode utilisée :**\
\> Ajout de façon permanent en rajoutant la ligne `net.ipv4.ip_forward=1` dans le dossier : `sudo nano /etc/sysctl.conf`

## Mise en place du NAT

**Commande utilisée :**\
\> `sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o enp0s3 -j MASQUERADE`

------------------------------------------------------------------------

## Questions NAT / Routage

### Où se configure ip_forward ?

**Réponse :**\
\> ip_forward se configure dans le fichier `/etc/sysctl.conf`

### Commande pour afficher les règles NAT actuelles

**Réponse :**\
\> `sudo iptables -t nat -L -n -v`

### Pourquoi masquerader le réseau VPN ?

**Réponse :**\
\> Parce que le réseau VPN (10.8.0.0/24) n’est pas connu sur Internet.  
Le masquage remplace l’IP source par celle du serveur pour permettre l’accès Internet.

------------------------------------------------------------------------

# Démarrage et analyse du service

## Démarrage du service

**Commande utilisée :**\
\> sudo systemctl restart openvpn-server@server


## Vérification de l'état

**Commande utilisée :**\
\> sudo systemctl status openvpn-server@server
![](img/TP6_start-service.png)

------------------------------------------------------------------------

## Diagnostic en cas d'échec

### Commande pour afficher les logs d'un service

**Réponse :**\
\> 

### Différence entre status et journalctl

**Réponse :**\
\>

### Vérification des chemins certificats

**Observation :**\
\>

------------------------------------------------------------------------

# Partie 3 -- Création du profil client

## Fichier .ovpn
fichier : `client.ovpn`

**Configuration client :**\
\> 
```conf
client
dev tun
proto udp
remote 10.0.2.15  2194

resolv-retry infinite
nobind
persist-key
persist-tun

remote-cert-tls server
cipher AES-256-GCM
auth SHA256

key-direction 1
```

------------------------------------------------------------------------

## Questions client

### Intégration d'un certificat dans un fichier .ovpn

**Réponse :**\
\> 

### Pourquoi la clé privée ne doit jamais être partagée

**Réponse :**\
\>

------------------------------------------------------------------------

# Tests et validation

## Connexion VPN

**Résultat :**\
\>

## Adresse IP obtenue

**Adresse :**\
\>

## Vérification accès Internet via tunnel

**Méthode et résultat :**\
\>

------------------------------------------------------------------------

## Questions validation

### Comment vérifier que le trafic passe par le VPN ?

**Réponse :**\
\>

### Que se passe-t-il si le port 1194 est bloqué ?

**Réponse :**\
\>

------------------------------------------------------------------------

# Partie 4 -- Bonus

## Ajout de plusieurs clients

**Méthode :**\
\>

## Révocation d'un certificat

**Méthode :**\
\>

## Passage en TCP

**Modification réalisée :**\
\>

## Authentification mot de passe + certificats

**Configuration :**\
\>

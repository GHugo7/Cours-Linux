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
```bash
<ca>
-----BEGIN CERTIFICATE-----
MIIDPzCCAiegAwIBAgIUSCIojZjnLG/Prro2YjTkRnxwiXMwDQYJKoZIhvcNAQEL
BQAwEjEQMA4GA1UEAwwHSHVnby1DQTAeFw0yNjAyMjYxNTM4MzJaFw0zNjAyMjQx
NTM4MzJaMBIxEDAOBgNVBAMMB0h1Z28tQ0EwggEiMA0GCSqGSIb3DQEBAQUAA4IB
DwAwggEKAoIBAQDtyDN7pIhfsOkA+ME/GhreucoOi5N3Uf43QAh4XwOKzW39mX0+
7D+y2nHVYe26Mo0AGxZA45GveWyGcRLUCwe0t2D/t+JVhJoxnGKKnH71YWSgmLpa
lOylbl/UrJKPWgrofIRkmte6MXJiVlbA02PD7/p2bRrnT/0hx2GiUVTwSjcfp5bg
g363nvc4atWi9oFIS5uuUIdQXCil656D5WaY4/LDQgVKjXmvyEdazGlSFUc5PJoJ
0tiIMxHigMefMs4leYbStuIIl7jcQ9IrtuWpy3nHrB6hDRu+K5/Jc2GdTKqZA+v7
lOhTY25v3CfslEWc+LLakz1UeDh3C0g5srvvAgMBAAGjgYwwgYkwDAYDVR0TBAUw
AwEB/zAdBgNVHQ4EFgQUpx9rLZ54Pc23MIKgHjJ9R8K6QIwwTQYDVR0jBEYwRIAU
px9rLZ54Pc23MIKgHjJ9R8K6QIyhFqQUMBIxEDAOBgNVBAMMB0h1Z28tQ0GCFEgi
KI2Y5yxvz666NmI05EZ8cIlzMAsGA1UdDwQEAwIBBjANBgkqhkiG9w0BAQsFAAOC
AQEAr7hbY5aI0fNXtTb9xETn/UxJNkmeL52o05U90IIpMNWDwIUG/9PpKbrdn2/D
27V8VRU/S9KNIYVAFX6xybT3KgebgwrKWQTJ51/ofprzIOQf6qqpkWN52dLYSCvM
aTwfBRGLSfQNX7ZQCJXtlucBlFYjvE90e8+kgIbHmV6ILZOGewJqrt11FAOOv1fT
24KO5BC/sEHjZQ7+3RgMw7gbKFAAleT6fUooHsCzqBf3wzY2kCaq2BE/HLdbiRsD
6UD022/VRi3XYk5iS3nXs7tgjOgc3dbRYu8+4vj0geR1DdINI3K3wqrGIFjfNt1M
HGD8y7xEp6j6EOjFBsRzChF8oQ==
-----END CERTIFICATE-----
</ca>

<cert>
-----BEGIN CERTIFICATE-----
MIIDSjCCAjKgAwIBAgIQQ2jtzZeNZGSqAxDrAUfvjzANBgkqhkiG9w0BAQsFADAS
MRAwDgYDVQQDDAdIdWdvLUNBMB4XDTI2MDIyNjE1MzkzN1oXDTI4MDUzMTE1Mzkz
N1owDzENMAsGA1UEAwwEaHVnbzCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoC
ggEBALH0tAhGKxr+PIWCgIH2UyO02WrbBDPIXWfyCiNSdhyGEQiQ+m5Xcmtuw+yb
MCjMMkx6oohbhrkgpdt0uIKQaxvjDaNFG1wDTqcEwvuZZYCL3+44KijdZE0FzOiu
4I7y5+uLvtoYrBdSab1TevA+ipKKvQfEhIx2MvVd045wj25p2RuP3I0yTZI+DiUq
4H5zA4VYOUSaSqxYzhfYP4U1EYCHY9zbl0jwOQjEOp+DgVr+WZVIvPXmktTtptrr
IQeULd0VCl25B82U7HTFzor0+xlZMdHCgzvgMSaycXiUfPU0Czbxv59KWX8PERdB
Y6tC66Pbupqx415WDaTiM/uWcFUCAwEAAaOBnjCBmzAJBgNVHRMEAjAAMB0GA1Ud
DgQWBBRTJqqxCdvMV6ik8lD0yCUensDs9DBNBgNVHSMERjBEgBSnH2stnng9zbcw
gqAeMn1HwrpAjKEWpBQwEjEQMA4GA1UEAwwHSHVnby1DQYIUSCIojZjnLG/Prro2
YjTkRnxwiXMwEwYDVR0lBAwwCgYIKwYBBQUHAwIwCwYDVR0PBAQDAgeAMA0GCSqG
SIb3DQEBCwUAA4IBAQAv6TkBgqxu7kJxwwS53YcHtCgSxitCJIcolfNU7HHgRfHS
sUsMn6PtzYJo47bcxdVTqbL6/DNa+T9Vvn2BO7hycxrwZafdHB7kmLrzHosh4gGa
K7sRurmTVRqXYb0KvNr1GptWhj1BrGS5HscsqCf443JsC/yhxk3xY90YmbA8pg7J
zeZI0Vmrkz2GhTDougWAbnLyXLGoe1oZ0cxjqM4H+P1iqIlzoaOpCtk59k+11PVq
UA36QylygVSrWf8ImJ/yJ0gnkgyy9pcRQXcnVd92HE3uxkUETZH8zBqAcDftu0Gr
+q0BbvxX2yO00gG3fvhz+WKHyb5clB99kXMlOrGN
-----END CERTIFICATE-----
</cert>

<key>
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCx9LQIRisa/jyF
goCB9lMjtNlq2wQzyF1n8gojUnYchhEIkPpuV3JrbsPsmzAozDJMeqKIW4a5IKXb
dLiCkGsb4w2jRRtcA06nBML7mWWAi9/uOCoo3WRNBczoruCO8ufri77aGKwXUmm9
U3rwPoqSir0HxISMdjL1XdOOcI9uadkbj9yNMk2SPg4lKuB+cwOFWDlEmkqsWM4X
2D+FNRGAh2Pc25dI8DkIxDqfg4Fa/lmVSLz15pLU7aba6yEHlC3dFQpduQfNlOx0
xc6K9PsZWTHRwoM74DEmsnF4lHz1NAs28b+fSll/DxEXQWOrQuuj27qaseNeVg2k
4jP7lnBVAgMBAAECggEAK2hXKdWD2je9p4tnsUvWh9UrW4dFBSSQtDQ0CN2qdda1
/PLwQ04NWOtR2zsXijwU4NbhIoXA3RN7oYZdI2v61HiT8QmAXPdpjg/5R8npGmwE
GWWV0xX9y1Po4bEWkYfqzmfuC+EMyTuPE2FzkXqP7qLs7SIgeuqyD2vtmMcYFYRQ
Ddnn4bs4SAcR28Mj9tpZRqOekgUFWdMq8zvTkVtF0w3TPBqrhVt3JPtCXSFFYVR5
pBdga98LPS/dWgUDkpHFmk/C0mBrbzdh8hd1T9te9Nba+yHJ9WqGX8fNWy6sTgnY
g05NOHN2q2ux7/X/oB9qSWbgaypo5yrl1eWhHOetgQKBgQDZZ9aN6bUY6sQEQEq7
38LD0BM69Q3DY8fH1gix/pzX8vxjJWnd/NIgCBBYTU7KxUswKJ9BTMJcsP0nCgG/
GCONTskvbtdRG0mdUIk+MBsGK3I4ypODbQmujwzd7pAETfO/WD+UnVdq0YKafda8
FcX6PpPPS5eBLTexyxvwgDlksQKBgQDRjAqx1dHKnxiD4B6R7n9O123Xf6EqIeZ+
rQ9nZLpAbwagWGsqAOZklOwB9WUA2R8EQ0wtXT5cUGigtfAt1Rdk80p1uO02+Xq5
WtqZhp9ttCBvFWRGw0SgNqGN+4Y5nFv1sjIs8wNQD5zXAldpk2O694jHX3TD+u4s
LiUVGMS+5QKBgQCX17N9yKZIseeYBINt9pmkg9Z6jh8/wZgOVcoa73cCSN7ILKBl
gCgOYTNxSQVoECY0i5q3U/JIJQGa251ep3BlSIqxi1vtdP6UVmSUv6qQQ5XyXmJr
H6PsfFMDSpThvMQqd2to0//zRkNOvedV1GtDHaPE28oEWd2VWwO3lQcA8QKBgGND
SImndSLbrJxD3ZdZeBsb999+iRTRTEOzrTlYQNZnAaeRWuph1MHOveHLohX9xDCb
xIk0w2atfHKs3OKJL/TVPu93M3+4PIdzX1wcpocsLbURWBghRe/zESKWBKZjyDgi
OpyKXYZebvh3Fntfo32c5sEzGbgAtTReg5hzzDd1AoGAKzRtw5AT+wl5Px+782Tf
ckSWE+KGcxIAiu33m3z91VH9U9Cc/JtJjprQ5StY6Wy4v9KLoBPyC9C1K7cbg9tU
2HVVY8B0wrImB6KjKv5lh5mNXaZGCB2j6xmvlaCPYuM2TgSvC9KlLNA4n5uXxlVA
bQeeYCKCTTjikw0/Sj7DsqM=
-----END PRIVATE KEY-----
</key>

<tls-auth>
-----BEGIN OpenVPN Static key V1-----
960218ee0f3081a54455d8659d6016c1
a08091f76c49da97ab5b261e2f1dcfad
d2167fc7001db15407a7fc334a22fbd1
2e24c4db5fd00b55fbf7cbca89b684ec
cd9d008dbdb8f864b162b774d85ae85a
365fce76d2ca9c70bb606ed84784dad2
0f60abb86a315ae8fc0d7ab95f90452e
73573b471339c8f76845ed84fdf50040
a6706c1fcf60718bc8f03f73b50f25d2
470181efc37d45452489333ef3c196ed
e15138be419ebbe7509b1d1c9e004f2d
daed41c6dc0fab7de3d7a69e3925fbc9
b904fd378daf0c55e97d0fd413e2a3a8
affab26be4a87921600e74d243ad8490
1e7ee1d457ffdb61d73be31d7b94f61f
a537295c459d5c77f1bf1a18dd10ba81
-----END OpenVPN Static key V1-----
</tls-auth>
```

### Pourquoi la clé privée ne doit jamais être partagée

**Réponse :**\
\> Car elle permet de prouver l’identité du client.  
Si elle est partagée, quelqu’un peut se connecter au VPN à sa place.

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

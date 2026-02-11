# 📘 Compte-rendu TP — Réseau & Linux

**Nom :** Hugo Giordano  
**Classe / Groupe :** B1 - Informatique 
**Date :** 23/01/2026
**Système :** Fedora Linux (KDE Plasma)  
**Interface réseau :** wlp195s0  
**Type de réseau :** Wi-Fi WPA/WPA2-Enterprise

---

# I. Exploration locale en solo
## 1. Affichage d'informations sur la pile TCP/IP locale
### En ligne de commande
#### Interface Wi-Fi
- nom: wlp195s0
- adresse IPv4: 10.33.64.149
- adresse MAC: 26:b2:fa:53:3c:40
- adresse réseau: 10.33.64.0
- adresse broadcast: 10.33.79.255
#### Interface Ethernet
- nom: eth1 
- adresse IPv4: 
- adresse MAC: 0c:37:96:d5:59:1f
- adresse réseau: 
- adresse broadcast: 

#### Affichage de la gateway
- Gateway: `ip route show` -> 10.33.79.254

## 2. Modifications des informations
### A. Modification d'adresse IP - pt. 1
- IPv4 changé par l'adresse 10.33.64.69/20
### B. nmap
### C. Modification d'adresse IP - pt. 2
- IPv4 changé par l'adresse 10.33.68.69/20

---

# II. Exploration locale en duo
## 3. Modification d'adresse IP

---

# III. Manipulations d'autres outils/protocoles côté client
## 1. DHCP
- Adresse IP du DHCP n'est pas identifiable.
- Durée du bail non fourni 
- Demande d'un nouvelle adresse IP (avec nmcli)
	- `nmcli device disconnect wlp195s0`
	- `nmcli device connect wlp195s0`
## 2. DNS
- `nmcli device show wlp195s0 | grep IP4.DNS`
   - J'utlise donc le serveur DNS 8.8.8.8 (Google) principal et 1.1.1.1 (Cloudflare) en secondaire 

```bash
IP4.DNS[1]:                             8.8.8.8
IP4.DNS[2]:                             1.1.1.1
```

- nslookup google.com
```bash	
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   google.com
Address: 172.217.20.46
Name:   google.com
Address: 2a00:1450:4007:80f::200e
```
- dig ynov.com
```bash
; <<>> DiG 9.18.44 <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44891
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;ynov.com.                      IN      A

;; ANSWER SECTION:
ynov.com.               220     IN      A       104.26.10.233
ynov.com.               220     IN      A       172.67.74.226
ynov.com.               220     IN      A       104.26.11.233

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Wed Feb 11 01:18:03 CET 2026
;; MSG SIZE  rcvd: 85
```

### Reverse lookup
- L'adresse 78.78.21.21 renvoie un domain appartenant à "Telia.com"
- L'adresse 92.16.54.88 renvoie un domain appartenant à "as13285.net"

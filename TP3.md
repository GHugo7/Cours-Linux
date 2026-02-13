# A. Base64
## 1. Génération d’un fichier binaire
- La commande `dd if=/dev/urandom of=data.bin bs=1k count=100` permettra de créer un fichier "data.bin" contenant 100Ko de données binaires aléatoires
![](img/binary_file_check.png)

## 2. Encodage
- La commande `openssl base64 -e -in data.bin -out data.b64` va encrypter un fichier sur la base64 et l'exporter dans un nouveau : data.bin -> data.b64	
![](img/encrypt_base64.png)
- La commande suivante va compter le pourcentage de différence entre les 2 fichiers data.bin et data.b64 : 
```bash
differ=$(cmp -l data.bin data.b64 2>/dev/null | wc -l)
total=$(stat -c %s data.bin)
echo "scale=2; $differ*100/$total" | bc
```
- Il y a donc 99,63% de différence donc seulement 0,37% de similitude 

## 3. Décodage
- La commande `openssl base64 -d -in data.b64 -out data_recovered.bin` fera l'effet inverse est decryptera le fichier data.b64 vers un nouveau : data_recovered.bin
![](img/decrypt_base64.png)

```bash
differ=$(cmp -l data.bin data_recovered.bin 2>/dev/null | wc -l)
total=$(stat -c %s data.bin)
echo "scale=2; $differ*100/$total" | bc
```
- 0% de différence entre les 2 fichiers, ils sont donc parfaitement identiques

## 4. Questions
#### 1. Base64 est-il un chiffrement ? Pourquoi ?
- Non base64 est un codage car il ne nécessite pas de clé de décryptage.
#### 2. Pourquoi la taille du fichier change-t-elle après encodage ?
- La taille du fichier change car on remplace des données binaires compactes par des système d'encodage plus longue par exemple 3 octects en base64 aura maintenant 4 octects.

#### 3. Quel est approximativement le pourcentage d’augmentation ?
- en base64 l'augmentation sera donc de 4/3 = 1.33 = 33% d'augmentation 

#### 4. Quelle méthode permet de vérifier rigoureusement que deux fichiers sont identiques ?
- La commande `diff <fichier1> <fichier2> vérifiera la différence entre ces 2 fichiers`

# B. Chiffrement symétrique – AES
## 1. Création d’un message
- Voici mon fichier confidentiel.txt :
![](img/confideltiel.txt.png)
## 2. Chiffrement
- La commande `openssl enc -aes-256-cbc -salt -pbkdf2 -md sha256 -i confidentiel.txt -out confidentiel.enc` encrype mon fichier confidentiel.txt avec - le système aes256-cbc grâce au hashage sha256, utilisation de PBKDF2 pour dériver la clé depuis le mot de passe et salt pour ajouter un sel
![](img/aes_encryption.png)
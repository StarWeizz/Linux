# Compte rendu – TP3 Cryptographie avec OpenSSL

Nom : RUSSO Antonin

## A. Base64

### 1. Génation d’un fichier binaire
#### Commande utilisée qu'on a vu dans le tp 3
```bash
dd if=/dev/urandom of=data.bin bs=1k count=100
```

#### Vérification de la taille avec ls et les paramètres
```
ls -lh data.bin
```

#### Résultat :
Le fichier fait 100 Ko

### 2. Encodage Base64
#### Commande
```
openssl base64 -e -in data.bin -out data.b64
```

#### Affichage
On affiche le contenu du fichier
```
cat data.b64
```

On observe un fichier texte illisible mais qui semble structuré

#### Comparaison des tailles
```
ls -lh data.bin data.b64
```

Constat :
data.b64 est plus volumineux que data.bin de 36ko

### 3. Décodage
#### Commande
```
openssl base64 -d -in data.b64 -out data_restored.bin
```

Vérification d’intégrité
```
diff -s data.bin data_restored.bin
```

Résultat :
ça dit que les fichiers sont identiques.
```bash
antonin@antonin-G5-5590:~/Documents/linux-b1/TP3$ diff -s data.bin data_restored.bin 
Files data.bin and data_restored.bin are identical
```

### 4. Questions

#### 1. Base64 est-il un chiffrement ?
> Non. Base64 est un encodage, pas un chiffrement. Il ne protège pas les données.

#### 2. Pourquoi la taille change ?
> Base64 convertit 3 octets en 4 caractères ASCII, ce qui augmente la taille

#### 3. Pourcentage d’augmentation ?
> Environ 33 %

#### 4. Méthode rigoureuse de vérification ?

La commande diff :
```
diff -s fichier1 fichier2
```

## B. Chiffrement symétrique – AES
### 1. Création du message

Création du fichier :

```
nano confidentiel.txt
```

Contenu :

```
RUSSO Antonin
14/02/2026
Bordeaux
Fichier confidentiel
de 5 lignes
```

### 2. Chiffrement
Commande utilisée
```
openssl enc -aes-256-cbc -salt -pbkdf2 -md sha256 -in confidentiel.txt -out confidentiel.enc
```

Vérification du type de fichier
```
file confidentiel.enc
```

Résultat :
Le fichier est détecté comme données binaires et ça dit que le fichier est avec protégé avec un mot de passe de salage.

Si on fait :

```
cat confidentiel.enc
```

On observe des caractères illisibles mais le fichier commence par Salted_

### 3. Déchiffrement
```
openssl enc -d -aes-256-cbc -pbkdf2 -md sha256 -in confidentiel.enc -out confidentiel_dechiffre.txt
```

#### Vérification :
```
diff -s confidentiel.txt confidentiel_dechiffre.txt
```

Résultat :
Les fichiers sont identiques

### 4. Analyse – Double chiffrement

On chiffre une seconde fois :

```
openssl enc -aes-256-cbc -salt -pbkdf2 -md sha256 -in confidentiel.txt -out confidentiel2.enc
```

Comparaison :
```
diff confidentiel.enc confidentiel2.enc
```

Résultat :
Les deux fichiers sont différents mais commencent tous les deux par Salted_

### 5. Questions
#### 1. Pourquoi les deux fichiers chiffrés sont différents ?

> Parce qu’un sel aléatoire est généré à chaque chiffrement basé sur un mot d'encryption

#### 2. Quel est le rôle du sel ?

> Il permet de rendre chaque chiffrement unique et il empêche d’identifier deux fichiers identiques chiffrés avec le même mot de passe comme j'ai fait

#### 3. Que se passe-t-il si une option change lors du déchiffrement ?

> Le déchiffrement échoue ou produit un fichier corrompu.
Les paramètres doivent être strictement identiques (algorithme, hash, PBKDF2…).

#### 4. Pourquoi utilise-t-on PBKDF2 ?

> PBKDF2 permet de renforcer la sécurité des mots de passe, il ralentit les attaques par force brute et dérive une clé robuste à partir d’un mot de passe faible comme `admin` que j'ai mis

#### 5. Différence entre encodage et chiffrement ?

> Encodage : transformation réversible sans secret (ex: Base64)

> Chiffrement : transformation nécessitant une clé secrète

## C. Cryptographie asymétrique – RSA
### 1. Génération des clés
Génération de la clé privée protégée
```
openssl genpkey -algorithm RSA -aes-256-cbc -out rsa_private.pem -pkeyopt rsa_keygen_bits:2048
```

Extraction de la clé publique
```
openssl rsa -in rsa_private.pem -pubout -out rsa_public.pem
```
Affichage des paramètres de la clé privée
```
openssl rsa -in rsa_private.pem -text -noout
```

On observe :

Modulus (n)

Public Exponent (e)

Private Exponent (d)

Prime1 (p)

Prime2 (q)

Exponents associés

Coefficient

#### Affichage des paramètres de la clé publique
```
openssl rsa -in rsa_public.pem -pubin -text -noout
```

On observe uniquement :

Modulus (n)

Public exponent (e)

#### Comparaison

La clé privée contient tous les paramètres mathématiques nécessaires et la clé publique ne contient que le modulo et l’exposant public.

### 2. Chiffrement asymétrique
Création du fichier
```
nano secret.txt
```

Chiffrement
```
openssl pkeyutl -encrypt -pubin -inkey rsa_public.pem -in secret.txt -out secret.enc
```

Déchiffrement
```
openssl pkeyutl -decrypt -inkey rsa_private.pem -in secret.enc -out secret_dechiffre.txt
```

Vérification :
```
diff -s secret.txt secret_dechiffre.txt
```
Résultat : les fichiers sont identiques
### 3. Questions
#### 1. Pourquoi la clé privée ne doit jamais être partagée ?

> Car elle permet :
>
> De déchiffrer les messages
>
> De signer des documents
>
#### 2. Pourquoi RSA n’est pas adapté aux gros fichiers ?

> Opérations mathématiques lourdes
> 
> Taille maximale limitée
> 
> Lent comparé à AES

#### 3. Différences clé publique / clé privée ?
Clé publique
> Composition `n + e` et elle est partagée	

Clé privée
> Composition `n + e + d + p + q + autres paramètres` et elle est secrète

#### 4. Rôle du modulo dans RSA ?

> Le modulo (n = p × q) :
>
> Base mathématique du système
>
> Sa factorisation est extrêmement difficile
>
> Garantit la sécurité

#### 5. Pourquoi RSA chiffre souvent une clé AES ?

> Parce que :
>
> AES est rapide pour les gros fichiers
>
> RSA est utilisé pour sécuriser la clé AES
>
> C’est le principe du chiffrement hybride (comme TLS)

## D. Signature numérique
### 1. Création et signature
Création
```
nano contrat.txt
```
Empreinte
```
openssl dgst -sha256 contrat.txt
```
Résultat :
```bash
antonin@antonin-G5-5590:~/Documents/linux-b1/TP3$ openssl dgst -sha256 contrat.txt
SHA2-256(contrat.txt)= 181210f8f9c779c26da1d9b2075bde0127302ee0e3fca38c9a83f5b1dd8e5d3b
```
Signature
```
openssl dgst -sha256 -sign rsa_private.pem -out contrat.sig contrat.txt
```
### 2. Vérification
```
openssl dgst -sha256 -verify rsa_public.pem -signature contrat.sig contrat.txt
```

```bash
antonin@antonin-G5-5590:~/Documents/linux-b1/TP3$ openssl dgst -sha256 -verify rsa_public.pem -signature contrat.sig contrat.txt
Verified OK
```
Résultat : Verified OK

Modification du fichier

On ajoute une ligne dans contrat.txt

Nouvelle vérification :
```
openssl dgst -sha256 -verify rsa_public.pem -signature contrat.sig contrat.txt
```

```bash
antonin@antonin-G5-5590:~/Documents/linux-b1/TP3$ openssl dgst -sha256 -verify rsa_public.pem -signature contrat.sig contrat.txt
Verification failure
40A7A4D385700000:error:02000068:rsa routines:ossl_rsa_verify:bad signature:../crypto/rsa/rsa_sign.c:430:
40A7A4D385700000:error:1C880004:Provider routines:rsa_verify:RSA lib:../providers/implementations/signature/rsa_sig.c:774:
```
Résultat : Verification Failure

### 3. Questions
#### 1. Que se passe-t-il après modification ?

> La signature devient invalide et ne marche plus

#### 2. Pourquoi ?

> Parce que le hash du fichier change et que la signature correspond à l’empreinte exacte du fichier original qu'on a créé avant qu'il soit modifié.

#### 3. Rôle du hachage ?

> Réduit le document à une empreinte fixe
>
>Rend la signature rapide
>
>Garantit l’intégrité

#### 4. Différence signature / chiffrement ?
Le Chiffrement :
> Protège la confidentialité
>
> Utilise clé publique pour chiffrer
>
> Déchiffrement avec clé privée

La signature : 
> Garantit authenticité et intégrité
>
> Utilise clé privée pour signer
>
> Vérification avec clé publique
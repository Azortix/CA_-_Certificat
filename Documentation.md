## Introduction
CA est l'accronyme de _Certificate Authority_, soit une Autorité de Certificats. Elle permet de délivrer des certificats et permettre d'approuver (ici au sein de
l'entreprise) les machines les recevant.  
C'est un système qui repose sur la confiance, il est donc extrêmement important de sécuriser notre CA.  
Une CA corrompuepeut ainsi approuver des machines non sensées l'être, ce qui serait dramatique avec des conséquences de pertes de données et autre.  

## Création d'une CA
Sur l'interface web d'OPNsense, aller dans **System**, **Trust**, **Authorities**.  
Cliquer sur **+**.  
**Method** : Create an internal Certificate Authority (ou choisir de l'importer si une est déja créée)  
**Descriptive name** : Choisir un nom  
**Key length** : 2048 ou 4096 bits (4096 plus sûr, mais un peu plus lent)  
**Digest Algorithm** : SHA256  
**Lifetime** : 999999 days  


## Déploiement de la CA sur les postes
### Exportation de la CA
Dans **System**, **Trust**, **Authorities**, une fois la CA créée, il est possible de la télécharger via les options à droite.  
Un fichier PEM est ainsi obtenu, avec le nom donné à sa création.  
Il n'est pas nécessaire de placer ce fichier dans un dossier partagé avec chaque machine. Une fois l'import fait, il pourra même être archivé ou supprimé.  


### Installation de la CA sur les postes
Il faut désormais installer la CA sur chaque poste, cela permettra d'approuver et d'authentifier les machines pour l'utilisation de VPN, de serveurs internes, pour l'envoi de mails, ou même pour approuver le proxy explicite.  

Une manière efficace et globale est d'utiliser une GPO.  
Pour cela, sur le serveur AD, ouvrir la console de gestion des GPO via `Win + R` et entrer **gpmc.msc**.  
Clic droit sur l'UO voulue et **Create a GPO in this domain, and Link it here…**  
Choisir son nom.  
Clic droit **Edit**.  
Suivre l'arborescence :  
**Computer Configuration**, **Policies**, **Windows Settings**, **Security Settings**, **Public Key Policies**, **Trusted Root Certification Authorities**  
Clic droit **Trusted Root Certification Authorities** puis **Import**  
**Next**, puis Sélectionner le fichier .crt exporté plus tôt. **Next**.  
Cocher **Place all certificates in the following store: Trusted Root Certification Authorities** puis **Next** et **Finish**.  

Il est possible d'appliquer directement les changements de GPO avec ``gpupdate /force``. Sinon, attendre le rafraichissement automatique.  

Il est possible de vérifier que le certificat est bien installé via ``Win + R`` et entrer **certmgr.msc**. Aller dans **Trusted Root Certification Authorities** puis **Certificates** et vérifier que la CA est bien présent  


## Création d'un certificat
Sur l'interface web d'OPNsense, aller dans **System**, **Trust**, **Certificates**.  
Cliquer sur **+**.  
**Method** : Create an internal Certificate (ou choisir de l'importer si une est déja créée)  
**Descriptive name** : Choisir un nom  
**Type** : Sélectionner Client/Server Certificate selon son utilisation  
**Key type** : `RSA-2048` ou `4096` bits (4096 plus sûr, mais un peu plus lent)  
**Digest Algorithm** : `SHA256` est un bon choix  
**Issuer** : Sélectionner la CA créée plus tôt  
**Lifetime** : 999999 days  

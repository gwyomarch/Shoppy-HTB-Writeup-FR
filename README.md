# Shoppy HTB

![Shoppy HTB](img/Shoppy.png)

## Enumeration

### NMAP

![Nmap Scan](img/01-Nmap.png)

3 services présents:
- SSH sur le port 22
- HTTP sur le port 80
- copycat sur le port 9093

Ajoutons le nom de domaine **shoppy.htb** à notre **/etc/hosts**.
Et allons voir manuellement avec un navigateur.

![Website](img/03-Website.png)

"Coming soon!", peut-être pourrons-nous trouver un hôte virtuel qui héberge une version en développement...

### GOBUSTER

Continuons avec l'énumération de l'arborescence du site à la recherche d'autres pages.

![Gobuster Directories](img/02-Gobuster-dir.png)

Nous avons une page de connexion **/login**.

![Gobuster Subdomains](img/12-Subdomains.png)

Et un sous-domaine **mattermost.shoppy.htb** sur lequel nous reviendrons plus tard. Ajoutons-le au fichier **/etc/hosts**.

## EXPLOITATION

![Login Page](img/04-Login-page.png)

Après quelques essais pour contourner l'authentification avec des méthodes basiques d'injection, le système a définitivement un problème avec les apostrophes ' (%27)

![BurpSuite](img/05-BurpSuite.JPG)

### NoSQLi AUTHENTICATION BYPASS

Finalement, on y arrive avec une [injection NoSQL](https://book.hacktricks.xyz/pentesting-web/nosql-injection#sql-mongo).

![Bypass Login Page](img/06-Bypass-login.png)

On se retrouve sur une page administration.


![Administration](img/07-Administration.png)

Et, semble-t-il, une fonctionnalité pour rechercher des utilisateurs !?
La base de données est probablement la même... Essayons la même injection que pour nous connecter !
 
![Search Users](img/08-Search-Users.png)

Ce qui nous permet d'exporter leurs données.

![Export Users](img/09-Export-Users.png)

2 utilisateurs avec les hashs MD5 de leurs mots de passe:
- admin 
- josh

![Get Users](img/10-Users.JPG)

Et les hashs des leurs mots de passe.
Crackons-les avec JohnTheRipper.

![Josh Password](img/11-Josh.JPG)

Seul celui de **Josh** est dans *rockyou.txt*.
Revenons maintenant au sous-domaine **mattermost.shoppy.htb**.

![Mattermost](img/13-Mattermost.JPG)

Une autre page de connexion, mais cette fois, nous avons les identifiants de **Josh**.

![Chat Application](img/14-Chat-App.JPG)

On arrive sur ce qui semble être la messagerie interne de l'entreprise.
Et d'autres identifiants, un nom d'utilisateur et son mot de passe, sont fournis dans la section qui parle du déploiement.

![Jaeger Credentials](img/15-Jaeger.JPG)

## FOOTHOLD

En nous connectant en SSH en tant que **Jaeger**, nous obtenons un point d'entrée stable sur la machine cible.

![SSH Login](img/16-SSH-Foothold.JPG)

Et nous pouvons récupérer le flag **user.txt**

![User Flag](img/17-User-Flag.JPG)

Voyons les permissions *sudo* de l'utilisateur **jaeger** sur la machine.

![Sudo Check](img/18-Sudo-Check.JPG)

Il a le droit d'exécution sur un fichier en tant que **deploy**. Apparemment un gestionnaire de mots de passe rudimentaire, qui demande un premier mot de passe maître pour lire un fichier dans lequel sont écrits d'autres identifiants.

Mais **jaeger** a également les droits de lecture dessus. Un simple **cat** suffit à le révéler.

![Cat File](img/19-Cat-File.JPG) 

Maintenant que nous avons le "master password", nous pouvons lire les identifiants de l'utilisateur **deploy**.

## LATERAL MOVEMENT

![Deploy Credentials](img/20-Deploy-Creds.JPG)

## PRIVILEGE ESCALATION

Ce nouvel utilisateur fait partie du groupe **docker**, qui permet de faire de la sorcellerie...

![Docker Group](img/21-Docker-Group.JPG)

Comme monter le dossier */root* (ou bien l'ensemble du système) dans un container dont il a tout les droits.

[GTFOBins](https://gtfobins.github.io/gtfobins/docker/#shell)

![Get Root](img/22-Get-Root.JPG)

Merci à [lockscan](https://app.hackthebox.com/users/217870) pour cette box très accessible pour les débutants.

N'hésitez pas à aller lui donner du "respect" sur son profil si elle vous a plu.

### Liens:
- [injection NoSQL sur une Base de données MongoDB](https://book.hacktricks.xyz/pentesting-web/nosql-injection#sql-mongo)
- [Escalade de privilèges grâce à Docker sur GTFOBins](https://gtfobins.github.io/gtfobins/docker/#shell)
- [Profil HackTheBox de lockscan](https://app.hackthebox.com/users/217870)



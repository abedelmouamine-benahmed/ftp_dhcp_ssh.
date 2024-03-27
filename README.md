# DHCP, DNS, SFTP et SSH

Configuration d'un environnement réseau virtuel à l'aide de deux
machines virtuelles Debian, en mettant en place un serveur
DHCP et DNS sur la première machine, ainsi qu'un
serveur-client SFTP (proFTPd) avec SSH sur la seconde machine.

# Installation (iso) et Mise à jour

Installation à partir d'un iso debian 12.5-amd64.
Puis on procède à la mise à jour des paquets en mode root.
Par la suite on s'offre le luxe d'installer la commande sudo et d'ajouter l'utilisateur au groupe sudo qui s'effectue en faisant un nano du fichier etc/group puis en ajoutant le user a la fin de la ligne :
```bash
sudo:x:X:User
```
Enfin il suffit de faire exit puis connecter à nouveau, pour que les modifs soient prises en compte par debian.

# Installation du Serveur FTP et SSH :

```bash
sudo apt install proftpd
```
```bash
# Installation de la commande ftp
sudo apt install ftp
```
Concernant le serveur SSH il est déjà (installé durant le processus d'installation de debian)
mais si cela n'a pas été fait, il suffit de l'installer avec la commande:
```bash  
sudo apt install openssh-server
```

Concernant la configuration du serveur Ftp voici comment j'ai procédé.

1. Aller dans le fichier de configuration proftpd.conf qui se situe dans etc/proftpd/proftpd.conf .
```bash
cd ../../etc/proftpd
```
Puis,
```bash
# Sans le sudo vous n'aurez pas le droit de sauvegarder les modifs
sudo nano proftpd.conf
```

2. Modifier les lignes suivantes:
```bash
# Nom du serveur
ServerName:"Nom_serveur"
# Evite que les utilisateurs du serveur FTP ne remonte dans les dossiers du système.
DefaultRoot ~
# Nom de compte de l'utilisateur pour la connection et le nom de groupe(pour notre cas il n'est pas pertinent d'en ajouter un)
User Nom_user
Group nogroup
```

3. Configuration SFTP:
Ouvrons d'abord le fichier de configuration sshd_config
```bash
cd  
sudo nano ../../etc/ssh/sshd_config
```
Ensuite mettez en commentaire la ligne (si elle existe):
```bash
Subsystem sftp /usr/lib/openssh/sftp-server
```
et ajouter:
```bash
Subsystem sftp internal-sftp
```
Lorsqu'un utilisateur se connecte au serveur SSH et demande l'accès SFTP (Secure File Transfer Protocol), le serveur traitera en interne la session SFTP en utilisant son propre serveur SFTP intégré plutôt que de compter sur un logiciel de serveur SFTP externe.
Ce qui ajoute une couche de sécurité supplementaire.

Une fois ceci fait nous allons décommander la partie suivantes:
```bash
Match Group sftpusers
        #On restreint les utilisateurs à leur propre répertoire /home ce qui limite la mobilité des utilisateurs:
        ChrootDirectory %h
        # Le serveur SSH ne permettra pas l'affichage graphique des applications sur le client local. Cette fonctionnalité est souvent désactivée par défaut pour des raisons de sécurité, car elle permettrait à un utilisateur distant d'afficher des fenêtres graphiques sur le système local, ce qui peut potentiellement être exploité pour des attaques:
        X11Forwarding no
        # On bloque la capacité des utilisateurs SSH à ouvrir des tunnels TCP depuis le serveur vers d'autres serveurs ou services accessibles depuis le serveur distant:
        AllowTcpForwarding no
        # (1)On limite les utilisateurs qui se connectent au serveur ssh à utilisation du protocole SFTP pour le transfert de fichiers, sans qu'ils puissent exécuter des commandes sur le serveur:
        ForceCommand internal-sftp
```
(1).
-Lorsqu'un utilisateur se connecte au serveur SSH avec cette directive activée, le serveur SSH ne lui permettra pas d'exécuter des commandes sur le système distant. Au lieu de cela, il restreindra automatiquement l'utilisateur à un environnement où il peut uniquement transférer des fichiers en utilisant le protocole SFTP.

-Cela signifie que l'utilisateur n'aura pas accès à un shell interactif où il peut exécuter des commandes système ou naviguer dans le système de fichiers du serveur distant comme il le ferait normalement avec une connexion SSH normale.

Maintenant Sauvegarder le fichier (Ctrl + X), puis relancer le serveur ssh pour appliquer le changement:
```bash
sudo systemctl restart sshd

```

Nous avons fini la configuration de notre serveur SFTP, il nous reste qu'ajouter notre utilisateur dans le groupe _ssh :
```bash
cd
sudo nano ../../etc/group
```
Puis
ajouter le user à la fin de la ligne :
```bash
_ssh:x:X:Nom_de_utilisateur
```

4.Test de connexion
```bash
sftp Nom_de_utilisateur@serveur_ip
```
puis votre mot de passe utilisateur.

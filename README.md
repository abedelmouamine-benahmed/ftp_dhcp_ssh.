# DHCP, DNS, FTP et SSH

Configuration d'un environnement réseau virtuel à l'aide de deux
machines virtuelles Debian, en mettant en place un serveur
DHCP et DNS sur la première machine, ainsi qu'un
serveur-client SFTP (proFTPd) avec SSH sur la seconde machine.

# Installation (iso) et Mise à jour

Installation à partir d'un iso debian 12.5-amd64
Puis on procède à la mise à jours des paquets en mode root.
Par la suite on s'offre le luxe d'installer la commande sudo et d'ajouter l'utilisateur au groupe sudo qui s'effectue en faisant un nano du fichier etc/group puis en ajoutant le user a la fin de la ligne sudo:x:X:User.
Enfin il suffit de faire exit puis connecter à nouveau pour que les modifs soient prises en compte par debian 

# Installation du Serveur FTP et SSH :

'''shell
sudo apt install proftpd
'''
Concernant le serveur SSH il est déjà (installé durant le processus d'installation de debian)
mais si cela n'a pas été fait, il suffit de l'installer avec la commande:
'''shell 
sudo apt install openssh-server
'''
Concernant la configuration du serveur Ftp voici comment j'ai procédé:
1.Aller dans le fichier de configuration proftpd.conf qui se situe dans etc/proftpd/proftpd.conf
'''shell
cd ../../etc/proftpd
'''
Puis 
'''shell
nano proftpd.conf
'''
2.

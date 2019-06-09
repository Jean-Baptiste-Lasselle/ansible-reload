# Ansible-reload
It's not that I miss metallica at all, just that I needed a personal bare-metal reference for Ansible State of the Art


So this survey is doomed to become a gitbook, which will deal with all I konw about Ansible, as a next-step prep.



## Ansible Configuration 

Ansible (son exécution) est configuré(e) via : 

* Les variables d'environnement définies pour le processus d'exécution de la commande `ansible`.
* Un fichier de configuration au format `ini`, nommé `ansible.cfg`. 
* À chacune de ses exécutions, `ansible` recherche ce fichier de configuration sur le système de fichiers, jusqu'à désigner un et un seul fichier, comme le fichier de configuration. 
* Si `ansible` n'arrive pas à ainsi sélectionner un tel fichier de configuration, l'exécution `ansible` se terminera son exécution avec un code de retour d'erreur. 
* À son installation, ansible provisionne un fichier de configuration à l'emplacement `/etc/ansible/ansible.cfg`, afin de fournir un fichier de configuration "par défaut", avec l'esprit _zero configuration / all defaults_ . 
* Enfin, à chacune de ses exécutions, `ansible` sélectionne le fichier de configuration actif avec les règles suivantes :  
  * À son exécution l'exécutable `ansible` vérifie si la variable d'environnement `ANSIBLE_CONFIG` est définie, si elle l'est, et que sa valauer est le chemin d'un fichier existant, alors ce fichier est utilisé comme  fichier de configuration pour cette exécution.
  * À son exécution l'exécutable `ansible`, si la variable d'environnement `ANSIBLE_CONFIG` n'est pas définie, recherche ans le répertoire courant, un fichier `./ansible.cfg`, s'il le trouve, il est utilisé comme  fichier de configuration pour cette exécution.
  * À son exécution l'exécutable `ansible`, si la variable d'environnement `ANSIBLE_CONFIG` n'est pas définie, et que le fichier `$(pwd)/ansible.cfg` n'existe pas, pour chaque utilisateur d'un hôte `UNIX / GNU linux`, le fichier `~/.ansible.cfg` s'il existe, est utilisé comme  fichier de configuration pour cette exécution, et surcharge le fichier `/etc/ansible/ansible.cfg`. 
  * À son exécution l'exécutable `ansible`, si la variable d'environnement `ANSIBLE_CONFIG` n'est pas définie, et que le fichier `$(pwd)/ansible.cfg` n'existe pas, pour chaque utilisateur d'un hôte `UNIX / GNU linux`, le fichier `/etc/ansible/ansible.cfg`, s'il existe, est utilisé comme  fichier de configuration pour cette exécution. Il s'agit donc d'une configuration s'appliquant à tous les utilisateurs d'un hôte `UNIX / GNU linux`: pour les paramètres de configuration pour lesquels cela est nécessaire, chaque utilisateur le surcharge à l'aide du fichier `~/.ansible.cfg`.
* des options de type GNU, `--mon-option-d-execution unev@leur`, utilisables à l'invocation de l'exécutable `ansible`, qui surchargent toutes les configurations par fichier.
* _**Chemins Fichiers/Répertoires Dans la configuration**_ Pour chaque paramètre de configuration, lorsque la valeur de celui-ci est un chemin de fichier ou de répertoire :
  * Ce chemin peut être précisé avec un chemin relatif
  * Le chemin est relatif au répertoire dans lequel se trouve le fichier de configuration actif pour l'exécution `ansible` en cours.


### Recommandantions quant au fichier de configuration

**No `{{ CWD}}`**

Son utilisation est un trou de sécurité, selon la doc officielle elle-même : je bannis son utilisation par _`Lint`_ au sein de tous les repos playbookset roles.

> 
> #### Avoiding security risks with ansible.cfg in the current directory
> 
> If Ansible were to load ansible.cfg from a world-writable current working directory, it would create a serious security risk. Another user could place their own config file there, designed to make Ansible run malicious code both locally and remotely, possibly with elevated privileges. For this reason, Ansible will not automatically load a config file from the current working directory if the directory is world-writable.
> 
> If you depend on using Ansible with a config file in the current working directory, the best way to avoid this problem is to restrict access to your Ansible directories to particular user(s) and/or group(s). If your Ansible directories live on a filesystem which has to emulate Unix permissions, like Vagrant or Windows Subsystem for Linux (WSL), you may, at first, not know how you can fix this as chmod, chown, and chgrp might not work there. In most of those cases, the correct fix is to modify the mount options of the filesystem so the files and directories are readable and writable by the users and groups running Ansible but closed to others. For more details on the correct settings, see:
> 
> * for Vagrant, Jeremy Kendall’s blog post covers synced folder permissions.
> * for WSL, the WSL docs and this Microsoft blog post cover mount options.
> 
> If you absolutely depend on storing your Ansible config in a world-writable current working directory, you can explicitly specify the config file via the ANSIBLE_CONFIG environment variable. Please take appropriate steps to mitigate the security concerns above before doing so.
> 

Moralité : 

* Ne jamais utiliser un fichier de configuration `$(pwd)/ansible.cfg` dans le répertoire courant, sauf contrainte absolue. 
* Si obligation d'utiliser un fichier de configuration dans le répertoire courant, alors ne jamais le faire qu'avec une gestion des droits d'exécution et écriture drastique, sur le répertoire courant : 
  * Un groupe d'utilisateurs Linux unique et bien déterminé, dans lequel sont ajoutés uniquement les personnes autorisées à opérer avec les playbooks concernés.
  * Les autres utilisateurs doivent se voir l'interdiction complète, d'écriture, lecture (IMPORTANT à cause des secrets), exécution.




Au-delà de la version `2.4` d'Ansible, l'utilitaire [`ansible-config`](https://docs.ansible.com/ansible/latest/cli/ansible-config.html#ansible-config) permet de "tout savoir" sur la configuration (un peu comme `docker-compose config`, masi avec beaucoup plus d'informations) : 

* pour voir la configuration de la configuration : 
```bash
ansible-configuration list
```
* Pour voir la configuration effective (un peu comme `docker-compose config`), ou seulement les changements de celle-ci : 

```bash
ansible-configuration dump
ansible-configuration dump --only-changed
```
* lorsque l'on exécute la commande `ansible-configuration`, la variable d'environnement `ANSIBLE_CONFIG` permet (tout comme pour la commande principale `ansible`) de définir le chemin du fichier de configuration Ansible : 

```bash
export ANSIBLE_CONFIG=/etc/ansible/ansible.cfg
mkdir -p /etc/justtosay
cat /etc/ansible/ansible.cfg > /etc/justtosay/tangible.cfg
export ANSIBLE_CONFIG=/etc/justtosay/tangible.cfg
export ANSIBLE_CONFIG=~/.ansible.cfg
ansible-configuration list
```




## Ansible Execution

Ansible est un outil qui permet, à chacune de ses exécutions, de réaliser un certain nombre d'opérations, sur un parc quelconque de machines.

Le parc de machines, est spécifié par configuration, et appelé inventaire (_inventory_). Le plus souvent, cet inventaire est spécifié dans un fichier de configuration dédié à cet effet, courramment nommé `./inventory` ou `/etc/ansible/hosts` `./ma.petite.recette.inventory`.

L'ensemble des opérations peut-être spécifié, par configuration et/ou spécifications d'options d'invocation.

Utiliser Ansible est simple, il s'agit :

* d'exécuter un exécutable de nom de fichier `ansible`, sur une machine `Machine1`,
* Ansible utilise alors par défaut `OpenSSH`, afin de se connecter à chacune des machines de l'inventaire, puis réaliser les opérations souhaitées sur chaque machine.


### Authentifications au cours d'une exécution

* Ansible utilise `SSH` pour opérer les machines d'un inventaire, donc s'authentifie au serveur `SSH` de chaque Machine de l'inventaire. La méthode d'authentification recommandée est celle faisant usage de clés privés / clés publiques, mais si l'on utilise l'option `ansible --ask-pass`, Ansible permettra une Authentification de type _username / password_. Le must de l'authentification aujorud'hui, est d'utiliser des clés SSH signées par une Autorité, comme `HashiCorp Vault`, afin d'avoir des clés `SSH` bien gérées / supervisées (HashiCorp Vault Logs), et qui expirent.
* Au sein de chaque machine, un fois connecté via SSH, et donc authentifié comme étant un utilisateur `OS` précis, disons `bernard22`,  Ansible peut exécuter des commandes qui nécessitent une escalade de droits de `bernard22`. On précise à Ansible d'exécuter les opérations planifiées, avec une escalade de droits, à l'aide de l'option `ansible --become`, et on précise que la méthode d'authentification à l'utilitaire d'escalade de droits, est de type _username / password_ en utilisant conjointement l'option `--ask-become-pass` (ce qui donen la combinaison `ansible --become --ask-become-pass`) : 
  * Au sein des OS tels que les distributions `Debian` / `Ubuntu` / `CentOS`, un exécutable qui permet à un utilisateur de réaliser une telle escalade de droits, est le bien connu `sudo` : en utilisant l'option `ansible --ask-become-pass`, Ansible invoquera donc `sudo` sur les mahcines de l'inventaire exploitées par une distribution `GNU / Linux` `Debian`, `Ubuntu`, ou `CentOS`. 
  * Cependant Ansible a la possibilité de faire appel à d'autres utilitaires d'escalade de droits, commu `su` (switch user), ou encore `pbrun` (Powerbroker), `pfexec`, `dzdo` (Centrify), etc... . L'option `ansible --ask-become-pass` s'est appelée dans le passé `--ask-sudo-pass`, mais a été renommée `ansible --ask-become-pass`, depuis l'extension à ces autres outils d'escalade de droits.
  * Enfin (cf. https://docs.ansible.com/ansible/2.4/become.html), précisons tout de suite les options d'exécution, qui permettent de configurer le comportement d'Ansible pour l'escalade de droits (et donc utilisées dès lors que l'option `--become` est utilisée, peu importe si `--ask-become-pass` est utilisée conjointement ou non) : 
  
  | Command line options         | WTF                                                                              |
  | -----------------------------| ---------------------------------------------------------------------------------|
  | `--ask-become-pass`, `-K`    |     _ask for privilege escalation password, does not imply become will be used_  |
  | `--become`, `-b`             |     _run operations with become (no password implied)_                           |
  | `--become-method=BECOME_METHOD`      |   _privilege escalation method to use (default=sudo), valid choices: [ `sudo`,  `su`, `pbrun`, `pfexec`, `doas`, `dzdo`, `ksu` ]_ |
  | `--become-user=BECOME_USER`      |     _run operations as this user (default=root), does not imply –become/-b_  |
    



## ANNEXES

### ANNEXE A : Configurer Ansible pour qu'il fasse usage de `scp` en lieu et place de `SFTP`

* Cela se fait daans le fichier de configuration `/etc/ansible/ansible.cfg`.
* Dans ce repo un exemple de [fichier de configuration `Ansible`](ttps://github.com/Jean-Baptiste-Lasselle/ansible-reload/blob/master/ansible.stock.cfg), et dans ce fichier de configuration d'Ansible la section de configuration est explicite : 

```ini

# Control the mechanism for transferring files (old)
#   * smart = try sftp and then try scp [default]
#   * True = use scp only
#   * False = use sftp only
#scp_if_ssh = smart
# Et moi j'utilise systématiquement le mode 'smart'
```
* cccc

### ANNEXE B : Quotes from the official doc

Source : https://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html


* Pour celui-là, il faut faire un POC avec un Bastion SSH, un jump configuré dans le cycle IAAC ops avec le fichier `~/.ssh/config` :  
> By default, Ansible will try to use native OpenSSH for remote communication when possible. This enables ControlPersist (a performance feature), Kerberos, and options in `~/.ssh/config` such as Jump Host setup.
* Pour celui-là, je vais faireun POC montrant qu'il vaut mieux utiliser les SSH signed keys gérées par `HashiCorp Vault`, au lieu des _Kerberized Keys_ évoquées par la doc ansible : 
>  If you wish to use features like Kerberized SSH and more, consider using Fedora, macOS, or Ubuntu as your control machine until a newer version of OpenSSH is available for your platform.
* Pour celui-là, on fera un test avec `HashiCorp Packer` gérant des images `CentOS 6`, `CentOS 7`, et `Fedora 2x`, pour déterminer à partir de quelle version on a ue version assez récente d' `OpenSSH` pour permettre l'utilisation de  `ControlPersist`, et en avoir l'amélioration de performances apportée par `ControlPersist` : 
>  However, when using Enterprise Linux 6 operating systems as the control machine (Red Hat Enterprise Linux and derivatives such as CentOS), the version of OpenSSH may be too old to support ControlPersist. On these operating systems, Ansible will fallback into using a high-quality Python implementation of OpenSSH called ‘paramiko’.
* Pour celui-là, je vais mettre ne oeuvre un exemple montrant comment configurer Ansible pour qu'il fasse usage de `scp` en lieu et place de `SFTP` : 
> Occasionally you’ll encounter a device that doesn’t support SFTP. This is rare, but should it occur, you can switch to SCP mode in Configuring Ansible.

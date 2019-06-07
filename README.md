# Ansible-reload
It's not that I miss metallica at all, just that I needed a personal bare-metal reference for Ansible State of the Art


So this survey is doomed to become a gitbook, which will deal with all I konw about Ansible, as a next-step prep.

# Quotes from the official doc

Source : https://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html


* Pour celui-là, il faut faire un POC avec un Bastion SSH, un jump configuré dans le cycle IAAC ops avec le fichier `~/.ssh/config` :  
> By default, Ansible will try to use native OpenSSH for remote communication when possible. This enables ControlPersist (a performance feature), Kerberos, and options in `~/.ssh/config` such as Jump Host setup.
* Pour celui-là, je vais faireun POC montrant qu'il vaut mieux utiliser les SSH signed keys gérées par `HashiCorp Vault`, au lieu des _Kerberized Keys_ évoquées par la doc ansible : 
>  If you wish to use features like Kerberized SSH and more, consider using Fedora, macOS, or Ubuntu as your control machine until a newer version of OpenSSH is available for your platform.
* Pour celui-là, on fera un test avec `HashiCorp Packer` gérant des images `CentOS 6`, `CentOS 7`, et `Fedora 2x`, pour déterminer à partir de quelle version on a ue version assez récente d' `OpenSSH` pour permettre l'utilisation de  `ControlPersist`, et en avoir l'amélioration de performances apportée par `ControlPersist` : 
>  However, when using Enterprise Linux 6 operating systems as the control machine (Red Hat Enterprise Linux and derivatives such as CentOS), the version of OpenSSH may be too old to support ControlPersist. On these operating systems, Ansible will fallback into using a high-quality Python implementation of OpenSSH called ‘paramiko’.
* Pour celui-là, je vais mettre ne oeuvre un exemple montrant comment configurer Ansible pour qu'il fasse usage de `scp` en lieu et place de `SFTP` : 
> Occasionally you’ll encounter a device that doesn’t support SFTP. This is rare, but should it occur, you can switch to SCP mode in Configuring Ansible.


## Ansible Configuration 

Ansible (son exécution) est configuré(e) via : 

* Les variables d'environnement définies pour le processus d'exécution de la commande `ansible`.
* Un fichier de configuration au format `ini`, nommé `ansible.cfg`. Les fichiers `/etc/ansible/ansible.cfg` et  `~/.ansible.cfg`, sont recherchés, et : 
  * À son exécution l'exécutable `ansible` vérifie si la variable d'environnement `ANSIBLE_CONFIG` est définie, si elle l'est, et que sa valauer est le chemin d'un fichier existant, alors ce fichier est utilisé comme  fichier de configuration pour cette exécution.
  * À son exécution l'exécutable `ansible`, si la variable d'environnement `ANSIBLE_CONFIG` n'est pas définie, recherche ans le répertoire courant, un fichier `./ansible.cfg`, s'il le trouve, il est utilisé comme  fichier de configuration pour cette exécution.
  * À son exécution l'exécutable `ansible`, si la variable d'environnement `ANSIBLE_CONFIG` n'est pas définie, et que le fichier `$(pwd)/ansible.cfg` n'existe pas, pour chaque utilisateur d'un hôte `UNIX / GNU linux`, le fichier `~/.ansible.cfg` s'il existe, est utilisé comme  fichier de configuration pour cette exécution, et surcharge le fichier `/etc/ansible/ansible.cfg`. 
  * À son exécution l'exécutable `ansible`, si la variable d'environnement `ANSIBLE_CONFIG` n'est pas définie, et que le fichier `$(pwd)/ansible.cfg` n'existe pas, pour chaque utilisateur d'un hôte `UNIX / GNU linux`, le fichier `/etc/ansible/ansible.cfg`, s'il existe, est utilisé comme  fichier de configuration pour cette exécution. Il s'agit donc d'une configuration s'appliquant à tous les utilisateurs d'un hôte `UNIX / GNU linux`: pour les paramètres de configuration pour lesquels cela est nécessaire, chaque utilisateur le surcharge à l'aide du fichier `~/.ansible.cfg`.
* des options de type GNU, `--mon-option-d-execution unev@leur`, utilisables à l'invocation de l'exécutable `ansible`, qui surchargent toutes les configurations par fichier.
* _**Chemins Fichiers/Répertoires Dans la configuration**_ Pour chaque paramètre de configuration, lorsque la valeur de celui-ci est un chemin de fichier ou de répertoire :
  * Ce chemin peut être précisé avec un chemin relatif
  * Le chemin est relatif au répertoire dans lequel se trouve le fichier de configuration actif pour l'exécution `ansible` en cours.


### Recommandantions quant au ficheir de configuration

**No `{{ CWD}}`**

Son utilisation est un trou de sécurité, slon la doc offficielle, et est non-recommandée : je bannis son utilisation par `Lint`

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






## Points of view

### Configurer Ansible pour qu'il fasse usage de `scp` en lieu et place de `SFTP`

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

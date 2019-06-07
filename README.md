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

Ansible est configuré via : 

* Les variables d'environnement définies pour le processus d'exécution de la commande `ansible`.
* Un fichier de configuration au format `ini`, nommé `ansible.cfg`.
* des options de type GNU, `--mon-option-d-execution unev@leur`, utilisables à l'invocation de l'exécutable `ansible`

Au-delà de la version `2.4` d'Ansible, l'utilitaire `ansible-config` pemet de "tout savoir" sur la configuration (un peu comme `docker-compose config`, masi avec beaucoup plus d'informations) : 

* ccc
* cccc





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

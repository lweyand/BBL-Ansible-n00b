# Préparation:

* Effacer la VM et la recréer
    ```
    vagrant destroy
    vagrant up
    ```
* activer le virtual env
    ```
    source ~/.virtualenv/ansible2.7/bin/activate
    ```

# Ansible

## Présentation Ansible
``
Ansible est une plate-forme logicielle libre pour la configuration et la gestion des ordinateurs. 
Elle combine le déploiement de logiciels multi-nœuds, l'exécution des tâches ad-hoc, et la gestion de configuration. 
Elle gère les différents nœuds à travers SSH et ne nécessite l'installation d'aucun logiciel supplémentaire sur ceux-ci. 
Les modules communiquent via la sortie standard en notation JSON et peuvent être écrits dans n'importe quel langage de 
programmation. Le système utilise YAML pour exprimer des descriptions réutilisables de systèmes, appelés playbook2.
``

[source](https://fr.wikipedia.org/wiki/Ansible_(logiciel\))

## Idempotence

``
En mathématiques et en informatique, l'idempotence signifie qu'une opération a le même effet qu'on l'applique une ou plusieurs fois.
``

[source](https://fr.wikipedia.org/wiki/Idempotence)

Rejouer plusieurs fois le même playbook, doit toujours aboutir au même état de la machine.

Ex. de cas à gérer:
* créer un utilisateur sous linux, second ajout du même utilisateur provoque un plantage

## Arborescence:

[Ansible directory layout](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#directory-layout)

## pattern/anti pattern

* pattern: installation d'un serveur from scratch
* anti pattern: mise à jour d'une application: les mises à jours mêmes mineurs ne sont pas idempotent. chaque mise à jour est différente de la précédente.


# Hands on!

Le but est d'écrire un playbook qui va installer nginx dans une VM Debian, et on ca y mettre notre page d'index.

## step_00

Initialisation du projet avec le Vagrantfile et le playbook fait un hello au monde en local

## step_01: se connecter à la VM et faire coucou
* Mettre en place l'inventaire
* Créer le répertoire **inventory/vagrant**
* Créer le fichier **hosts**
* Ajouter les infos pour vagrant dans le  fichier hosts (avec le groupe web)
```
[web]
127.0.0.1 ansible_port=2222 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/default/virtualbox/private_key
```

## step_02: déplacer les infos dans les host_vars
* Créer le répertoire **inventory/vagrant/host_vars**
* Y créer le fichier **127.0.0.1.yml** et y mettre les données du hosts:
```
ansible_port: 2222
ansible_user: vagrant
ansible_ssh_private_key_file: .vagrant/machines/default/virtualbox/private_key
```
* Aller sur la VM Virtualbox pour s'y connecter et constater le mapping du clavier

## step_03: remap du clavier
Pour changer le layout du clavier, il faut modifier la valeur de la propriété *XKBLAYOUT* dans le fichier */etc/default/keyboard*.

* Changer la valeur de XKBLAYOUT pour la valeur 'fr'
```
    - name: change keyboard layout
      become: yes
      lineinfile:
        path: "/etc/default/keyboard"
        regexp: '^XKBLAYOUT='
        line: 'XKBLAYOUT="fr"'
```
* Aller sur la VM et constater que rien n'a changé
* Un redémarrage est nécessaire: ajouter le reboot dans le playbook
```
    - name: reboot
      become: yes
      reboot:
        msg: "Ansible reboot!"
```
* Exécuter le playbook, la VM est redémarrée, et le clavier est bon.
* Relancer le playbook constater que le reboot se fait: non idempotent
* Il faut ajouter un test, pour ne pas redémarrer si la valeur du fichier n'a pas été changé. Pour cela, il faut
enregistrer la sortie de la modification du fichier:
```
      register: kbd_changed
    - name: debug
      debug:
        var: kbd_changed
```
* Lancer le script pour étudier le json retourné
* Ajouter le test pour ne pas redémarrer sur le fichier *keyboard* n'est pas modifié:
```
      when: kbd_changed.changed == true
```
* Relancer le test et constater que la vm ne reboot plus.


## step_04: Installer nginx
Sous debian la commande pour installer un paquet est *apt_get install*.

Ceci a été traduit par Ansible:
* Commande pour installer package sur **Debian**: apt
```
    - name: install nginx
      become: yes
      apt:
        name: nginx
        state: present
```
* Se connecter à l'url: [http://localhost:8080/](http://localhost:8080/)

C'est bien, mais on veut notre homepage.
* A la racine du playbook, créer le répertoire **files**
* Dans ce répertoire, y créer un fichier html nommé **our_index.html**:
```
<html>
<body>
<h1>It's our page</h1>
</body>
</html>
```
* Pour déployer le fichier, il faut utiliser la tâche **copy**:
```
    - name: deploy index file
      become: yes
      copy:
        src: "files/our_index.html"
        dest: "/var/www/html/index.html"
```

## step_05: refactoring: extraction des variables

* Créer le répertoire **inventory/group_vars**
* Créer le fichier **web.yml** pour y mettre les variables liées au groupe
```
kbd_config_file: /etc/default/keyboard

home_page_src: files/our_index.html
home_page_dest: /var/www/html/index.html
```
* mettre à jour le playbook avec ces variables

## step_06: refactoring: déplacement des variables dans un répertoire dédié au groupe
* Créer le répertoire **inventory/group_vars/web**
* Créer le fichier **all.yml** dans ce répertoire

**all.yml** est toujours appelé par Ansible, et les fichiers sont pris par ordre alphabétique -> ceci peut avoir un impacte
        si des variables d'un fichier *c* ont besoin de variables d'un ficher *a*

* Lancer le script

## step_07: refactoring: séparation des variables par fonctionnalités
Pour plus de lisibilité, séparer les variables de inventory/group_vars/web/all.yml dans 2 fichiers:
**inventory/group_vars/web/nginx.yml** et **inventory/group_vars/web/os.yml**, et supprimer all.yml

## step_08: refactoring: faire des includes dans le playbook
Toujours de la lisibilité, les tâches (tasks) du playbook, vont être réparties dans des fichiers dédiés.
A la racine du projet, créé un fichier **os.yml**, et un **nginx.yml**.
Y déplacer les *tasks* qui leur correspondent, et les inclures dans le fichier playbook.yml à l'aide du module **include_tasks**.
```
    - name: Modify OS
      include_tasks:
        file: os.yml
    - name: Install and configure nginx
      include_tasks:
        file: nginx.yml
```

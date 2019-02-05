# Préparaion:
* Effacer la VM et la recréer
    ```
    vagrant destroy
    vagrant up
    ```
* activer le virtual env
    ```
    source ~/.virtualenv/ansible2.7/bin/activate
    ```


* présentation Ansible
* Immutabilité
* Arborescence
  * https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#directory-layout
* pattern/anti pattern


* écriture playbook:
    * Installer Nginx
    * changement du layout du fichier
    * ajouter une page html

* réfactorer
  * variables dans un fichier
  * variables dans un répertoire avec vars par type de fonction
  * playbook
    * extraire la partie conf OS
    * extraire la partie web 

créer le répertoire inventaire
créer le fichier hosts
créer le fichier playbook
lancer vagrant
constater le pb de clavier
modifier playbook pour changer le layout

# Hands on!

## step_00

Initialisation du projet avec le Vagrantfile

## step 01 se connecter à la VM
Mettre en place l'inventaire
création du répertoire inventory/vagrant
création du fichier hosts avec le groupe web
Ajouter les infos pour vagrant dans le  fichier hosts
```
[web]
127.0.0.1 ansible_port=2222 ansible_user=vagrant ansible_ssh_private_key_file: .vagrant/machines/default/virtualbox/private_key
```

## step 02 les host_vars
Création du fichier 127.0.0.1.yml et y mettre les données du hosts
Aller sur la VM pour se connecter et constater le mapping

## step 03 remap du clavier
Changer la valeur de XKBLAYOUT pour la valeur 'fr'
```
    - name: change keyboard layout
      become: yes
      lineinfile:
        path: "/etc/default/keyboard"
        regexp: '^XKBLAYOUT='
        line: 'XKBLAYOUT="fr"'
```
aller sur la VM et constater que rien n'a changé
Reboot nécessaire: ajouter le reboot dans le playbook
```
    - name: reboot
      become: yes
      reboot:
        msg: "Ansible reboot!"
```
relancer le playbook constater que le reboot se fait: non idempotent
ajouter le test, pour évaluer la valeur:
```
      register: kbd_changed
    - name: debug
      debug:
        var: kbd_changed
```
Ajouter le test pour invalider le reboot
```
      when: kbd_changed.changed == true
```
Relancer le test et constater que la vm ne reboot plus.


## step 04 Installer nginx
Commande pour installer package sur debian: apt
```
    - name: install nginx
      become: yes
      apt:
        name: nginx
        state: present
```
Se connecter à l'url: http://localhost:8080/

Changer la homepage
Créer le répertoire files
Créer un fichier html dans ce répertoire
```
<html>
<body>
<h1>It's our page</h1>
</body>
</html>
```
Déployer le fichier:
```
    - name: deploy index file
      become: yes
      copy:
        src: "files/our_index.html"
        dest: "/var/www/html/index.html"
```

## step 05 refactoring: extraction des variables

Créer le répertoire inventory/group_vars pour y mettre les variables liées au groupe
Créer le fichier web.yml
Créer les variables:
```
kbd_config_file: /etc/default/keyboard

home_page_src: files/our_index.html
home_page_dest: /var/www/html/index.html
```

## step 06 refactoring: déplacement des variables dans un répertoire dédié au groupe
Créer le répertoire web, et le fichier all.yml
all.yml est toujours appelé par Ansible, et les fichiers sont pris par ordre alphabétique -> ceci peut avoir un impacte
        si des variables d'un fichier *c* ont besoin de variables d'un ficher *a*
Lancer le script

## step 07 refactoring: séparation des variables par fonctionnalités
Séparer les variables de all dans 2 fichier: nginx.yml et os.yml

## setp 08 refactoring: faire des includes dans le playbook

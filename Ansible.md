* présentation Ansible
* Immutabilité
* Arborescence
  * https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#directory-layout
* pattern/anti pattern

* Effacer la VM et la recréer
    ```
    vagrant destroy
    vagrant up
    ```
* activer le virtual env
    ```
    source ~/.virtualenv/ansible2.7/bin/activate
    ```

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


## step_00

Initialisation du projet avec le Vagrantfile

## step 01 se connecter à la VM
Mettre en place l'inventaire
création du répertoire inventory/vagrant
création du fichier hosts avec le groupe web
Ajouter les infos pour vagrant dans hosts
```
127.0.0.1 ansible_port=2222 ansible_user=vagrant ansible_ssh_private_key_file: .vagrant/machines/default/virtualbox/private_key
```

## step 02 les host_vars

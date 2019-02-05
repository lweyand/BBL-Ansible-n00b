* présentation Ansible
* Immutabilité
* Arborescence
  * https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#directory-layout
* pattern/anti pattern

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
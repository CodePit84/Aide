Les 2 clés SSH (privée et publique) sont sous linux dans le dossier ".ssh"

Si vous n'avez pas générer encore de clé alors taper :
```ssh-keygen -t ed25519 - C "votreemail@exemple.com```

sinon la clé privée n'a pas d'extension et la clé publique a l'extension ".pub"

```cd .ssh``` à la racine

```ls - lah``` pour affichier tous les fichiers et dossiers (même cachés)

taper : ```cat id_rsa.pub``` par exemple (id_rsa.pub étant le nom de votre clé publique)

et copiez la dans GitHub ou tout service vous demandant une clé ssh !

ex GitHub : 
SSH and GPG Keys
  check out our to "generating SSH keys"
    Generating a new SSH key and adding...
ensuite copié votre clé publique et l'ajouter (New SSH key)


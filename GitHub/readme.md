## Commit et push :

```git status```

```git add .```

```git status```

```git commit -m "message du commit```

```git push origin main``` ou ```git push origin master```


---

## Github: ajouter un projet déjà créé

On a l’habitude de cloner des projets qui sont déjà sur Github… voici maintenant comment mettre en ligne un projet créé localement. (version PC)
Mode d’emploi

(ne tapez pas les « $ » ou les « > » en début de commande, ils sont là pour indiquer un début de ligne)

Ouvrir une ligne de commande (Win+R et cmd)
naviguer et se mettre à l’intérieur du répertoire du projet
```cd..``` pour remonter
```cd nomDuDossier``` pour descendre
ls ou dir pour afficher la liste du contenu

```$ git init``` pour que git initialise ce dossier en projet git. Cela ajoute les trois arbres nécessaire au suivi de version:
le répertoire de travail (la version actuelle): working directory
l’index : la liste des choses qui sont préparées (la commande pour ajouter au « stage » est « add« )
le head qui correspond a ce qui a été « commit« , c’est à dire vraiment envoyé comme version

```$ git add .``` est la commande qui « stage » tous les documents, qui les ajoute à l’index. Pour info, on peut sortir de la pile un document avec la commande ```$ git reset HEAD votreFichier```

```$ git commit -m "mon premier commit"``` permet de transférer les fichiers en attente dans l’index, vers le Head. Le message permet de savoir à quoi cela correspond.

Créez le projet sur Github.com, en ne mettant PAS de Readme. Il vaut mieux l’ajouter après, une fois que les fichiers ont été uploader, pour éviter tout conflit.

Copier le lien vers le répertoire, situé généralement en haut du projet dans une boite bien visible: le lien se termine par « .git »

```$ git remote add origin leLienCopié``` va désigner le répertoire distant comme cible du projet.

Répondez aux information soit dans la boite de dialogue qui s’affiche, soit dans la suite de la commande.

```$ git remote -v``` J’avoue que je n’ai pas vraiment compris, il me semble que c’est juste la vérification que les choses fonctionnent dans les deux sens.

```$ git push origin master``` est la commande qui va finalement envoyer les fichiers

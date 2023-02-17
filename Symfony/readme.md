Symfony commande :

```symfony server:start -d``` ou ```symfony serve -d```

```symfony server:stop```

```php bin/console make:entity``` ou ```symfony console make:entity```
==> créer une Entity

```php bin/console d:d:d --force```
==> on drop la base de données (suppr)

```php bin/console d:d:c```
==> on créé la base de données (ou recréé après une suppression)

```php bin/console d:m:m```
==> on push les migration de la bdd (migration migrate)

```php bin/console make:migration```
==> migration

```php bin/console d:f:l```
==> puger la base et la charger avec des fixtures

```php bin/console make:controller BlaBlaBlaController```
==> créer le controller BlaBlaBla

```php bin/console make:form```
==> créer un formulaire

```php bin/console make:user```
==> créer un utilisateur

```php bin/console make:entity User```
==> compléter une entité (par exemple ici User)

```php bin/console make:controller SecurityController```
==> créer un controller sécurité



---------
Pour update une base de données sans effacer sons contenu : (tuto symrecipe oublie d'une colonne et modif donc du controller) :

```php bin/console make:migration```

puis :

```php bin/console d:m:m```
---------

```php bin/console make:entity user```
==> mise en place de la relation entre 2 entitées (par exemple l'entity user et l'entity ingredient)

ingredients

relation

Ingredient

OneToMany

user

no

yes
-----------

```[notice] Migrating up to DoctrineMigrations\Version20221208081548
[error] Migration                                        (...)                                         
  An exception occurred while executing a query: SQLSTATE[23000]: Integrity constraint violation: 1452 Cannot add or update a child row: a foreign key const  
  raint fails (`symrecipe`.`#sql-38eb_127`, CONSTRAINT `FK_DA88B137A76ED395` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`))```
==> erreur à la suite d'un :
```php bin/console d:m:m```

c'est normal, quand il y a de nouvelles clés étrangères... donc il faut supprimer la bdd, et la recréer, on migre toutes les migrations, et éventuellement rajouter des fixtures...

```php bin/console d:d:d --force```

```php bin/console d:d:c```

```php bin/console d:m:m```

```php bin/console d:f:l```
------------

Erreur suite à un ```php bin/console d:m:m``` :
```[OK] Already at the latest version ("DoctrineMigrations\Version20221212085834")```
c'est que la dernière migration n'a pas été crée donc :
```php bin/console make:migration```
puis :
```php bin/console d:m:m```
-----------

ERREUR : 
```An exception occurred while executing a query: SQLSTATE[42S22]: Column not found: 1054 Unknown column 'r0_.image_name' in 'field list'```
==> dû à un oublie de migration donc :
```php bin/console make:migration```
==> puis comme d'hab :
```php bin/console d:m:m```
```php bin/console d:f:l```
-----------

ERREUR HTTP:500

(j'ai tout effacé pour relancer un clone de ma dernière sauvegarde GitHub qui fonctionnait...)

lorsque j'ai lancé VS Code j'ai eu le message :
(https://github.com/CodePit84/Aide/blob/main/Symfony/img/erreur1.png)
j'ai donc exécuté :
```composer require symfony/runtime```
pour obtenir le message suivant : (erreur database)
(https://github.com/CodePit84/Aide/blob/main/Symfony/img/erreur2.png)
J'ai donc ensuite copié mon dernier fichier (caché) ".env.dev.local" que j'avais toujours, et qui contenais :
```# DATABASE_URL="mysql://root:@127.0.0.1:3306/symrecipe?serverVersion=8"
DATABASE_URL="mysql://root:@127.0.0.1:3306/symrecipe?serverVersion=mariadb-10.4.24"```
et tout refonctionnait :)
------------------

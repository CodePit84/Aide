On va installer tout ça en --dev car on n'en a pas besoin en production...

1. On installe le composant Doctrine DataFixtures avec la commande :
``` 
composer require --dev orm-fixtures
```

2. On installe FakerPHP :
``` 
composer require fakerphp/faker
```

Vous pouvez effacer le fichier src/DataFixtures/AppFixtures.php

3. On va créer des utilisateurs projet) :

``` 
symfony console make:fixtures UserFixtures
```

Dans le fichier src/DataFixtures/UserFixtures.php



99. On va créer des mouvements (pour rester dans la suite de notre projet) :

``` 
symfony console make:fixtures MovementFixtures
```





``` 
symfony console doctrine:fixtures:load
```
ou
``` 
symfony console d:f:l
```

31:10 Nouvelle techno
#[ORM\JoinColumn(onDelete: 'CASCADE')]
symfony console make:migration
symfony console doctrine:migrations:migrate

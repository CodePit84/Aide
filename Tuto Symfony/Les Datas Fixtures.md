On va installer tout ça en --dev car on n'en a pas besoin en production...

1. On installe le composant Doctrine DataFixtures avec la commande :
``` 
composer require --dev orm-fixtures
```

2. On installe FakerPHP :
``` 
composer require fakerphp/faker
```

Vous pouvez effacer le fichier src/DataFixtures/AppFixtures.php (on en aura pas besoin).

3. On va créer des utilisateurs projet) :

``` 
symfony console make:fixtures UserFixtures
```

Dans le fichier src/DataFixtures/UserFixtures.php

Vous compléter les champs du User, ici :
```
<?php

namespace App\DataFixtures;

use App\Entity\User;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Persistence\ObjectManager;
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;

class UserFixtures extends Fixture
{
    public function __construct(private UserPasswordHasherInterface $passwordEncoder)
    {
        
    }
    
    public function load(ObjectManager $manager): void
    {
        $user = new User();
        $user->setEmail('toto@tata.fr');
        $user->setRoles(['ROLE_USER']);
        $user->setPassword(
            $this->passwordEncoder->hashPassword($user, 'secret'));
        $user->setName('Toto');
        
        $manager->persist($user);

        $manager->flush();
    }
}
```

Executer dans la console pourcharger les fixtures dans la bdd :
``` 
symfony console doctrine:fixtures:load
``` 

A partir de là on a créer un utilisateur mais 

Rajouter à la main en haut de src/DataFixtures/UserFixtures.php :
``` 
use Faker;
```  
et réaliser la création de plusieurs utilisateurs, comme ceci :
``` 
<?php

namespace App\DataFixtures;

use App\Entity\User;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Persistence\ObjectManager;
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;
use Faker;

class UserFixtures extends Fixture
{
    public function __construct(private UserPasswordHasherInterface $passwordEncoder)
    {
        
    }
    
    public function load(ObjectManager $manager): void
    {
        $admin = new User();
        $admin->setEmail('admin@tuto.fr');
        $admin->setRoles(['ROLE_ADMIN']);
        $admin->setPassword(
            $this->passwordEncoder->hashPassword($admin, 'secret'));
        $admin->setName('Toto');
        
        $manager->persist($admin);

        ///////////////////////////////////////

        $faker = Faker\Factory::create('fr_FR');

        for ($i=1; $i <=5 ; $i++) { 
            $user = new User();
            $user->setEmail($faker->email());
            $user->setRoles(['ROLE_USER']);
            $user->setPassword(
                $this->passwordEncoder->hashPassword($user, 'secret'));
            $user->setName($faker->name());
            
            $manager->persist($user);
        }

        $manager->flush();
    }
}

``` 

et executer avec la même commande (ici légèrement différente pour ne pas taper le yes ;) ) :
``` 
symfony console doctrine:fixtures:load --no-interaction
``` 

On a désomais 1 Admin et 5 utlisateurs générés.






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

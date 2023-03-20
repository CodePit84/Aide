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
ou
``` 
symfony console d:f:l
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


4. On va créer des mouvements (pour rester dans la suite de notre projet) :

``` 
symfony console make:fixtures MovementFixtures
```

On va avoir 2 problèmes :
- Le Premier c'est qu'il faut lié un mouvement à un utilisateur !
- Le Second c'est l'ordre dans lequel sont executer le chargement des DataFixtures (par ordre Alphabétique...) Il faut que le chargement de UserFixtures soit exécuter avant MovementFixtures... 

Dans App\DataFixtures\MovementFixtures.php on va mettre :
```
<?php

namespace App\DataFixtures;

use Faker;
use Faker\Factory;
use App\Entity\Movement;
use Doctrine\Persistence\ObjectManager;
use Doctrine\Bundle\FixturesBundle\Fixture;

class MovementFixtures extends Fixture
{
    public function load(ObjectManager $manager): void
    {
        $faker = Factory::create('fr_FR');

        for ($i=1; $i <=25 ; $i++) { 
            $mov = new Movement();
            $mov->setAmount($faker->numberBetween(1000, 150000));
            $mov->setPlace($faker->optional->name);
            $mov->setDate(new \DateTimeImmutable());

            // On va chercher une référence de user
            $user = $this->getReference('user-'. rand(1, 5));
            $mov->setUser($user);
            
            $manager->persist($mov);
        }

        $manager->flush();

        // $manager->flush();
    }
    
    public function createMov(string $amount, ObjectManager $manager)
    {
        $mov = new Movement;
        $mov->setAmount($amount);
        $mov->setPlace('Winamax');
        $mov->setDate(new \DateTimeImmutable());
    
        $manager->persist($mov);

        return $mov;
    }
}
```

On va rajouter dans App\DataFixtures\UserFixtures.php un compteur :
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
    // On crée un compteur pour compter les users pour les fixtures des Movement
    private $counter = 1;
    
    public function __construct(private UserPasswordHasherInterface $passwordEncoder)
    {
        
    }
    
    public function load(ObjectManager $manager): void
    {
        // On crée 1 seul Admin
        $admin = new User();
        $admin->setEmail('admin@tuto.fr');
        $admin->setRoles(['ROLE_ADMIN']);
        $admin->setPassword(
            $this->passwordEncoder->hashPassword($admin, 'secret'));
        $admin->setName('Toto');
        
        $manager->persist($admin);

        //////////////////////////////////////

        // On crée 5 Users
        $faker = Faker\Factory::create('fr_FR');

        for ($i=1; $i <=5 ; $i++) { 
            $user = new User();
            $user->setEmail($faker->email());
            $user->setRoles(['ROLE_USER']);
            $user->setPassword(
                $this->passwordEncoder->hashPassword($user, 'secret'));
            $user->setName($faker->name());
            
            $manager->persist($user);

            // Pour les data fixtures de Movement (MovementFixtures.php)
            $this->addReference('user-'.$this->counter, $user);
            $this->counter++;
        }

        $manager->flush();
    }
}

```
- On va chercher la référence que l'on vient de créer avec le compteur dans la class User avant le persist. 
- Et on va implémenter DependentFixtureInterface à la class, pour changer l'ordre de chargement des DatasFixtures. et crée la méthode obligatoire getDependencies que cet implementation requert. Comme on a créer 5 utilisateurs, on lui dira de prendre un "user-" entre 1 et 5 avec le rand(random).

``` 
<?php

namespace App\DataFixtures;

use Faker;
use Faker\Factory;
use App\Entity\Movement;
use Doctrine\Persistence\ObjectManager;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Common\DataFixtures\DependentFixtureInterface;

class MovementFixtures extends Fixture implements DependentFixtureInterface
{
    public function load(ObjectManager $manager): void
    {
        $faker = Factory::create('fr_FR');

        for ($i=1; $i <=25 ; $i++) { 
            $mov = new Movement();
            $mov->setAmount($faker->numberBetween(1000, 150000));
            // $mov->setPlace($faker->optional->name);
            $mov->setPlace($faker->optional->randomElement(['Winamax', 'Betclic', 'FDJ']));
            $mov->setDate(new \DateTimeImmutable());

            // On va chercher une référence de user
            $user = $this->getReference('user-'. rand(1, 5));
            $mov->setUser($user);
            
            $manager->persist($mov);
        }

        $manager->flush();

        // $manager->flush();
    }
    
    // public function createMov(string $amount, ObjectManager $manager)
    // {
    //     $mov = new Movement;
    //     $mov->setAmount($amount);
    //     $mov->setPlace('Winamax');
    //     $mov->setDate(new \DateTimeImmutable());
    
    //     $manager->persist($mov);

    //     return $mov;
    // }


    // On créer un nouvelle méthode dû au rajout de "implements DependentFixtureInterface" dans "class MovementFixtures extends Fixture"
    // Pour dire que UserFixtures doit être executer avant MovementFixtures (car c'est exécuter de manière alphabétique)
    public function getDependencies(): array
    {
        return [
            UserFixtures::class    
        ];
    }
}
```
On peur alors charger les fixtures sans problème :
``` 
symfony console doctrine:fixtures:load --no-interaction

   > purging database
   > loading App\DataFixtures\AppFixtures
   > loading App\DataFixtures\UserFixtures
   > loading App\DataFixtures\MovementFixtures
```
UserFixtures est bien exécuter avant MovementFixtures ! :)




_________________________________________________________________________________________

* *Note :*
Si problème au niveau de l'effacement des données, lors du chargement des fixtures voir :

6 - Optimisation des entités et DataFixtures (Symfony 6) / 31:10 / Nouvelle Techno (https://youtu.be/JVVeBiewhNg)

``` 
#[ORM\JoinColumn(onDelete: 'CASCADE')]
```

_________________________________________________________________________________________

* *Note 2 :*
Rappel pour créer la bdd : 
``` 
symfony console make:migration
```
``` 
symfony console doctrine:migrations:migrate
```
_________________________________________________________________________________________

* *Note 3 :*
Pour repartir de zéro après plusieurs essais les id dans la bdd sont décaler (autoincrémente oblige + effacement si plusieurs essais... ainsi notre premier utilisateur pourra avoir un id de 16 par exemple). Donnc effaçons tout et chargement 1 fois nos DataFixturs :
``` 
php bin/console d:d:d --force
```
=> On efface la bdd
``` 
symfony console d:d:c
```
=> On recrée la bdd
``` 
symfony console doctrine:schema:update --force
```
=> On met à jour la bdd
``` 
symfony console doctrine:fixtures:load --no-interaction
```
=> Et on charge les Fixtures

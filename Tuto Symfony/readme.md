## Tuto Symfony

1. ```cd Workspace```

2. ```symfony new --webapp nomduprojet```

3. Créer un .env.local et modifier .env

4. ```symfony console doctrine:database:create``` ou d:d:c

5. ```symfony console make:user```

À noter que :

#[ORM\Id] veut dire que c'est une clé primaire.

#[ORM\GeneratedValue] veut dire que c'est auto-incrémenté.

Le champs Id étant automatiquement généré par Symfony !

6. ```symfony console make:entity User```

Pour ensuite ajouter des propriétés (ainsi que leurs get / set) à l'entité comme par exemple :

lastName, firstName, address, zipcode, city, phone, createdAt

"createdAt" sera du type datetime_immutable.
A noter que si propriété "zipcode" (code postale) choisir "string" de "5" caractères, sinon le premier zéro ne sera pas pris en compte pour les 9 premiers départements (ex : 03500) !! Pareil pour le numéro de téléphone ! ;)

7. Dans Entity/User.php rajouter à la ligne $createdAt :
```
#[ORM\Column(type: 'datetime_immutable', options: ['default' => 'CURRENT_TIMESTAMP'])]
    private ?\DateTimeImmutable $createdAt = null;
```

/!\ On rajoutera plus tard dans le Constructeur de l'Entity User.php
```
    public function __construct()
    {
        $this->createdAt = new \DateTimeImmutable();
    }
````  
7. blabla

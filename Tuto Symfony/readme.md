## Tuto Symfony

1. ```cd Workspace```

2. ```symfony new --webapp nomduprojet```

3. Créer un .env.local et modifier .env

4. ```symfony console doctrine:database:create``` ou d:d:c

5. ```symfony console make:user```

6. ```symfony console make:entity User```
Ensuite ajouter des champs à l'entité comme par exemple :
lastName, firstName, address, zipcode, city, phone, createdAt

À noter que :
#[ORM\Id] veut dire que c'est une clé primaire.

#[ORM\GeneratedValue] veut dire que c'est auto-incrémenté.

Le champs Id étant automatiquement généré par Symfony !

7. Dans Entity/User.php rajouter à la ligne $createdAt :
```
#[ORM\Column(type: 'datetime_immutable', options: ['default' => 'CURRENT_TIMESTAMP'])]
    private ?\DateTimeImmutable $createdAt = null;
```

7. blabla

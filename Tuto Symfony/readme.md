## Tuto Symfony

1. ```cd Workspace```

2. ```symfony new --webapp nomduprojet```

3. Créer un .env.local et modifier .env

4. ```symfony console doctrine:database:create``` ou d:d:c

5. ```symfony console make:user```

6. ```symfony console make:entity User```
Pour ajouter des champs à l'entité comme :
lastName, firstName, address, zipcode, city, phone, createdAt

6bis. pour createdAt rajouter :
``` #[ORM\Column(type: 'datetime_immutable', options: ['default' => 'CURRENT_TIMESTAMP'])]
    private ?\DateTimeImmutable $createdAt = null; ```


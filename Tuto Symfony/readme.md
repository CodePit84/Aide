## Tuto Symfony

1. Aller dans le dossier où le projet sera créé :
```cd Workspace```

2. ```symfony new --webapp nomduprojet```

3. Créer un .env.local et modifier .env

Ne pas laisser APP_SECRET et DATABASE_URL dans le .env et le mettre dans le .env.local

4. ```symfony console doctrine:database:create``` ou d:d:c

Répondez Entrée à chaque question.

5. ```symfony console make:user```

À noter que dans src/Entity/User.php :

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
[Chap.3 / 11:59 / Symfony6 / Nouvelle Techno]

:warning: On rajoutera plus tard dans le Constructeur de l'Entity : User.php (étape 19), la ligne : 
```
    public function __construct()
    {
        $this->createdAt = new \DateTimeImmutable();
    }
````  
8. blabla

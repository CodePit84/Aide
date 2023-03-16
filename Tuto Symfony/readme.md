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

:warning: On rajoutera plus tard dans le Constructeur de l'Entity : User.php (étape 19), ou le rajouter maintenant avant les get /set : 
```
    public function __construct()
    {
        $this->createdAt = new \DateTimeImmutable();
    }
````  
8. On va Créer un HomeController.php pour l'affichage de notre première page :
```
symfony console make:controller Home
```
Dans src/Controller/HomeController.php modifier la route pour que l'affichage de cette page soit à la racine de notre application :
```
class HomeController extends AbstractController
{
    #[Route('/', name: 'home')]
    public function index(): Response
    {
        return $this->render('home/index.html.twig', [
            'controller_name' => 'HomeController',
        ]);
    }
}

9. Dans templates/base.html.twig rajouter les balises Meta de votre thème Bootstrap ou Bootswatch, et vos scripts js, par exemple :

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Le-Collectif{% endblock %}</title>
        <link rel="icon" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 128 128%22><text y=%221.2em%22 font-size=%2296%22>⚫️</text></svg>">
        {# Run `composer require symfony/webpack-encore-bundle` to start using Symfony UX #}
        {% block stylesheets %}
            {{ encore_entry_link_tags('app') }}
            <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bootswatch/5.2.3/zephyr/bootstrap.min.css" integrity="sha512-dcTg+pv6j02FTyko5ua8nsnARs/l4u43vmnbeVgkFWB5wdLgfUq4CEotFWOlTE4XK7FfVriWj7BrpqET/a+SJQ==" crossorigin="anonymous" referrerpolicy="no-referrer" />
            <link rel="stylesheet" href="{{ asset('assets/css/styles.css') }}">

        {% endblock %}

        {% block javascripts %}
            {{ encore_entry_script_tags('app') }}
            <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/js/bootstrap.bundle.min.js" integrity="sha384-w76AqPfDkMBDXo30jS1Sgez6pr3x5MlQ1ZAGC+nuZB+EYdgRZgiwxhTBTkF7CXvN" crossorigin="anonymous" defer></script>
            <script src="{{ asset('assets/js/scripts.js') }}" defer></script>
        {% endblock %}
    </head>
    <body>
        {% block header %}
        {% include "partials/_header.html.twig" %}
        
        {% endblock %}
        {% block body %}{% endblock %}
        {% include "partials/_footer.html.twig" %}
    </body>
</html>
```

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
```
9. Dans templates/base.html.twig rajouter les balises Meta de votre thème Bootstrap ou Bootswatch, et vos scripts js, par exemple :

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}SymBetcheck{% endblock %}</title>
        <link rel="icon" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 128 128%22><text y=%221.2em%22 font-size=%2296%22>⚫️</text></svg>">
        {# Run `composer require symfony/webpack-encore-bundle` to start using Symfony UX #}
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <!-- Appel de la Feuille de style minifiée De La librairie Bootswatch -->
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bootswatch/5.2.3/zephyr/bootstrap.min.css" integrity="sha512-dcTg+pv6j02FTyko5ua8nsnARs/l4u43vmnbeVgkFWB5wdLgfUq4CEotFWOlTE4XK7FfVriWj7BrpqET/a+SJQ==" crossorigin="anonymous" referrerpolicy="no-referrer" />
        <!-- Noyau JavaScript de Bootstrap -->
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/js/bootstrap.bundle.min.js" integrity="sha384-w76AqPfDkMBDXo30jS1Sgez6pr3x5MlQ1ZAGC+nuZB+EYdgRZgiwxhTBTkF7CXvN" crossorigin="anonymous" defer></script>
        <!-- Appel de la Feuille de style minifiée De l'extension Datepicker -->
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap-datepicker/1.9.0/css/bootstrap-datepicker.min.css">

        <!-- Extension jquery -->
        <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js"></script>
        <!-- Extension DATEPICKER -->
        <script src="https://cdnjs.cloudflare.com/ajax/libs/bootstrap-datepicker/1.9.0/js/bootstrap-datepicker.min.js"></script>
        <!-- Appel de la Feuille de style pour la police -->
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css">
        <!-- Mon script JavaScript de Datepicker -->
        <script src="{{ asset('assets/js/scripts.js') }}" defer></script>
        
        {% block stylesheets %}
        {% endblock %}

        {% block javascripts %}
        {% endblock %}

    </head>
    <body>
        {% block header %}
            {% include "partials/_header.html.twig" %}
        {% endblock %}

        
        {% include "partials/_flash.html.twig" %}
        
        {% block body %} 

        {% endblock %}

        {% block footer %}
            {% include "partials/_footer.html.twig" %}
        {% endblock %}
    </body>
</html>
```
Veuillez noté le rajout aussi de la balise ViewPort :
```
<meta name="viewport" content="width=device-width, initial-scale=1.0">
``` 
Tout est préférable de le rajouter avant les blocs ou en dehors.
Nous en avons profité aussi pour rajouter des includes, qui nous appelerons nos header, footer, et message flash sur toutes les pages ! ;)

exemple de notre templates/partials/_header.html.twig:
```
<nav class="navbar navbar-expand-lg navbar-dark bg-primary">
  <div class="container-fluid">
    <a class="navbar-brand" href="#">SymBetcheck</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarColor01" aria-controls="navbarColor01" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarColor01">
      <ul class="navbar-nav me-auto">
        <li class="nav-item">
          <a class="nav-link active" href="#">Home
            <span class="visually-hidden">(current)</span>
          </a>
        </li>
        {% if app.user %}
          <li class="nav-item">
            <a class="nav-link" href="{{ path('app_movement_user', {id: app.user.id}) }}">Consulter</a>
          </li>
          <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" data-bs-toggle="dropdown" href="#" role="button" aria-haspopup="true" aria-expanded="false">Ajouter </a>
          <div class="dropdown-menu">
            <a class="dropdown-item" href="{{ path('app_movement_addDeposit_user', {id: app.user.id}) }}">Une dépense</a>
            <div class="dropdown-divider"></div>
            <a class="dropdown-item" href="{{ path('app_movement_addWithdraw_user', {id: app.user.id}) }}">Un encaissement</a>
          </div>
        </li>
        {% endif %}
        {# <li class="nav-item">
          <a class="nav-link" href="#">About</a>
        </li> #}
      </ul>
      <ul class="navbar-nav ms-auto">
      {% if app.user %}
        <li class="nav-item">
          <a class="nav-link" href="{{ path('app_edit_user', {id: app.user.id}) }}">Mon profil</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="{{ path('app_logout') }}">Me déconnecter</a>
        </li>
      {% else %}
        <li class="nav-item">
          <a class="nav-link" href="{{ path('app_register') }}">M'inscrire</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="{{ path('app_login') }}">Me Connecter</a>
        </li>      
      {% endif %}
      </ul>
    </div>
  </div>
</nav>
```
de notre templates/partials/_footer.html.twig:
```
<footer class="text-center text-lg-start bg-light text-muted">
    <div class="text-center p-4" style="background-color: rgba(0, 0, 0, 0.05);">
        © 2023 Copyright:
        <a class="text-reset fw-bold" href="https://github.com/CodePit84">CodePit84</a>
    </div>
</footer>
```
et de notre templates/partials/_flash.html.twig:
```
{% for label, messages in app.flashes %}
    {% for message in messages %}
        <div class="alert alert-{{ label }} alert-dismissible" role="alert">
            <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
            <div class="alert-message">
                {{ message|raw }}
            </div>
        </div>
    {% endfor %}
{% endfor %}
``` 

10. Nous allons créer un système d'authentification avec la commande :
```
symfony console make:auth
```
 

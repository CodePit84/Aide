# Tuto Symfony

## 1. Aller dans le dossier où le projet sera créé :
```cd Workspace```

## 2. ```symfony new --webapp nomduprojet```

## 3. Créer un .env.local et modifier .env

Ne pas laisser APP_SECRET et DATABASE_URL dans le .env et le mettre dans le .env.local

## 4. ```symfony console doctrine:database:create``` ou d:d:c

Répondez Entrée à chaque question.

## 5. ```symfony console make:user```

À noter que dans src/Entity/User.php :

#[ORM\Id] veut dire que c'est une clé primaire.

#[ORM\GeneratedValue] veut dire que c'est auto-incrémenté.

Le champs Id étant automatiquement généré par Symfony !

## 6. ```symfony console make:entity User```

Pour ensuite ajouter des propriétés (ainsi que leurs get / set) à l'entité comme par exemple :

lastName, firstName, address, zipcode, city, phone, createdAt

"createdAt" sera du type datetime_immutable.
A noter que si propriété "zipcode" (code postale) choisir "string" de "5" caractères, sinon le premier zéro ne sera pas pris en compte pour les 9 premiers départements (ex : 03500) !! Pareil pour le numéro de téléphone ! ;)

## 7. Dans Entity/User.php rajouter à la ligne $createdAt :
```
#[ORM\Column(type: 'datetime_immutable', options: ['default' => 'CURRENT_TIMESTAMP'])]
    private ?\DateTimeImmutable $createdAt = null;
```
[Chap.3 / 11:59 / Symfony6 / Nouvelle Techno]

:warning: On rajoutera aussi dans le Constructeur de l'Entity : User.php (pour que la date soit automatiquement importée dans la bdd lors de l'enregistrement d'un nouvel utilisateur), le rajouter avant les get /set : 
```
    public function __construct()
    {
        $this->createdAt = new \DateTimeImmutable();
    }
````  
## 8. On va créer nos Entités, ici 1 seule (Movement) mais vous pouvez créer vos tables maintenant :
## :warning: Attention plus loin dans le tuto, je m'apperçois que c'est un erreur de mettre le champs "date" en "date_immutable" et qu'il faut mieux le mettre en "date", donc si possible le faire maintenant pour éviter la modif plus tard...
```
symfony console make:entity
```
Puis remplisser les champs de l'entité :
```
 Class name of the entity to create or update (e.g. GentleChef):
 > Movement

 created: src/Entity/Movement.php
 created: src/Repository/MovementRepository.php
 
 Entity generated! Now let's add some fields!
 You can always add more fields later manually or by re-running this command.

 New property name (press <return> to stop adding fields):
 > amount

 Field type (enter ? to see all types) [string]:
 > integer

 Can this field be null in the database (nullable) (yes/no) [no]:
 > 

 updated: src/Entity/Movement.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 > place

 Field type (enter ? to see all types) [string]:
 > 

 Field length [255]:
 > 80

 Can this field be null in the database (nullable) (yes/no) [no]:
 > yes

 updated: src/Entity/Movement.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 > date

 Field type (enter ? to see all types) [string]:
 > ?

Main Types
  * string
  * text
  * boolean
  * integer (or smallint, bigint)
  * float

Relationships/Associations
  * relation (a wizard 🧙 will help you build the relation)
  * ManyToOne
  * OneToMany
  * ManyToMany
  * OneToOne

Array/Object Types
  * array (or simple_array)
  * json
  * object
  * binary
  * blob

Date/Time Types
  * datetime (or datetime_immutable)
  * datetimetz (or datetimetz_immutable)
  * date (or date_immutable)
  * time (or time_immutable)
  * dateinterval

Other Types
  * ascii_string
  * decimal
  * guid


 Field type (enter ? to see all types) [string]:
 > date_immutable

 Can this field be null in the database (nullable) (yes/no) [no]:
 > 

 updated: src/Entity/Movement.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 > user

 Field type (enter ? to see all types) [string]:
 > relation

 What class should this entity be related to?:
 > User

What type of relationship is this?
 ------------ --------------------------------------------------------------------- 
  Type         Description                                                          
 ------------ --------------------------------------------------------------------- 
  ManyToOne    Each Movement relates to (has) one User.                             
               Each User can relate to (can have) many Movement objects.            
                                                                                    
  OneToMany    Each Movement can relate to (can have) many User objects.            
               Each User relates to (has) one Movement.                             
                                                                                    
  ManyToMany   Each Movement can relate to (can have) many User objects.            
               Each User can also relate to (can also have) many Movement objects.  
                                                                                    
  OneToOne     Each Movement relates to (has) exactly one User.                     
               Each User also relates to (has) exactly one Movement.                
 ------------ --------------------------------------------------------------------- 

 Relation type? [ManyToOne, OneToMany, ManyToMany, OneToOne]:
 > ManyToOne

 Is the Movement.user property allowed to be null (nullable)? (yes/no) [yes]:
 > no

 Do you want to add a new property to User so that you can access/update Movement objects from it - e.g. $user->getMovements()? (yes/no) [yes]:
 > 

 A new property will also be added to the User class so that you can access the related Movement objects from it.

 New field name inside User [movements]:
 > 

 Do you want to activate orphanRemoval on your relationship?
 A Movement is "orphaned" when it is removed from its related User.
 e.g. $user->removeMovement($movement)
 
 NOTE: If a Movement may *change* from one User to another, answer "no".

 Do you want to automatically delete orphaned App\Entity\Movement objects (orphanRemoval)? (yes/no) [no]:
 > yes

 updated: src/Entity/Movement.php
 updated: src/Entity/User.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 > 


           
  Success! 
           

 Next: When you're ready, create a migration with php bin/console make:migration
``` 
10. On va préparer la migration avec la commande :
```
symfony console make:migration
```
11. On va procéder à la migration avec la commande :
```
symfony console doctrine:migrations:migrate
```
ou 
```
symfony console d:m:m
```

12. On va Créer un HomeController.php pour l'affichage de notre première page :
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
13. Dans templates/base.html.twig rajouter les balises Meta de votre thème Bootstrap ou Bootswatch, et vos scripts js, par exemple :

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
:warning: Attention les routes des liens pourront afficher des erreurs car ici les routes n'existent pas encore à cette étape ou ne sont pas les bonnes !

Notre templates/partials/_footer.html.twig:
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

14. Nous allons créer un système d'authentification avec la commande :

[Chap.5 / Symfony6 / Nouvelle Techno]
```
symfony console make:auth
```
et répondez 1. et lui donner le nom de UsersAuthenticator
```

 What style of authentication do you want? [Empty authenticator]:
  [0] Empty authenticator
  [1] Login form authenticator
 > 1

 The class name of the authenticator to create (e.g. AppCustomAuthenticator):
 > UsersAuthenticator

 Choose a name for the controller class (e.g. SecurityController) [SecurityController]:
 > 

 Do you want to generate a '/logout' URL? (yes/no) [yes]:
 > 
``` 
15. Dans src/Security/UsersAuthenticator.php :
- commenter throw new \Exception
- et décommenter return new RedirectResponse  et lui mettre la Route désirée pour la redirection en cas d'authentification réussie (ici : home).
```
    public function onAuthenticationSuccess(Request $request, TokenInterface $token, string $firewallName): ?Response
    {
        if ($targetPath = $this->getTargetPath($request->getSession(), $firewallName)) {
            return new RedirectResponse($targetPath);
        }

        // For example:
        return new RedirectResponse($this->urlGenerator->generate('home'));
        // throw new \Exception('TODO: provide a valid redirect inside '.__FILE__);
    }
```
16. Pour plus de gueule avec Bootstrap, dans le bloc Body, insérer une section et un container dans templates/security/login.html.twig

en emmet faire : ```section.container>div.row>div.col``` et mettre le formulaire dedans.
Pour obtenir
```
{% extends 'base.html.twig' %}

{% block title %}Connexion{% endblock %}

{% block body %}
<section class="container mt-4">
    <div class="row">
        <div class="col">
            <form method="post">
                {% if error %}
                    <div class="alert alert-danger">{{ error.messageKey|trans(error.messageData, 'security') }}</div>
                {% endif %}

                {% if app.user %}
                    <div class="mb-3">
                        Vous êtes connecté(e) comme {{ app.user.userIdentifier }}, <a href="{{ path('app_logout') }}">Me déconnecter</a>
                    </div>
                {% endif %}

                <h1 class="h3 mb-3 font-weight-normal">Me connecter</h1>
                <label for="inputEmail">Email</label>
                <input type="email" value="{{ last_username }}" name="email" id="inputEmail" class="form-control" autocomplete="email" required autofocus>
                <label for="inputPassword" class="mt-3">Mot de passe</label>
                <input type="password" name="password" id="inputPassword" class="form-control" autocomplete="current-password" required>

                <input type="hidden" name="_csrf_token"
                    value="{{ csrf_token('authenticate') }}"
                >

                {#
                    Uncomment this section and add a remember_me option below your firewall to activate remember me functionality.
                    See https://symfony.com/doc/current/security/remember_me.html

                    <div class="checkbox mb-3">
                        <label>
                            <input type="checkbox" name="_remember_me"> Remember me
                        </label>
                    </div>
                #}

                <button class="btn btn-lg btn-primary mt-3" type="submit">
                    Me connecter
                </button>
                {# <a href="{{ path('app_register') }}" class="btn btn-secondary mt-3">M'inscrire</a> #}
            </form>        
        
        </div>
    </div>
</section>
{% endblock %}

```
J'ai commenté la ligne du lien de l'inscription car la route n'existe pas encore ainsi que le formulaire qui va avec...

17. Il faut désormais créer le formulaire d'inscription :
```
symfony console make:registration-form
```
et valider tout, sauf la vérification par email :
``` 
 Creating a registration form for App\Entity\User

 Do you want to add a @UniqueEntity validation annotation on your User class to make sure duplicate accounts aren't created? (yes/no) [yes]:
 > 

 Do you want to send an email to verify the user's email address after registration? (yes/no) [yes]:
 > no

 Do you want to automatically authenticate the user after registration? (yes/no) [yes]:
 > 
```
18. Dans src/Form/RegistrationFormType.php il va falloir ajouter les champs manquants du formulaire dans son builder, par exemple :

```
->add('name')
```

Vous pouvez aussi changer le agreeTerms, par un RGPDcontent...

19. Editer le rendu templates/registration/register.html.twig comme ça par exemple :
```
{% extends 'base.html.twig' %}

{% block title %}Inscription{% endblock %}

{% block body %}
<section class="container">
    <div class="row">
        <div class="col">
            <h1>Inscription</h1>

            {{ form_start(registrationForm) }}
                <fieldset class="mb-3">
                    <legend>Mon identité</legend>
                    {{ form_row(registrationForm.name) }}
                    {{ form_row(registrationForm.email) }}
                </fieldset>

                {{ form_row(registrationForm.plainPassword, {
                    label: 'Mot de passe'
                }) }}
                {{ form_row(registrationForm.agreeTerms) }}

                <button type="submit" class="btn btn-primary btn-lg my-3">M'inscrire</button>
                <a href="{{ path('app_login') }}" class="btn btn-secondary">Me connecter</a>
            {{ form_end(registrationForm) }}        
        </div>
    </div>
</section>
{% endblock %}

```
20. Vous pouvez ajouter les classes Bootstrap au niveau du twig ou au niveau du formulaire (RegistrationFormType.php) :
ci dessous l'ajout de class Bootstrap, les imports des class type de symfony et la mise en place de la confirmation du mot de passe :

```
<?php

namespace App\Form;

use App\Entity\User;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Validator\Constraints\IsTrue;
use Symfony\Component\Validator\Constraints\Length;
use Symfony\Component\Validator\Constraints\NotBlank;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Component\Form\Extension\Core\Type\CheckboxType;
use Symfony\Component\Form\Extension\Core\Type\PasswordType;
use Symfony\Component\Form\Extension\Core\Type\RepeatedType;

class RegistrationFormType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('email', EmailType::class, [
                'attr' => [
                    'class' => 'form-control'
                ],
                'label' => 'E-mail',
                'label_attr' => [
                    'class' => 'form-label'    
                ]    
            ])
            ->add('name', TextType::class, [
                'attr' => [
                    'class' => 'form-control mb-3'
                ],
                'label' => 'Nom',
                'label_attr' => [
                    'class' => 'form-label'    
                ]   
            ])
            ->add('agreeTerms', CheckboxType::class, [
                'mapped' => false,
                'constraints' => [
                    new IsTrue([
                        'message' => 'You should agree to our terms.',
                    ]),
                ],
                'attr' => [
                    'class' => 'form-check-input mx-1 mt-3'
                ],
                'label' => 'J\'accepte l\'usage de mes coordonnées pour l\'utilisation de l\'application. Nous ne partagerons jamais données.',
                'label_attr' => [
                    'class' => 'form-label mt-3'    
                ]
            ])
            ->add('plainPassword', RepeatedType::class, [
                // instead of being set onto the object directly,
                // this is read and encoded in the controller
                'type' => PasswordType::class,
                'first_options' => [
                    'attr' => [
                        'class' => 'form-control'
                    ],
                    'label' => 'Mot de passe',
                    'label_attr' => [
                        'class' => 'form-label'    
                    ]
                ],
                'second_options' => [
                    'attr' => [
                        'class' => 'form-control'
                    ],
                    'label' => 'Confirmation du mot de passe',
                    'label_attr' => [
                        'class' => 'form-label mt-3'    
                    ]
                ],
                'invalid_message' => 'Les mots de passe ne correspondent pas.',
                'mapped' => false,
                'attr' => [
                    'autocomplete' => 'new-password',
                    'class' => 'form-control'
                ],
                'constraints' => [
                    new NotBlank([
                        'message' => 'Please enter a password',
                    ]),
                    new Length([
                        'min' => 6,
                        'minMessage' => 'Your password should be at least {{ limit }} characters',
                        // max length allowed by Symfony for security reasons
                        'max' => 4096,
                    ]),
                ],
                'label' => 'Mot de passe'
            ])
        ;
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => User::class,
        ]);
    }
}

```
21. On peut dès lors modifier le template _header.html.twig et décommenter les quelques lignes qui nous empêchait jusqu'alors de rendre l'affichage :
```
<ul class="navbar-nav ms-auto">
      {% if app.user %}
        <li class="nav-item">
          {# <a class="nav-link" href="{{ path('app_edit_user', {id: app.user.id}) }}">Mon profil</a> #}
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
```  

Voilà c'est fini pour cette première partie !

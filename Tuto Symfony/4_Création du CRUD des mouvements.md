# 1. On commence par créer le controller des Mouvements
```
symfony console make:controller MovementController
```

# 2. On va editer notre controller : src/Controller/MovementController.php
```
<?php

namespace App\Controller;

use App\Repository\MovementRepository;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class MovementController extends AbstractController
{
    #[Route('/movement', name: 'app_movement')]
    public function index(MovementRepository $movementRepository): Response
    {
        $movements = $movementRepository->findAll();
        // dd($movements);
        
        return $this->render('movement/index.html.twig', [
            'movements' => $movements
        ]);
    }
}
```
Dans un premier temps on affichera TOUS les mouvements de TOUS les utilisateurs...

# 3. On va éditer notre templates/movement/index.html.twig, pour avoir le rendu dans un tableau :
```
{% extends 'base.html.twig' %}

{% block title %}Mes Mouvements{% endblock %}

{% block body %}
    <div class="container mt-4">
        <h1>Mes mouvements</h1>

            {# {{ dump(movement) }} #}

        <table class="table table-hover">
                    <thead class="table-primary">
                        <tr>
                            <th>ID</th>
                            <th>Mouvements en €</th>
                            <th>Endroit</th>
                            <th>Date</th>
                            <th>User ID</th>
                            <th>Modifier</th>
                            <th>Supprimer</th>
                        </tr>
                    </thead>
                    <tbody>
                        {% for movement in movements %}
                        {# {% for movement in movement.user_id %} #}

                            {# {{ dump(movement) }} #}

                            <tr class="table-info">
                                <td>{{ movement.id }}</td>
                                {% if movement.amount <0 %}
                                    <td class="text-danger"><strong>{{ movement.amount }}<strong></td>
                                {% else %}
                                    <td class="text-success"><strong>+{{ movement.amount }}<strong></td>
                                
                                {% endif %}
                                
                                <td>{{ movement.place }}</td>
                                <td>{{ movement.date.format('d/m/Y') }}</td>
                                <td>{{ movement.user.id }}</td>
                                <td>
                                    <a href="#" class="btn btn-info btn-sm">Modifier</a>
                                    {# <a href="{{ path('app_movement_edit', {id: movement.id}) }}" class="btn btn-info btn-sm">Modifier</a> #}
                                </td>
                                <td>
                                    <a href="#" class="btn btn-danger btn-sm" onclick="return confirm('Voulez-vous réellement supprimer ce mouvement ?')">Supprimer</a>
                                    {# <a href="{{ path('app_movement_delete', {id: movement.id}) }}" class="btn btn-danger btn-sm" onclick="return confirm('Voulez-vous réellement supprimer ce mouvement ?')">Supprimer</a> #}
                                </td>
                            </tr>
                        {% endfor %}
                    </tbody>
                </table>

    </div>
{% endblock %}
```

# 4. On va pouvoir procéder à la pagination 

Il va falloir plus tard changer la requête de notre controller pour n'afficher que les mouvements de notre utilisateur ! Mais attaquons nous à la pagination, installons le bundle KNP-Paginator :

```
composer require knplabs/knp-paginator-bundle
```

(la doc de KNP-Paginator est disponible sur GitHub : https://github.com/KnpLabs/KnpPaginatorBundle)

# 5. Il va falloir copié le contenu yaml suivant (dispo sur la doc) :

``` 
knp_paginator:
    page_range: 5                       # number of links shown in the pagination menu (e.g: you have 10 pages, a page_range of 3, on the 5th page you'll see links to page 4, 5, 6)
    default_options:
        page_name: page                 # page query parameter name
        sort_field_name: sort           # sort field query parameter name
        sort_direction_name: direction  # sort direction query parameter name
        distinct: true                  # ensure distinct results, useful when ORM queries are using GROUP BY statements
        filter_field_name: filterField  # filter field query parameter name
        filter_value_name: filterValue  # filter value query parameter name
    template:
        pagination: '@KnpPaginator/Pagination/sliding.html.twig'     # sliding pagination controls template
        sortable: '@KnpPaginator/Pagination/sortable_link.html.twig' # sort link template
        filtration: '@KnpPaginator/Pagination/filtration.html.twig'  # filters template
``` 
Pour celà il faut aller dans config\packages et créer un nouveau fichier "knp_paginator.yaml" et lui coller le contenu !

( 👨‍💻 Apprendre #Symfony 6 - CRUD des ingrédients #8 / Développeur Musclé / 17 minutes / https://youtu.be/AcMtHRfg0fk)
Il va falloir créer au niveau de notre controller une injection de dépendance de KNPPaginator, de de request, ainsi notre MovementController.php, ressemblera à :
```
<?php

namespace App\Controller;

use App\Repository\MovementRepository;
use Knp\Component\Pager\PaginatorInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class MovementController extends AbstractController
{
    #[Route('/movement', name: 'app_movement')]
    public function index(MovementRepository $movementRepository, PaginatorInterface $paginator, Request $request): Response
    {
        $movements = $paginator->paginate(
            $movementRepository->findAll(),
            $request->query->getInt('page', 1), /*page number*/
            10 /*limit per page*/
        );
                
        return $this->render('movement/index.html.twig', [
            'movements' => $movements
        ]);
    }
}
```

On a alors 10 mouvements d'afficher mais pas encore la pagination ! Simplement parce-que dans notre vue, on doit rajouter ceci en bas de la page :
```
<div class="navigation">
    {{ knp_pagination_render(movements) }}
</div>
```
On peut l'améliorer avec les thèmes de Bootstrap, donc on copie ``` @KnpPaginator/Pagination/bootstrap_v5_pagination.html.twig ``` et on va changer le thème dans config/packages/knp_paginator.yaml :
```
knp_paginator:
    page_range: 5                       # number of links shown in the pagination menu (e.g: you have 10 pages, a page_range of 3, on the 5th page you'll see links to page 4, 5, 6)
    default_options:
        page_name: page                 # page query parameter name
        sort_field_name: sort           # sort field query parameter name
        sort_direction_name: direction  # sort direction query parameter name
        distinct: true                  # ensure distinct results, useful when ORM queries are using GROUP BY statements
        filter_field_name: filterField  # filter field query parameter name
        filter_value_name: filterValue  # filter value query parameter name
    template:
        pagination: '@KnpPaginator/Pagination/bootstrap_v5_pagination.html.twig'     # Bootstrap5 pagination controls template
        sortable: '@KnpPaginator/Pagination/sortable_link.html.twig' # sort link template
        filtration: '@KnpPaginator/Pagination/filtration.html.twig'  # filters template
```
On peut aussi le centrer en utilisant les classes de Bootstrap suivante ``` d-flex justify-content-center ``` , ce qui donnera donc :
```
<div class="navigation d-flex justify-content-center mt-4">
    {{ knp_pagination_render(movements) }}
</div>
```

# 6. On peut même au niveau de notre vue rajouter le nombre de mouvements.
KNPPaginator nous propose le twig suivant :
``` 
<div class="count">
    {{ movements.getTotalItemCount }}
</div>
```
Que l'ont peut rajouter en haut, ce qui nous donnera au final :
```
{% extends 'base.html.twig' %}

{% block title %}Mes Mouvements{% endblock %}

{% block body %}
    <div class="container mt-4">
        <h1>Mes mouvements</h1>

            {# {{ dump(movement) }} #}
        <div class="count mt-4">
            Il y a {{ movements.getTotalItemCount }} mouvements au total
        </div>

        <table class="table table-hover mt-4">
                    <thead class="table-primary">
                        <tr>
                            <th>ID</th>
                            <th>Mouvements en €</th>
                            <th>Endroit</th>
                            <th>Date</th>
                            <th>User ID</th>
                            <th>Modifier</th>
                            <th>Supprimer</th>
                        </tr>
                    </thead>
                    <tbody>
                        {% for movement in movements %}
                        {# {% for movement in movement.user_id %} #}

                            {# {{ dump(movement) }} #}

                            <tr class="table-info">
                                <td>{{ movement.id }}</td>
                                {% if movement.amount <0 %}
                                    <td class="text-danger"><strong>{{ movement.amount }}<strong></td>
                                {% else %}
                                    <td class="text-success"><strong>+{{ movement.amount }}<strong></td>
                                
                                {% endif %}
                                
                                <td>{{ movement.place }}</td>
                                <td>{{ movement.date.format('d/m/Y') }}</td>
                                <td>{{ movement.user.id }}</td>
                                <td>
                                    <a href="#" class="btn btn-info btn-sm">Modifier</a>
                                    {# <a href="{{ path('app_movement_edit', {id: movement.id}) }}" class="btn btn-info btn-sm">Modifier</a> #}
                                </td>
                                <td>
                                    <a href="#" class="btn btn-danger btn-sm" onclick="return confirm('Voulez-vous réellement supprimer ce mouvement ?')">Supprimer</a>
                                    {# <a href="{{ path('app_movement_delete', {id: movement.id}) }}" class="btn btn-danger btn-sm" onclick="return confirm('Voulez-vous réellement supprimer ce mouvement ?')">Supprimer</a> #}
                                </td>
                            </tr>
                        {% endfor %}
                    </tbody>
                </table>

                <div class="navigation d-flex justify-content-center mt-4">
                    {{ knp_pagination_render(movements) }}
                </div>

    </div>
{% endblock %}
```
# 7. On va traduire notre Paginator et en profiter pour mettre le projet symfony en français !
Dans config/services.yaml rajouter le paramètre suivant :
```
parameters:
    locale: 'fr'
```  
et dans config/packages/translation.yaml, passer en fr :
```
framework:
    default_locale: fr
    translator:
        default_path: '%kernel.project_dir%/translations'
        fallbacks:
            - fr
```
Voilà c'est tout ! ;)

# 8. On va changer notre requête pour n'afficher que les mouvements de l'utilisateur connecté !
Dans notre MovementController.php on va donc remplacer
```
$movementRepository->findAll(),
```
par :

```
$movementRepository->findBy(['user' => $this->getUser()]),
``` 
c'est tout !
Pour plus de lisibilité et pour la suite on va plutôt le mettre dans une variable :
```
$movementsUser = $movementRepository->findBy(['user' => $this->getUser()]);
        
        $movements = $paginator->paginate(
            $movementsUser,
            $request->query->getInt('page', 1), /*page number*/
            10 /*limit per page*/
        );
```

# 9. On va aussi vouloir afficher notre solde (la somme de tous les mouvements).
Il va falloir ajouter dans la méthode la somme :
```
<?php

namespace App\Controller;

use App\Repository\MovementRepository;
use Knp\Component\Pager\PaginatorInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class MovementController extends AbstractController
{
    /**
     * This function allows us to display all movement's user
     *
     * @param MovementRepository $movementRepository
     * @param PaginatorInterface $paginator
     * @param Request $request
     * @return Response
     */
    #[Route('/movement', name: 'app_movement')]
    public function index(MovementRepository $movementRepository, PaginatorInterface $paginator, Request $request): Response
    {
        // Requête :
        $movementsUser = $movementRepository->findBy(['user' => $this->getUser()]);
        
        // La somme de tous les mouvements :
        $sum = 0;
        foreach ($movementsUser as $movement) {
            // dump($movement->getMovement());
            $sum = $sum + $movement->getAmount();
        }

        
        // Pagination :
        $movements = $paginator->paginate(
            $movementsUser,
            $request->query->getInt('page', 1), /*page number*/
            10 /*limit per page*/
        );
                
        return $this->render('movement/index.html.twig', [
            'sum' => $sum,
            'movements' => $movements
        ]);
    }
}
```

On va aussi diviser par 100 dans la vue, car nos sommes sont entrées dans la base de donnée en centimes (integer) et on va en profiter pour rajouter une conditions s'il l'utilisateurs n'a pas de mouvement :
``` 
{% extends 'base.html.twig' %}

{% block title %}Mes Mouvements{% endblock %}

{% block body %}
    <div class="container mt-4">

        {# Conditions si Mouvements ou Pas : #}
        {% if not movements.items is same as ([]) %}

        <h3>Mouvements de {{ app.user.name }}</h3> 
                <div class="ml-auto p-2">
                    <div class="d-flex justify-content-between">
                        <div class="d-flex">
                        Solde :
                        {% if sum > 0 %}
                            <span class="badge rounded-pill bg-success mx-2">{{ sum / 100 }} €</span>
                        {% else %}
                            <span class="badge rounded-pill bg-danger mx-2">{{ sum / 100 }} €</span>
                        {% endif %}
                        </div>
                        <div class="d-flex">
                        ({{ movements.getTotalItemCount }} mouvements)
                        </div>
                    </div>
                </div>

        <table class="table table-hover mt-4">
                    <thead class="table-primary">
                        <tr>
                            <th>ID</th>
                            <th>Mouvements en €</th>
                            <th>Endroit</th>
                            <th>Date</th>
                            <th>User ID</th>
                            <th>Modifier</th>
                            <th>Supprimer</th>
                        </tr>
                    </thead>
                    <tbody>
                        {% for movement in movements %}
                        {# {% for movement in movement.user_id %} #}

                            {# {{ dump(movement) }} #}

                            <tr class="table-info">
                                <td>{{ movement.id }}</td>
                                {% if movement.amount <0 %}
                                    <td class="text-danger"><strong>{{ movement.amount / 100 }}<strong></td>
                                {% else %}
                                    <td class="text-success"><strong>+{{ movement.amount / 100 }}<strong></td>
                                
                                {% endif %}
                                
                                <td>{{ movement.place }}</td>
                                <td>{{ movement.date.format('d/m/Y') }}</td>
                                <td>{{ movement.user.id }}</td>
                                <td>
                                    <a href="#" class="btn btn-info btn-sm">Modifier</a>
                                    {# <a href="{{ path('app_movement_edit', {id: movement.id}) }}" class="btn btn-info btn-sm">Modifier</a> #}
                                </td>
                                <td>
                                    <a href="#" class="btn btn-danger btn-sm" onclick="return confirm('Voulez-vous réellement supprimer ce mouvement ?')">Supprimer</a>
                                    {# <a href="{{ path('app_movement_delete', {id: movement.id}) }}" class="btn btn-danger btn-sm" onclick="return confirm('Voulez-vous réellement supprimer ce mouvement ?')">Supprimer</a> #}
                                </td>
                            </tr>
                        {% endfor %}
                    </tbody>
                </table>

                <div class="navigation d-flex justify-content-center mt-4">
                    {{ knp_pagination_render(movements) }}
                </div>

        {% else %}
            <h4>Il n'y a pas de mouvements</h4>
        {% endif %}

    </div>
{% endblock %}
``` 

# 10. On va procéder maintenant à l'ajout d'un mouvement :
```
symfony console make:form
``` 
que l'on appellera "MovementFormType" et que l'on liera à l'entitée Movement :
``` 
symfony console make:form

 The name of the form class (e.g. TinyPuppyType):
 > MovementFormType

 The name of Entity or fully qualified model class name that the new form will be bound to (empty for none):
 > Movement

 created: src/Form/MovementFormType.php
```

11. Créer une vue Ajout (templates)
En fait ici on va en créer 2, c'est ce que j'ai choisi, 1 pour un dépot (addDeposit.html.twig), 1 pour un encaissement (addWithdraw.html.twig), dans templates/movement :

- addDeposit.html.twig : 
```
{% extends 'base.html.twig' %}

{% block title %}Ajout d'un mouvement{% endblock %}

{% block body %}
<section class="container">
    <div class="row mt-4">
        <div class="col">
            <h1>Ajout d'une Dépense</h1>

            {# {{ dump(app) }} #}

            {{ form_start(movementForm) }}

                {# {{ dump(movementForm) }} #}

                <fieldset class="mb-3">
                    <legend>Ma dépense</legend>

                    {{ form_row(movementForm.amount) }}
                    {{ form_row(movementForm.place) }}
                    {{ form_row(movementForm.date) }}
                    {# {{ form_row(movementForm.user_id) }} #}

                </fieldset>
                <button type="submit" class="btn btn-primary btn-lg my-3">Enregistrer</button>
            {{ form_end(movementForm) }}        
        </div>
    </div>
</section>
{% endblock %}
```


- addWithdraw.html.twig : 
```
{% extends 'base.html.twig' %}

{% block title %}Ajout d'un mouvement{% endblock %}

{% block body %}
<section class="container">
    <div class="row mt-4">
        <div class="col">
            <h1>Ajout d'un Encaissement</h1>

            {# {{ dump(app) }} #}

            {{ form_start(movementForm) }}

        
                {# {{ dump(movementForm) }} #}

                <fieldset class="mb-3">
                    <legend>Mon Encaissement</legend>

                    {{ form_row(movementForm.amount) }}
                    {{ form_row(movementForm.place) }}
                    {{ form_row(movementForm.date) }}
                    {# {{ form_row(movementForm.user_id) }} #}
                </fieldset>
                <button type="submit" class="btn btn-primary btn-lg my-3">Enregistrer</button>
            {{ form_end(movementForm) }}        
        </div>
    </div>
</section>
{% endblock %}
```

Au Niveau de notre formulaire MovementFormType, on peut supprimer le ``` ->add('user') ``` on le settera (set) au niveau de notre controller... Mais on va lui donner un peut de gueule avec les classes Bootstrap (à noter le spoil de la Date en commentaire, car on passera par un widget JS)
```
<?php

namespace App\Form;

use App\Entity\Movement;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Form\Extension\Core\Type\DateType;
use Symfony\Component\Form\Extension\Core\Type\MoneyType;
use Symfony\Component\Form\Extension\Core\Type\ChoiceType;

class MovementFormType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('amount', MoneyType::class, [
                'attr' => [
                    'class' => 'form-control'
                ],
                'label' => 'Montant'    
            ])
            ->add('place', ChoiceType::class, [
                'attr' => [
                    'class' => 'form-control mb-3 form-select'
                ],
                'label' => 'Endroit',
                'choices' => [
                    'Choisir (facultatif)' => ' ',
                    'Betclic' => 'Betclic',
                    'Winamax' => 'Winamax',
                    'FDJ' => 'FDJ' 
                    ]
            ])
            ->add('date', DateType::class, [
                // 'widget' => 'single_text',
                // 'html5' => false,
                // 'attr' => [
                //     'class' => 'form-control js-datepicker'
                // ],
                'attr' => [
                    'class' => 'form-control'
                ],
                'label' => 'Date'
            ])
        ;
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Movement::class,
        ]);
    }
}
``` 
# :caution: On a fait une erreur lors de la création de l'entité, le champs date on l'a mis en DateTimeImmutable alors qu'il aurait fallu le mettre en simple Date.
On va corriger tout ça au niveau de notre Entity/Mouvement.php et remplacer le champs ainsi que ses get/set :
```
    // #[ORM\Column(type: Types::DATE_IMMUTABLE)]
    // private ?\DateTimeImmutable $date = null;
    #[ORM\Column(type: Types::DATE_MUTABLE)]
    private ?\DateTimeInterface $date = null;
    
    
    

    // public function getDate(): ?\DateTimeImmutable
    // {
    //     return $this->date;
    // }
    public function getDate(): ?\DateTimeInterface
    {
        return $this->date;
    }

    // public function setDate(\DateTimeImmutable $date): self
    // {
    //     $this->date = $date;

    //     return $this;
    // }
    public function setDate(\DateTimeInterface $date): self
    {
        $this->date = $date;

        return $this;
    }
```
J'ai mis en commentaire volontairement ce qu'il y a a remplacé par ce qui n'est pas en commentaire... ;)

# 12. Ensuite au niveau de notre MovementController il faudra lui créer une route et une méthode :

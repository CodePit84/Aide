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

On va aussi diviser par 100 car nos sommes sont entrée dans la base de donnée en centimes (integer)

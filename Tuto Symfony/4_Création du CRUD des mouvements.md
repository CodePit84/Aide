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

# 11. Créer une vue Ajout (templates)
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
                'label' => 'Montant',
                'divisor' => 100
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
A noté le 'divisor' => 100 au niveau de l'add('amount') car nos prix doivent être enregistrés en centimes dans la bdd !

# :warning: On a fait une erreur lors de la création de l'entité, le champs date on l'a mis en DateTimeImmutable alors qu'il aurait fallu le mettre en simple Date.
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

Il faut effacer la bdd, faire la migration et réimporter des fixtures, allez hop :
```
symfony console d:d:d --force
symfony console d:d:c
symfony console make:migration
(symfony console d:m:m)
symfony console d:f:l
```
# 12. On peut intégrer aussi les classes de formulaire de Bootstrap automatiquement comme ceci :
https://github.com/CodePit84/Aide/blob/main/Tuto%20Symfony/Int%C3%A9grer%20les%20classes%20Bootstrap%20pour%20les%20formulaires.md

# 13. Ensuite au niveau de notre MovementController il faudra lui créer une route et une méthode :
(en fait 2 car on a 2 méthodes, une pour le deposit et une pour le withdraw :
``` 
<?php

namespace App\Controller;

use App\Entity\User;
use App\Entity\Movement;
use App\Form\MovementFormType;
use App\Repository\MovementRepository;
use Doctrine\ORM\EntityManagerInterface;
use Knp\Component\Pager\PaginatorInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

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

    #[Route('/movement/addWithdraw/user/{id}', name: 'app_movement_addWithdraw_user')]
    public function addWithdraw(Request $request, EntityManagerInterface $entityManager, User $user): Response
    {
        $userId = $user->getId();
        
        $movement = new Movement();
        
        $form = $this->createForm(MovementFormType::class, $movement);
        
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {

            // ceci fonctionnait :
            // $form->getData()->setUser($user);

            $movement = $form->getData();
            // dd($movement);

            $movement->setUser($this->getUser());

            $entityManager->persist($movement);
            $entityManager->flush();

            $this->addFlash('success', 'Mouvement ajouté avec succès');

            // return $this->redirectToRoute('app_movement_user', array('id' => $userId));
            return $this->redirectToRoute('app_movement');
        }
        
        return $this->render('movement/addWithdraw.html.twig', [
            'movementForm' => $form->createView(),
        ]);
    }
    
    #[Route('/movement/addDeposit/user/{id}', name: 'app_movement_addDeposit_user')]
    public function addDeposit(Request $request, EntityManagerInterface $entityManager, User $user): Response
    {
        $userId = $user->getId();
        
        $movement = new Movement();
        
        $form = $this->createForm(MovementFormType::class, $movement);
        
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            
            $movement = $form->getData();
            $movement->setUser($this->getUser());

            // Pour transformer la valeur en négatif
            $amount = $movement->getAmount() * -1 ;
            $movement->setAmount($amount);

            $entityManager->persist($movement);
            $entityManager->flush();

            $this->addFlash('success', 'Mouvement ajouté avec succès');

            return $this->redirectToRoute('app_movement');
        }
        
        return $this->render('movement/addDeposit.html.twig', [
            'movementForm' => $form->createView(),
        ]);
    }
}
```

# 14. Passons à l'édition, celà sera relativement pareil !
Créons une route au niveau de notre Controller :
```
#[Route('/movement/edit/{id}', name: 'app_movement_edit')]
    public function edit(Movement $movement, Request $request, EntityManagerInterface $entityManager): Response
    {
        // On récupère le montant
        $movementAmount = $movement->getAmount();
        
        // On crée une variable isDeposit qui permet de savoir si c'est une dépense et on remet le montant en positif (*-1) pour l'affichage
        if ($movementAmount < 0) {
            $movement->setAmount($movementAmount * -1);
            $isDeposit = true;
        } else {
            $isDeposit = false;
        }
        
        $form = $this->createForm(MovementFormType::class, $movement);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            // Si c'est une dépense on remet le montant en négatif
            if ($isDeposit) {
                $amount = $movement->getAmount() * -1 ;
                $movement->setAmount($amount);
            }
               
            $entityManager->persist($movement);
            $entityManager->flush();

            $this->addFlash('success', 'Mouvement modifié avec succès');

            return $this->redirectToRoute('app_movement');
        }
        
        return $this->render('movement/edit.html.twig', [
            'movementForm' => $form->createView(),
            'movement' => $movement,
            'isDeposit' => $isDeposit
        ]);
    }
```
J'ai rajouté des conditions et une variable (bool) isDeposit pour que l'affichage reste des valeurs positives, que l'on enverra à la vue...

Il faut aussi modifié légèrement la vue, qui inclura la lecture de cette variable mais qui reprend les mêmes grandes lignes que nos add*****.html.twig

On crée donc donc un fichier edit.html.twig :
``` 
{% extends 'base.html.twig' %}

{% block title %}Modification d'un mouvement{% endblock %}

{% block body %}
<section class="container">
    <div class="row mt-4">
    {# {{ dump(isDeposit) }} #}
        <div class="col">
            {% if isDeposit %}
                <h1>Modification d'une Dépense</h1>
            {% else %}
                <h1>Modification d'un Encaissement</h1>    
            {% endif %}
            {{ form_start(movementForm) }}
                <fieldset class="mb-3">
                    <legend>Mon mouvement</legend>
                    {{ form_row(movementForm.amount) }}
                    {{ form_row(movementForm.place) }}
                    {{ form_row(movementForm.date) }}
                </fieldset>
                <button type="submit" class="btn btn-primary btn-lg my-3">Enregistrer</button>
            {{ form_end(movementForm) }}        
        </div>
    </div>
</section>
{% endblock %}
```
Voilà l'édition est terminé, On peut Voir les mouvements, en Ajouter, les Editer, il nous reste plus qu'à pouvoir les Supprimer ! ;)

# 15. Suppression d'un mouvement
Dans le controller on rajoutera la route suivante :
``` 
    #[Route('/movement/delete/{id}', name: 'app_movement_delete')]
    public function delete(Movement $movement, EntityManagerInterface $entityManager):Response
    {
        $entityManager->remove($movement);
        $entityManager->flush();

        $this->addFlash('success', 'Mouvement supprimé avec succès');

        return $this->redirectToRoute('app_movement');
    }
``` 

# 16. La sécurité des accès à nos Routes :
Il va falloir tout sécuriser pour qu'un utilisateur n'aie accès qu'à ses propres mouvements...
Pour celà nous devons rajouter dans notre function : ``` User $choosenUser ```  une Route Security au dessus de la Route ``` #[Security("is_granted('ROLE_USER') and user === choosenUser")] ``` et pour certaine Route ``` #[Security("is_granted('ROLE_USER') and user === movement.getUser()")] ``` et le use qui va avec ``` use Sensio\Bundle\FrameworkExtraBundle\Configuration\Security; ``` pour qu'on obtienne au final un MovementController.php ainsi :
``` 
<?php

namespace App\Controller;

use App\Entity\User;
use App\Entity\Movement;
use App\Form\MovementFormType;
use App\Repository\MovementRepository;
use Doctrine\ORM\EntityManagerInterface;
use Knp\Component\Pager\PaginatorInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Security;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class MovementController extends AbstractController
{
    /**
     * This controller allows us to View / Display all movement's user
     *
     * @param User $choosenUser
     * @param MovementRepository $movementRepository
     * @param PaginatorInterface $paginator
     * @param Request $request
     * @return Response
     */
    #[Security("is_granted('ROLE_USER') and user === choosenUser")]
    #[Route('/movement/user/{id}', name: 'app_movement_user')]
    public function index(User $choosenUser, MovementRepository $movementRepository, PaginatorInterface $paginator, Request $request): Response
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

    /**
     * This controller allows us to Add a Withdraw
     *
     * @param User $choosenUser
     * @param Request $request
     * @param EntityManagerInterface $entityManager
     * @return Response
     */
    #[Security("is_granted('ROLE_USER') and user === choosenUser")]
    #[Route('/movement/addWithdraw/user/{id}', name: 'app_movement_addWithdraw_user')]
    public function addWithdraw(User $choosenUser, Request $request, EntityManagerInterface $entityManager): Response
    {
        $userId = $choosenUser->getId();
        
        $movement = new Movement();
        
        $form = $this->createForm(MovementFormType::class, $movement);
        
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {

            // ceci fonctionnait :
            // $form->getData()->setUser($user);

            $movement = $form->getData();
            // dd($movement);

            $movement->setUser($this->getUser());

            $entityManager->persist($movement);
            $entityManager->flush();

            $this->addFlash('success', 'Mouvement ajouté avec succès');

            return $this->redirectToRoute('app_movement_user', array('id' => $userId));
        }
        
        return $this->render('movement/addWithdraw.html.twig', [
            'movementForm' => $form->createView(),
        ]);
    }
    
    /**
     * This controller allows us to Add a Deposit
     *
     * @param Request $request
     * @param EntityManagerInterface $entityManager
     * @param User $choosenUser
     * @return Response
     */
    #[Security("is_granted('ROLE_USER') and user === choosenUser")]
    #[Route('/movement/addDeposit/user/{id}', name: 'app_movement_addDeposit_user')]
    public function addDeposit(Request $request, EntityManagerInterface $entityManager, User $choosenUser): Response
    {
        $userId = $choosenUser->getId();
        
        $movement = new Movement();
        
        $form = $this->createForm(MovementFormType::class, $movement);
        
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            
            $movement = $form->getData();
            $movement->setUser($this->getUser());

            // Pour transformer la valeur en négatif
            $amount = $movement->getAmount() * -1 ;
            $movement->setAmount($amount);

            $entityManager->persist($movement);
            $entityManager->flush();

            $this->addFlash('success', 'Mouvement ajouté avec succès');

            return $this->redirectToRoute('app_movement_user', array('id' => $userId));
        }
        
        return $this->render('movement/addDeposit.html.twig', [
            'movementForm' => $form->createView(),
        ]);
    }

    /**
     * This controller allows us to Edit a movement
     *
     * @param Movement $movement
     * @param Request $request
     * @param EntityManagerInterface $entityManager
     * @return Response
     */
    #[Security("is_granted('ROLE_USER') and user === movement.getUser()")]
    #[Route('/movement/edit/{id}', name: 'app_movement_edit')]
    public function edit(Movement $movement, Request $request, EntityManagerInterface $entityManager): Response
    {
        // On récupère l'Id de l'utilisateur courant pour le redirectToRoute :
        // $userId = $user->getId();
        // $userId = $this->getUser()->getId();
        $userId = $movement->getUser()->getId();
        
        // On récupère le montant
        $movementAmount = $movement->getAmount();
        
        // On crée une variable isDeposit qui permet de savoir si c'est une dépense et on remet le montant en positif (*-1) pour l'affichage
        if ($movementAmount < 0) {
            $movement->setAmount($movementAmount * -1);
            $isDeposit = true;
        } else {
            $isDeposit = false;
        }
        
        $form = $this->createForm(MovementFormType::class, $movement);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            // Si c'est une dépense on remet le montant en négatif
            if ($isDeposit) {
                $amount = $movement->getAmount() * -1 ;
                $movement->setAmount($amount);
            }
               
            $entityManager->persist($movement);
            $entityManager->flush();

            $this->addFlash('success', 'Mouvement modifié avec succès');

            return $this->redirectToRoute('app_movement_user', array('id' => $userId));
        }
        
        return $this->render('movement/edit.html.twig', [
            'movementForm' => $form->createView(),
            'movement' => $movement,
            'isDeposit' => $isDeposit
        ]);
    }

    /**
     * This controller allows us to Delete a movement
     *
     * @param Movement $movement
     * @param EntityManagerInterface $entityManager
     * @return Response
     */
    #[Security("is_granted('ROLE_USER') and user === movement.getUser()")]
    #[Route('/movement/delete/{id}', name: 'app_movement_delete')]
    public function delete(Movement $movement, EntityManagerInterface $entityManager):Response
    {
        $userId = $movement->getUser()->getId();

        $entityManager->remove($movement);
        $entityManager->flush();

        $this->addFlash('success', 'Mouvement supprimé avec succès');

        return $this->redirectToRoute('app_movement_user', array('id' => $userId));
    }

}
```
Voilà ! A noter que pour l'affichage des mouvement de l'utilisateur on a dû changer la Route :
On a remplacer : 

``` #[Route('/movement', name: 'app_movement')] ``` 

par :

``` #[Route('/movement/user/{id}', name: 'app_movement_user')] ```

pour que l'utilisateur passe dans l'URL...

donc on a dû changer tous nos redirectToRoute :
``` 
return $this->redirectToRoute('app_movement');
```
par :
``` 
return $this->redirectToRoute('app_movement_user', array('id' => $userId));
```
parce que l'id demandé doit être du type array et pas string...
et que l'on doit rajouter dans la méthode ``` $userId = $movement->getUser()->getId(); ``` pour récupérer l'id de l'utilisateur courant...

    
# 17 : On va devoir créer enfin nos liens dans nos vues pour facilement accéder à nos Routes :
Dans un premier temps notre templates/partials/_header.html.twig
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
    </div>
  </div>
</nav>
```

J'ai laissé vonlontaire en commentaire le futur lien pour l'édition du profil...

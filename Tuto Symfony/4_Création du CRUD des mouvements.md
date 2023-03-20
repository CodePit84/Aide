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

# 4. Il va falloir changer la requête de notre controller pour n'afficher que les mouvements de notre utilisateur !

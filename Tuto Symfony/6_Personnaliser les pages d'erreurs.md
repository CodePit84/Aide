# On va personnaliser nos 404 et nos 403 !

[11 - Personnaliser les pages d'erreurs (Symfony 6) / Nouvelle Techno / https://youtu.be/BXMjfCk8DSY]

## 1. Installer twigpack : 
``` 
composer require symfony/twig-pack
``` 

## 2. Créer un Dossier "bundle" dans le dossiers "templates"
- Créer un dossier "TwigBundle" dans ce dossier (attention au Majuscules)
- Créer un dossier "Exception" dans ce dossier
- Créer un fichier "error404.html.twig" et un fichier "error403.html.twig" dans ce dossier

## 3. On va éditer nos fichiers d'erreurs :
notre error404.html.twig :
``` 
{% extends "base.html.twig" %}

{% block title %}Page non trouvée{% endblock %}

{% block body %}
    <main class="container">
        <section class="row">
            <div class="col-12">
                <h1>Page non trouvée</h1>
                <p>La page demandée n'existe pas</p>
            </div>
        </section>
    </main>
{% endblock %}
``` 

et notre error403.html.twig :
``` 
{% extends "base.html.twig" %}

{% block title %}Accès interdit{% endblock %}

{% block body %}
    <main class="container">
        <section class="row">
            <div class="col-12">
                <h1>Accès interdit</h1>
                <p>Vous n'avez pas l'autorisation de consulter cette page.</p>
            </div>
        </section>
    </main>
{% endblock %}
``` 


## Pour accéder aux erreurs en prod, il suffira de se rendre sur les URL suivantes :
- https://127.0.0.1:8000/_error/404
- https://127.0.0.1:8000/_error/403

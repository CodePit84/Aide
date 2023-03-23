# Pour notre application, on aurait besoin de l'affichage d'un calendrier pour le choix de la date.

## 1. Pour celà on va utiliser DATEPICKER, on va rajouter dans notre templates/base.html.twig les extensions suivantes :
``` 
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
``` 

à la suite de nos feuilles de styles Bootstrap et de son noyau Javascript.

## 2. Dans notre dossier "public", on va créer un dossier "assets", et créer un dossier "js" dans celui-ci. Là on va créer un fichier "scripts.js" contenant :
``` 
$.fn.datepicker.dates['fr'] = {
    days: ["Dimanche", "Lundi", "Mardi", "Mercredi", "Jeudi", "Vendredi", "Samedi"],
    daysShort: ["Dim", "Lun", "Mar", "Mer", "Jeu", "Ven", "Sam"],
    daysMin: ["Di", "Lu", "Ma", "Me", "Je", "Ve", "Sa"],
    months: ["Janvier", "Fevrier", "Mars", "Avril", "Mai", "Juin", "Juillet", "Août", "Septembre", "Octobre", "Novembre", "Décembre"],
    monthsShort: ["Jan", "Fev", "Mar", "Avr", "Mai", "Jun", "Jul", "Aoû", "Sep", "Oct", "Nov", "Dec"],
    today: "Aujourd'hui",
    clear: "Effacer",
    format: "yyyy-mm-dd",
    titleFormat: "MM yyyy", /* Leverages same syntax as 'format' */
    weekStart: 1
};

jQuery(document).ready(function() {
    $('.js-datepicker').datepicker({
        language: 'fr'
    });
});
```  
Voilà c'est tout !

ce qui donnera base.html.twig :
``` 
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Welcome!{% endblock %}</title>
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
            {{ encore_entry_link_tags('app') }}
        {% endblock %}

        {% block javascripts %}
            {{ encore_entry_script_tags('app') }}
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

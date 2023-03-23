# Pour notre application, on aurait besoin de l'affichage d'un calendrier pour le choix de la date.

# 1. Pour celà on va utiliser DATEPICKER, on va rajouter dans notre templates/base.html.twig les extensions suivantes :
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

# 2. Dans notre dossier "public", on va créer un dossier "assets", et créer un dossier "js" dans celui-ci. Là on va créer un fichier "scripts.js" contenant :
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


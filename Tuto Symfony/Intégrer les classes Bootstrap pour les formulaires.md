# Vous pouvez intégrer automatiquement les classes de formulaire de Bootstrap, pour celà il suffit de rajouter à votre fichier config/packages/twig.yaml :
```
twig:
    default_path: '%kernel.project_dir%/templates'
    form_themes: ['bootstrap_5_layout.html.twig']

when@test:
    twig:
        strict_variables: true
```

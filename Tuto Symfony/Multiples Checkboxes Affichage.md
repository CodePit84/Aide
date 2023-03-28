## Vous pouvez intégrer automatiquement les classes de formulaire de Bootstrap, pour celà il suffit de rajouter à votre fichier config/packages/twig.yaml :
```
twig:
    default_path: '%kernel.project_dir%/templates'
    form_themes: ['bootstrap_5_layout.html.twig']

when@test:
    twig:
        strict_variables: true
```
Mettre la class checkbox-inline dans label_attr et attr
Dans le FormulaireType.php :
``` 
    ->add('skill', EntityType::class, [
            'class' => Skill::class,
            'query_builder' => function (SkillRepository $r) {
                return $r->createQueryBuilder('i')
                    // ->where('i.user_id = :user_id')
                    ->orderBy('i.name', 'ASC');
                    // ->setParameter('id', $this->token->getToken()->getUser());
                    // ->setParameter('user_id', $this->token->getToken()->getUser()->getId());
            },
            'label' => 'Mes compétences',
            
            // Pour les checkboxes bootstrap !!!!!!!!!!
            'label_attr' => [
                'class' => 'checkbox-inline'
            ],
            // Pour les checkboxes bootstrap !!!!!!!!!!
            'attr' => [
                'class' => 'checkbox-inline'
            ],
            // et rajouter "form_themes: ['bootstrap_5_layout.html.twig']"" dans config/twig.yaml
            'choice_label' => 'name',
            'multiple' => true,
            'expanded' => true,
        ])
``` 

Dans le tig appeller juste le formulaire de manière simple :
``` 
{{ form(form) }}
``` 
Voilà :)

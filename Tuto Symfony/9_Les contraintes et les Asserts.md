## La validation des entités est très importante aux niveau de nos Entités et au niveau de nos formulaires !
On va les mettre en place !

## 1. Aux niveau des entités :
- src/Entity/Movement.php :

On va rajouter un :
``` 
use Symfony\Component\Validator\Constraints as Assert;
``` 
et les contraintes sur nos champs, ici juste au niveau de notre "montant" et de la "date" qui ne devront pas être null :

``` 
    #[ORM\Column]
    #[Assert\NotNull()]
    private ?int $amount = null;

    #[ORM\Column(type: Types::DATE_MUTABLE)]
    #[Assert\NotNull()]
    private ?\DateTimeInterface $date = null;
``` 

Et on va rajouter les mêmes contraintes au niveau de notre formulaire, pour éviter une requête inutile à bdd et filtrer déjà en amont au niveau du formulaire src/Form/MovementFormType.php :

On va aussi rajouter un :
``` 
use Symfony\Component\Validator\Constraints as Assert;
``` 
et des contraintes au niveau de nos 2 champs :
``` 
           ->add('amount', MoneyType::class, [
                'attr' => [
                    'class' => 'form-control'
                ],
                'label' => 'Montant',
                'divisor' => 100,
                'constraints' => [
                    new Assert\NotNull()
                ]
            ])
            ->add('date', DateType::class, [
                'widget' => 'single_text',
                'html5' => false,
                'attr' => [
                    'class' => 'form-control js-datepicker'
                ],
                'label' => 'Date',
                'constraints' => [
                    new Assert\NotNull()
                ]
            ])
``` 
Voilà pour les mouvements !

- src/Entity/User.php :

On va rajouter un :
``` 
use Symfony\Component\Validator\Constraints as Assert;
``` 
et les contraintes sur nos champs :

``` 

``` 

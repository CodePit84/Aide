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



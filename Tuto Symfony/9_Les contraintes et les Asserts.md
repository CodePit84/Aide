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
use Symfony\Component\Validator\Constraints as Assert;

#[ORM\Entity(repositoryClass: UserRepository::class)]
#[UniqueEntity(fields: ['email'], message: 'Il existe déjà un compte avec cet e-mail')]
class User implements UserInterface, PasswordAuthenticatedUserInterface
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 180, unique: true)]
    #[Assert\Email()]
    #[Assert\Length(min:2, max : 180)]
    private ?string $email = null;

    #[ORM\Column]
    #[Assert\NotNull()]
    private array $roles = [];

    private ?string $plainPassword = null;

    /**
     * @var string The hashed password
     */
    #[ORM\Column]
    #[Assert\NotBlank()]
    private ?string $password = null;

    #[ORM\Column(length: 100)]
    #[Assert\NotBlank()]
    private ?string $name = null;

    #[ORM\Column(type: 'datetime_immutable', options: ['default' => 'CURRENT_TIMESTAMP'])]
    #[Assert\NotNull()]
    private ?\DateTimeImmutable $createdAt = null;

    #[ORM\OneToMany(mappedBy: 'user', targetEntity: Movement::class, orphanRemoval: true)]
    #[Assert\NotNull()]
    private Collection $movements;

    public function __construct()
    {
        $this->createdAt = new \DateTimeImmutable();
        $this->movements = new ArrayCollection();
    }
``` 
Et on va rajouter les mêmes contraintes au niveau de nos formulaires src/Form/RegistrationFormType.php et src/Form/EditUserFormType.php et src/Form/UserPasswordFormType.php :

On va aussi rajouter un :
``` 
use Symfony\Component\Validator\Constraints as Assert;
``` 
et







et puisqu'on à toucher à nos entitées, il ne faut pas oublié de faire un :
``` 
symfony console make:migration
``` 
et
``` 
symfony console d:m:m
``` 

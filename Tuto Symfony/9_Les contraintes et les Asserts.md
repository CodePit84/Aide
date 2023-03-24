## La validation des entités est très importante aux niveau de nos Entités et au niveau de nos formulaires !
On va les mettre en place !

Aux niveau des entités :
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
et src/Form/RegistrationFormType.php :
``` 
<?php

namespace App\Form;

use App\Entity\User;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Validator\Constraints\IsTrue;
use Symfony\Component\Validator\Constraints\Length;
use Symfony\Component\Validator\Constraints\NotBlank;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Component\Form\Extension\Core\Type\CheckboxType;
use Symfony\Component\Form\Extension\Core\Type\PasswordType;
use Symfony\Component\Form\Extension\Core\Type\RepeatedType;

use Symfony\Component\Validator\Constraints as Assert;

class RegistrationFormType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('email', EmailType::class, [
                'attr' => [
                    'class' => 'form-control',
                    'minlenght' => '2',
                    'maxlenght' => '180'
                ],
                'label' => 'E-mail',
                'label_attr' => [
                    'class' => 'form-label'    
                ],
                'constraints' => [
                    new Assert\NotBlank(),
                    new Assert\Email(),
                    new Assert\Length(['min' => 2, 'max' => 180])
                ]
            ])
            ->add('name', TextType::class, [
                'attr' => [
                    'class' => 'form-control mb-3',
                    'minlenght' => '2',
                    'maxlenght' => '50',

                ],
                'label' => 'Nom / Prénom',
                'label_attr' => [
                    'class' => 'form-label'    
                ],
                'constraints' => [
                    new Assert\NotBlank(),
                    new Assert\Length(['min' => 2, 'max' => 50])
                ]
            ])
            ->add('agreeTerms', CheckboxType::class, [
                'mapped' => false,
                'constraints' => [
                    new IsTrue([
                        'message' => 'Vous devez accepter nos conditions.',
                    ]),
                ],
                'attr' => [
                    'class' => 'form-check-input mx-1 mt-3'
                ],
                'label' => 'J\'accepte l\'usage de mes coordonnées pour l\'utilisation de l\'application. Nous ne partagerons jamais données.',
                'label_attr' => [
                    'class' => 'form-label mt-3'    
                ]
            ])
            ->add('plainPassword', RepeatedType::class, [
                // instead of being set onto the object directly,
                // this is read and encoded in the controller
                'type' => PasswordType::class,
                'first_options' => [
                    'attr' => [
                        'class' => 'form-control'
                    ],
                    'label' => 'Mot de passe',
                    'label_attr' => [
                        'class' => 'form-label'    
                    ]
                ],
                'second_options' => [
                    'attr' => [
                        'class' => 'form-control'
                    ],
                    'label' => 'Confirmation du mot de passe',
                    'label_attr' => [
                        'class' => 'form-label mt-3'    
                    ]
                ],
                'invalid_message' => 'Les mots de passe ne correspondent pas.',
                'mapped' => false,
                'attr' => [
                    'autocomplete' => 'new-password',
                    'class' => 'form-control'
                ],
                'constraints' => [
                    new NotBlank([
                        'message' => 'Veuillez saisir un mot de passe',
                    ]),
                    new Length([
                        'min' => 6,
                        'minMessage' => 'Votre mot de passe doit comporter au moins {{ limit }} caractères',
                        // max length allowed by Symfony for security reasons
                        'max' => 4096,
                    ]),
                ],
                'label' => 'Mot de passe'
            ])
        ;
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => User::class,
        ]);
    }
}
``` 

et src/Form/EditUserFormType.php :
``` 
<?php

namespace App\Form;

use App\Entity\User;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Component\Form\Extension\Core\Type\PasswordType;

use Symfony\Component\Validator\Constraints as Assert;

class EditUserFormType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
        ->add('name', TextType::class, [
            'attr' => [
                'class' => 'form-control mb-3',
                'minlenght' => '2',
                'maxlenght' => '50',

            ],
            'label' => 'Nom / Prénom',
            'label_attr' => [
                'class' => 'form-label'    
            ],
            'constraints' => [
                new Assert\NotBlank(),
                new Assert\Length(['min' => 2, 'max' => 50])
            ]
        ])
        ->add('email', EmailType::class, [
            'attr' => [
                'class' => 'form-control',
                'minlenght' => '2',
                'maxlenght' => '180'
            ],
            'label' => 'E-mail',
            'label_attr' => [
                'class' => 'form-label'    
            ],
            'constraints' => [
                new Assert\NotBlank(),
                new Assert\Email(),
                new Assert\Length(['min' => 2, 'max' => 180])
            ]
        ])
            ->add('plainPassword', PasswordType::class, [
                'attr' => [
                    'class' => 'form-control'
                ],
                'label' => 'Mot de passe',
                'label_attr' => [
                    'class' => 'form-label'
                ]
            ])
        ;
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => User::class,
        ]);
    }
}
``` 

et src/Form/UserPasswordFormType.php :
``` 
<?php

namespace App\Form;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Validator\Constraints as Assert;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;

use Symfony\Component\Form\Extension\Core\Type\PasswordType;
use Symfony\Component\Form\Extension\Core\Type\RepeatedType;

use Symfony\Component\Validator\Constraints\Length;
use Symfony\Component\Validator\Constraints\NotBlank;

class UserPasswordFormType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('plainPassword', PasswordType::class, [
                'attr' => [
                    'class' => 'form-control'
                ],
                'label' => 'Mot de passe',
                'label_attr' => [
                    'class' => 'form-label  mt-4'
                ]
            ])
            ->add('newPassword', RepeatedType::class, [
                'type' => PasswordType::class,
                'first_options' => [
                    'constraints' => [new Assert\NotBlank()],
                    'attr' => [
                        'class' => 'form-control'
                    ],
                    'label' => 'Nouveau mot de passe',
                    'label_attr' => [
                        'class' => 'form-label  mt-4'
                    ]
                ],
                'second_options' => [
                    'constraints' => [new Assert\NotBlank()],
                    'attr' => [
                        'class' => 'form-control'
                    ],
                    'label' => 'Confirmation du nouveau mot de passe',
                    'label_attr' => [
                        'class' => 'form-label  mt-4'
                    ]
                    ],
                'invalid_message' => 'Les mots de passe ne correspondent pas.',
                'mapped' => false,
                'attr' => [
                    'autocomplete' => 'new-password',
                    'class' => 'form-control'
                ],
                'constraints' => [
                    new NotBlank([
                        'message' => 'Veuillez saisir un mot de passe',
                    ]),
                    new Length([
                        'min' => 6,
                        'minMessage' => 'Votre mot de passe doit comporter au moins {{ limit }} caractères',
                        // max length allowed by Symfony for security reasons
                        'max' => 4096,
                    ]),
                ]
            ])
            ->add('submit', SubmitType::class, [
                'attr' => [
                    'class' => 'btn btn-primary mt-4'
                ],
                'label' => 'Changer mon mot de passe'
            ]);
    }
}
``` 

et puisqu'on à toucher à nos entitées, il ne faut pas oublié de faire un :
``` 
symfony console make:migration
``` 
et
``` 
symfony console d:m:m
``` 

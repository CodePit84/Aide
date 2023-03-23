# Mise en place de l'edition d'un utilisateur et de la modification de son mot de passe (sur une autre page) :

## 1. Rajouter dans l'Entity User.php :
``` private ?string $plainPassword = null; ```

au dessus de :
``` 
    /**
     * @var string The hashed password
     */
    #[ORM\Column]
    private ?string $password = null;
``` 


et les get / set :
```
    /**
     * Get the value of plainPassword
     */
    public function getPlainPassword()
    {
        return $this->plainPassword;
    }

    /**
     * Set the value of plainPassword
     *
     * @return  self
     */
    public function setPlainPassword($plainPassword)
    {
        $this->plainPassword = $plainPassword;

        return $this;
    }
```

ainsi on aura User.php :
``` 
<?php

namespace App\Entity;

use App\Repository\UserRepository;
use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\Common\Collections\Collection;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Bridge\Doctrine\Validator\Constraints\UniqueEntity;
use Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface;
use Symfony\Component\Security\Core\User\UserInterface;

#[ORM\Entity(repositoryClass: UserRepository::class)]
#[UniqueEntity(fields: ['email'], message: 'There is already an account with this email')]
class User implements UserInterface, PasswordAuthenticatedUserInterface
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 180, unique: true)]
    private ?string $email = null;

    #[ORM\Column]
    private array $roles = [];

    private ?string $plainPassword = null;

    /**
     * @var string The hashed password
     */
    #[ORM\Column]
    private ?string $password = null;

    #[ORM\Column(length: 100)]
    private ?string $name = null;

    #[ORM\Column(type: 'datetime_immutable', options: ['default' => 'CURRENT_TIMESTAMP'])]
    private ?\DateTimeImmutable $createdAt = null;

    #[ORM\OneToMany(mappedBy: 'user', targetEntity: Movement::class, orphanRemoval: true)]
    private Collection $movements;

    public function __construct()
    {
        $this->createdAt = new \DateTimeImmutable();
        $this->movements = new ArrayCollection();
    }
    
    public function getId(): ?int
    {
        return $this->id;
    }

    public function getEmail(): ?string
    {
        return $this->email;
    }

    public function setEmail(string $email): self
    {
        $this->email = $email;

        return $this;
    }

    /**
     * A visual identifier that represents this user.
     *
     * @see UserInterface
     */
    public function getUserIdentifier(): string
    {
        return (string) $this->email;
    }

    /**
     * @see UserInterface
     */
    public function getRoles(): array
    {
        $roles = $this->roles;
        // guarantee every user at least has ROLE_USER
        $roles[] = 'ROLE_USER';

        return array_unique($roles);
    }

    public function setRoles(array $roles): self
    {
        $this->roles = $roles;

        return $this;
    }

    /**
     * Get the value of plainPassword
     */
    public function getPlainPassword()
    {
        return $this->plainPassword;
    }

    /**
     * Set the value of plainPassword
     *
     * @return  self
     */
    public function setPlainPassword($plainPassword)
    {
        $this->plainPassword = $plainPassword;

        return $this;
    }
    
    /**
     * @see PasswordAuthenticatedUserInterface
     */
    public function getPassword(): string
    {
        return $this->password;
    }

    public function setPassword(string $password): self
    {
        $this->password = $password;

        return $this;
    }

    /**
     * @see UserInterface
     */
    public function eraseCredentials()
    {
        // If you store any temporary, sensitive data on the user, clear it here
        // $this->plainPassword = null;
    }

    public function getName(): ?string
    {
        return $this->name;
    }

    public function setName(string $name): self
    {
        $this->name = $name;

        return $this;
    }

    public function getCreatedAt(): ?\DateTimeImmutable
    {
        return $this->createdAt;
    }

    public function setCreatedAt(\DateTimeImmutable $createdAt): self
    {
        $this->createdAt = $createdAt;

        return $this;
    }

    /**
     * @return Collection<int, Movement>
     */
    public function getMovements(): Collection
    {
        return $this->movements;
    }

    public function addMovement(Movement $movement): self
    {
        if (!$this->movements->contains($movement)) {
            $this->movements->add($movement);
            $movement->setUser($this);
        }

        return $this;
    }

    public function removeMovement(Movement $movement): self
    {
        if ($this->movements->removeElement($movement)) {
            // set the owning side to null (unless already changed)
            if ($movement->getUser() === $this) {
                $movement->setUser(null);
            }
        }

        return $this;
    }
}
``` 

## 2 . Créer un formulaire à la main d'édition du profil EditUserFormType.php
Nouveau Fichier dans src/Form

(on ne s'occupera pas de la modification du mot de passe, on le verra dans une autre méthode, on ne s'occupera que de la modification du profil de l'utilisateur, le mot de passe demandé servira seulement à valider les changements)


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

class EditUserFormType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('name', TextType::class, [
                'attr' => [
                    'class' => 'form-control mb-3'
                ],
                'label' => 'Pseudo'    
            ])
            ->add('email', EmailType::class, [
                'attr' => [
                    'class' => 'form-control'
                ],
                'label' => 'E-mail'    
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
## 3. On créera à la main également le fichier UserController.php 
dans src/Controller et on rajoutera la méthode d'édition :
``` 
<?php

namespace App\Controller;

use App\Entity\User;
use App\Form\EditUserFormType;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Security;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;


class UserController extends AbstractController
{
    #[Security("is_granted('ROLE_USER') and user === choosenUser")]
    #[Route('/edit/user/{id}', name: 'app_edit_user')]
    public function editUser(User $choosenUser, Request $request, UserPasswordHasherInterface $hasher, EntityManagerInterface $manager): Response
    {
        // Avec le "Security" devant la "Route" les 2 conditions (if) suivantes ne sont plus nécessaires :

        // Si l'utilisateur n'est pas connecté le renvoyer sur le formulaire de connexion
        // if(!$this->getUser()) {
        //     return $this->redirectToRoute('app_login');
        // }

        // Vérification de l'utilisateur actif
        // Si l'utilisateur actif veut modifier un autre utilisateur le renvoyer sur la page d'accueil
        // if($this->getUser() !== $choosenUser){
        //     return $this->redirectToRoute('home');
        // }

        // Si tout est OK on lui passe le formulaire
        
        $form = $this->createForm(EditUserFormType::class, $choosenUser);
        $form->handleRequest($request);

        if($form->isSubmitted() && $form->isValid()) {
            if ($hasher->isPasswordValid($choosenUser, $form->getData()->getplainPassword())) {
                $user = $form->getData();
                $manager->persist($user);
                $manager->flush();

                $this->addFlash(
                    'success',
                    'Les informations de votre compte ont bien été modifiées.'
                );

                return $this->redirectToRoute('app_movement_user', array('id' => $choosenUser->getId()));
            } else {
                $this->addFlash(
                    'warning',
                    'Le mot de passe renseigné est incorrect.'
                );
            }
        }

        return $this->render('registration/edit_user.html.twig', [
            'editUserForm' => $form->createView(),
        ]);
    }        
}
```
## 4. Nous devons créer le rendu du formulaire : edit_user.html.twig
au niveau du dossier templates et par exemple dans le sous-dossier registration

templates/registration/edit_user.html.twig :

```
{% extends 'base.html.twig' %}

{% block title %}Modification du profil utilisateur{% endblock %}

{% block body %}
<section class="container">
    <div class="row">
        <div class="col">
            <h1>Modification du profil utilisateur</h1>

            {{ form_start(editUserForm) }}
                <fieldset class="mb-3">
                    <legend>Mon identité</legend>
                    {{ form_row(editUserForm.name) }}
                    {{ form_row(editUserForm.email) }}
                </fieldset>
                    {{ form_row(editUserForm.plainPassword) }}

                <button type="submit" class="btn btn-primary btn-lg my-3">Sauvegarder</button>
                {# <a href="{{ path('app_edit_password_user', {id: app.user.id}) }}" class="btn btn-warning">Modifier le mot de passe</a>            #}
            {{ form_end(editUserForm) }} 
        </div>
    </div>
</section>
{% endblock %}
``` 
J'ai mis en commentaire le future lien de la modification du mot de passe...

## 5. Pour l'édition (modification) du mot de passe :
On rajoute la nouvelle méthode (editPassword) dans le UserController.php
(pour plus de lisibilité je mets ci-dessous la totalité du fichier UserController.php :
```
<?php

namespace App\Controller;

use App\Entity\User;
use App\Form\EditUserFormType;
use App\Form\UserPasswordFormType;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Security;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;


class UserController extends AbstractController
{
    /**
     * This controller allow us to edit user's profile
     *
     * @param User $choosenUser
     * @param Request $request
     * @param EntityManagerInterface $manager
     * @return Response
     */
    #[Security("is_granted('ROLE_USER') and user === choosenUser")]
    #[Route('/edit/user/{id}', name: 'app_edit_user')]
    public function editUser(User $choosenUser, Request $request, UserPasswordHasherInterface $hasher, EntityManagerInterface $manager): Response
    {
        // Avec le "Security" devant la "Route" les 2 conditions (if) suivantes ne sont plus nécessaires :

        // Si l'utilisateur n'est pas connecté le renvoyer sur le formulaire de connexion
        // if(!$this->getUser()) {
        //     return $this->redirectToRoute('app_login');
        // }

        // Vérification de l'utilisateur actif
        // Si l'utilisateur actif veut modifier un autre utilisateur le renvoyer sur la page d'accueil
        // if($this->getUser() !== $choosenUser){
        //     return $this->redirectToRoute('home');
        // }

        // Si tout est OK on lui passe le formulaire
        
        $form = $this->createForm(EditUserFormType::class, $choosenUser);
        $form->handleRequest($request);

        if($form->isSubmitted() && $form->isValid()) {
            if ($hasher->isPasswordValid($choosenUser, $form->getData()->getplainPassword())) {
                $user = $form->getData();
                $manager->persist($user);
                $manager->flush();

                $this->addFlash(
                    'success',
                    'Les informations de votre compte ont bien été modifiées.'
                );

                return $this->redirectToRoute('app_movement_user', array('id' => $choosenUser->getId()));
            } else {
                $this->addFlash(
                    'warning',
                    'Le mot de passe renseigné est incorrect.'
                );
            }
        }

        return $this->render('registration/edit_user.html.twig', [
            'editUserForm' => $form->createView(),
        ]);
    }

    /**
     * This controller allow us to edit user's password
     *
     * @param User $choosenUser
     * @param Request $request
     * @param EntityManagerInterface $manager
     * @param UserPasswordHasherInterface $hasher
     * @return Response
     */
    #[Security("is_granted('ROLE_USER') and user === choosenUser")]
    #[Route('/edit/password_user/{id}', 'app_edit_password_user', methods: ['GET', 'POST'])]
    public function editPassword(
        User $choosenUser,
        Request $request,
        EntityManagerInterface $manager,
        UserPasswordHasherInterface $hasher
    ): Response {
        $form = $this->createForm(UserPasswordFormType::class);

        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {
            if ($hasher->isPasswordValid($choosenUser, $form->getData()['plainPassword'])) {
                $choosenUser->setPassword(
                    $hasher->hashPassword(
                        $choosenUser,
                        $form->getData()['newPassword']
                        )
                );

                $this->addFlash(
                    'success',
                    'Le mot de passe a été modifié.'
                );

                $manager->persist($choosenUser);
                $manager->flush();

                return $this->redirectToRoute('app_movement_user', array('id' => $choosenUser->getId()));
            } else {
                $this->addFlash(
                    'warning',
                    'Le mot de passe renseigné est incorrect.'
                );
            }
        }

        return $this->render('registration/edit_password.html.twig', [
            'form' => $form->createView()
        ]);
    }

}
```
## 6. Nous devons aussi créer le Fomulaire d'édition du mot de passe de l'utilisateur : UserPasswordFormType.php
Toujours dans src/Form
```
<?php

namespace App\Form;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Validator\Constraints as Assert;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;

use Symfony\Component\Form\Extension\Core\Type\PasswordType;
use Symfony\Component\Form\Extension\Core\Type\RepeatedType;

class UserPasswordType extends AbstractType
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
                'invalid_message' => 'Les mots de passe ne correspondent pas.'
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
## 7. Puis nous devons créer le rendu de ce formulaire : edit_password.html.twig
```
{% extends 'base.html.twig' %}

{% block title %}Modification du mot de passe de l'utilisateur{% endblock %}

{% block body %}
    <div class="container">
		<h1 class="mt-4">Formulaire de modification de mot de passe de l'utilisateur</h1>

		{% for message in app.flashes('warning') %}
			<div class="alert alert-warning mt-4">
				{{ message }}
			</div>
		{% endfor %}

		{{ form(form) }}
    </div>
{% endblock %}
```

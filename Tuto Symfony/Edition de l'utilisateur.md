1. Rajouter dans l'Entity User.php :
``` private ?string $plainPassword = null; ```
et les get / set :
``` /**
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
2 . Créer un formulaire d'édition du profil EditUserFormType.php 
(on ne s'occupera pas du mot de passe à modifier, le mot de passe servira à valider les changements)


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
            ->add('nickname', TextType::class, [
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
            ->add('plainpassword', PasswordType::class, [
                'attr' => [
                    'class' => 'form-control'
                ],
                'label' => 'Mot de passe',
                'label_attr' => [
                    'class' => 'form-label  mt-3'
                ]
            ])
            ->add('submit', SubmitType::class, [
                'attr' => [
                    'class' => 'btn btn-primary mt-3'
                ],
                'label' => 'Valider'
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
3. Dans le UserController.php rajouter la méthode d'édition :
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
4. Pour l'edition (modification) du mot de passe :
On rajoute la nouvelle méthode (editPassword) dans le UserController.php
(pour plus de lisibilité je met ci-dessous la totalité du fichier UserController.php
```
<?php

namespace App\Controller;

use App\Entity\User;
use App\Form\EditUserFormType;
use App\Form\UserPasswordType;
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
        $form = $this->createForm(UserPasswordType::class);

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
5. Nous devons aussi créer le Fomulaire d'édition du mot de passe de l'utilisateur : UserPasswordType.php
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
6. Puis nous devons créer le rendu de ce formulaire : edit_password.html.twig
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
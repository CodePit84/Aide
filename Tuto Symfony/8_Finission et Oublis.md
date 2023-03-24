# On a quelques finissions à faire ! ;)
## 1. Definir la nouvelle route après le login / l'authentification réussie :
On va renvoyé vers la consultation des mouvements de l'utilisateur, dans la méthode onAuthenticationSuccess, on va créer la variable $userId à partir du token et modifier le new RedirectResponse.

Dans src/Security/UsersAuthenticator.php :

``` 
    public function onAuthenticationSuccess(Request $request, TokenInterface $token, string $firewallName): ?Response
    {
        if ($targetPath = $this->getTargetPath($request->getSession(), $firewallName)) {
            return new RedirectResponse($targetPath);
        }
        
        $userId = $token->getUser()->getId();

        // For example:
        // return new RedirectResponse($this->urlGenerator->generate('home'));
        return new RedirectResponse($this->urlGenerator->generate('app_movement_user', ["id" => $userId]));

        // throw new \Exception('TODO: provide a valid redirect inside '.__FILE__);
    }
```  

## 2. Modifions la vue de la page d'accueil qui n'a toujours pas été modifiée...
Modifions donc templates/home/index.html.twig :
``` 
{% extends 'base.html.twig' %}

{% block title %}SymBetcheck - Accueil{% endblock %}

{% block body %}
    <div class="container mt-4">
        <div class="jumbotron">
        <h1 class="display-4">Bienvenue sur SymBetcheck</h1>
        <p class="lead">SymBetcheck est une application qui va te permettre à tenir tes comptes à jour !</p>
        <p class="lead">Tu peux enfin enregistres tous tes mouvements (dépenses et gains) dans l'univers du Jeu (PMU / Casino / Paris sportifs / Grattages, etc.)</p>
        <hr class="my-4">
        <p>Pour commencer, rendez-vous sur la page d'inscription pour utiliser l'application.</p>
        <a class="btn btn-primary btn-lg" href="{{ path('app_register') }}" role="button">Inscription</a>
        <a class="btn btn-success btn-lg" href="{{ path('app_login') }}" role="button">Connexion</a>
        </div>
    </div>
{% endblock %}
```


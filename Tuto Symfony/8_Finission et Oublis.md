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

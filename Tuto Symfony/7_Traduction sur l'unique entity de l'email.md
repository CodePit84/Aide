## Petite traduction du message d'erreur lorsque l'on veut rentrer (dans l'édition par exemple ou l'inscription) un email déjà rentré dans la bdd :

Dans src/Entity/User.php, remplacer : 
``` 
#[UniqueEntity(fields: ['email'], message: 'There is already an account with this email')]
``` 
par :
``` 
#[UniqueEntity(fields: ['email'], message: 'Il existe déjà un compte avec cet e-mail')]
``` 

## La validation des entités est très importante aux niveau de nos Entités et au niveau de nos formulaires !
On va les mettre en place !

## 1. Aux niveau des entités :
- src/Entity/Movement.php :

On va rajouter un :
``` 
use Symfony\Component\Validator\Constraints as Assert;
``` 
et les contraintes sur nos champs :

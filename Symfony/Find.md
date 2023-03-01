# Find

find($id) te permet de récupérer un objet à partir d'un identifiant.

findBy(array()) te permet de récupérer une liste d'objets à partir des champs souhaités. Exemple : 
findBy(array('nom' => 'Symfony')) retournera une liste d'objets comportant le nom "Symfony".

findOneBy(array()) a le même comportement que findBy pour effectuer la recherche, mais ne retourne qu'un seul résultat.

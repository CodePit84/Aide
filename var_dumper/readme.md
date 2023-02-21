# Pour installer var dumper :

source : (https://symfony.com/doc/current/components/var_dumper.html)

## 1. installer d'abord "composer" :

source : (https://getcomposer.org/download/)

Tapez ligne après ligne les 4 lignes suivantes :

```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"

```

source : [Cours "Grafikart" PHP  : Autoloader 5'10](https://grafikart.fr/tutoriels/autoloader-composer-php-1144)

---

## :warning: **si erreur SSL "https" Warning, au niveau de la 1ère ligne de code** 
```php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"```

car OpenSSL n'est pas installer donc :
editer "php.ini" et décommenter : 

```extension=openssl```

---

## 2. initialiser le projet :
```
php composer.phar init
``` 

#### Laissez par défaut ou vide (sauf si partage sur le web ;) )
 
#### répondez "No" aux 2 questions pour definir les dépendances.

#### Add PSR-4 laissez "src/"

#### et "yes" à la génération (composer.json) 

## 3. Tapez :
```
php composer.phar dump-autoload
```
###### ou
###### ```
composer dump-autoload
```

## 4. installer "var_dumper" :

```
composer require --dev symfony/var-dumper
```

source : [Cours "Grafikart" PHP  : Librairies Tierce 9'20](https://grafikart.fr/tutoriels/composer-require-1146)

## 5. rajoutez en haut de votre fichier PHP (où le dump() ou le dd() sera utilisé) :
```
require __DIR__.'/vendor/autoload.php';
```

Pour installer var dumper :

(https://symfony.com/doc/current/components/var_dumper.html)

installer d'abord "composer" :

voir le site : (https://getcomposer.org/download/)

```php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"```

[Cours "Grafikart" PHP  : Autoloader 5'10]

si erreur SSL "https" Warning, au niveau de la 1ère ligne de code 
(php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');")
car OpenSSL n'est pas installer donc :
editer "php.ini" et décommenter 
```extension=openssl```

puis installer "var_dumper" :

```composer require --dev symfony/var-dumper```

[Cours "Grafikart" PHP  : Librairies Tierce


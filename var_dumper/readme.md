Pour installer var dumper :

(https://symfony.com/doc/current/components/var_dumper.html)

installer d'abord "composer" :

voir le site : (https://getcomposer.org/download/)




[Cours "Grafikart" PHP  : Autoloader 5'10]

si erreur SSL "https" Warning, au niveau de la 1ère ligne de code 
```php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"```
car OpenSSL n'est pas installer donc :
editer "php.ini" et décommenter : 
```extension=openssl

puis installer "var_dumper" :

```composer require --dev symfony/var-dumper```

[Cours "Grafikart" PHP  : Librairies Tierce


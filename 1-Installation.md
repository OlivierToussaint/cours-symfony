## Installation

Nous allons dans un premier temps ouvrir la console. Nous allons à la racine du dossier qui va nous servir pour notre exercice:

```bash
# composer create-projet symfony/website-skeleton blog
```

À partir de ce moment-là nous avons la structure de notre futur projet symfony.
Regardons cela de plus près:

* **Bin** : On y trouve le fichier console & phpunit, nous reviendrons très vite sur l’utilisation de console.

* **Config** : On y trouve tous les fichiers de configuration du framework que ce soit pour votre environnement de production ou bien de développement.

* **Public** : On y trouve tous vos fichiers dits public, c’est-à-dire accessible par le navigateur. Il Comporte donc un index.php qui va s’occuper de rooter vos données, vos fichiers css, js, et images.

* **Src** : Le répertoire qui contiendra vos controllers, vos entités et vos services.

* **Templates** : On y trouve tous les fichiers de twig.

* **Translations** : Tous fichiers de langue seront ici.

* **Var** : On y trouve les fichiers de cache et vos logs.

* **Vendor** : C'est le répertoire qui est géré par composer.

À la racine du projet nous allons nous intéresser à deux fichiers:

* **.env** : C'est dans ce fichier que seront nos données pour connecter la base de données pour dire si le projet est en prod ou en dev ainsi que différentes choses comme switfmailer.

* **composer.json** : Tous les paquets dont nous avons besoin pour notre projet.

Nous allons ensuite démarrer notre serveur de développement avec deux commandes, soit le serveur interne de php, soit le raccourci par symfony

```bash
# php -S 127.0.0.1:8000 -t public
# php bin/console server:run
```

Nous accédons à la page qui nous prévient que nous avons bien installé la dernière version de symfony.
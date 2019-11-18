## L'environnement 

Sur votre poste de travail, il faut vérifier que composer et php sont installés
 
```bash
# php -v
PHP 7.3.11 (cli) (built: Oct 24 2019 11:29:00) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.11, Copyright (c) 1998-2018 Zend Technologies
    with Xdebug v2.7.2, Copyright (c) 2002-2019, by Derick Rethans
    with Zend OPcache v7.3.11, Copyright (c) 1999-2018, by Zend Technologies
```
```bash
# composer -v
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 1.9.0 
```

## Installation 

Nous allons ouvrir la console, et taper les lignes suivants dans notre dossier de travail :

```bash
# composer create-projet symfony/website-skeleton mon-projet
```

À partir de ce moment-là nous avons la structure de notre futur projet symfony.
Regardons cela de plus près:

* **Bin** : On y trouve le fichier console & phpunit, nous reviendrons sur l’utilisation du fichier console.

* **Config** : On y trouve tous les fichiers de configuration du framework que ce soit pour votre environnement de production ou bien de développement.

* **Public** : On y trouve tous vos fichiers publics, c’est-à-dire accessible par le navigateur. Il Comportebun index.php qui va s’occuper de rooter vos données, vos fichiers css, js, et images.

* **Src** : Le répertoire contiendra vos controllers, vos entités et vos services.

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
```
```bash
# php bin/console server:run
```
On peut utiliser le binary de symfony disponible sur le site officiel ou bien un environnement docker



## Le controller

Le rôle du controller va être de nous retourner une réponse par rapport à une route ou une requête donnée.



```php
namespace App\Controller;

use Symfony\Component\Routing\Annotation\Route;

class HomeController
{
    /**
     * @Route("/home", name="home")
     */
    public function index()
    {
        return new Response('<html><body>Nous sommes sur la page Home</body></html>');
    }
}
```

Nous pouvons aller voir le résultat sur notre page en y accéder avec l'url ```home```

Pour la suite, nous allons utiliser le moteur de template ```twig``` pour l'affichage de nos données.

---

La team de symfony a créée le composant maker pour simplifier la création de fichier.

Nous allons créer un controller avec maker, symfony va nous demander de lui donner un nom. Nous le nommerons HomeController tout simplement.


```bash
# php bin/console make:controller
Choose a name for your controller class (e.g. FiercePuppyController):
 > 
```

Une fois cette commande faite, nous pouvons observer qu’il crée deux fichiers: un fichier dans src/Controller et un autre dans /template.

Nous allons regarder notre nouveau controller:

```php
namespace App\Controller;

use Symfony\Component\Routing\Annotation\Route;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;


class BlogController extends AbstractController
{
    /**
     * @Route("/home", name="home")
     */
    public function index()
    {
        return $this->render('blog/index.html.twig', [
            'controller_name' => 'BlogController',
        ]);
    }
}
```

Au-dessus de la méthode index, nous pouvons observer une annotation qui est @Route (<https://symfony.com/doc/current/routing.html>), elle permet de définir la route d'accès à notre méthode. 

La valeur de retour de ce controller est un templates html avec en paramètre ```controller_name```. 

Symfony nous mâche le travail, notre BlogController extend de Controller ce qui permet des raccourcis tels que `$this->render`. Elle permet directement de retourner le rendu de notre template twig en lui passant différents paramètres. 

## Twig 

Nous allons au fichier précréé `home/index.html.twig` par maker

{% raw %} 
```twig
{% extends 'base.html.twig' %}

{% block title %}Hello {{ controller_name }}!{% endblock %}

{% block body %}
 ...
{% endblock %}
```
{% endraw %}
 
Nous retrouvons notre variable qui est `controller_name` qui est entourée d'accolades.

Nous pouvons observer différents `block`, c'est ce que l'on va trouver dans le fichier qu'il extend `base.html.twig`. Dans ce fichier il y a tout simplement la structure de base de notre site.

Pour le blog nous allons la remanier, nous allons y rajouter bootstrap, mais juste le css.

Ca donne ça 

{% raw %} 
```twig
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Bienvenue sur notre site{% endblock %}</title>
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css" integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
        {% block stylesheets %}{% endblock %}
    </head>
    <body>
        {% block body %}{% endblock %}
        {% block javascripts %}{% endblock %}
    </body>
</html>
```
{% endraw %}


C'est dans ce fichier que nous allons créer nos blocks que l'on remplira par des données.

Plus tard nous pourrons avoir besoin des variables globales telles que `{{ app.request }}`, `{{ app.session }}`, `{{app.user }}`

Twig à une documentation juste pour lui <https://twig.symfony.com/doc/2.x/>.


## Post & Get

Pour récuperer les données POST & GET nous allons utiliser l'objet Request de symfony.

```php
    /**
     * @Route("/home", name="home")
     */
    public function index(Request $request)
    {
        return $this->render('home/index.html.twig', [
            'controller_name' => 'HomeController',
            'variable' => $request->get('variable')
        ]);
    }
```

Symfony permet d'injecter automatiquement dans la méthode les classes dont nous avons besoin, si cela vous intéressse référez-vous à la documentation <https://symfony.com/doc/current/service_container/autowiring.html>. Nous allons utiliser régulièrement cette méthode pour injecter les services (ou autres, ici on peut voir que c'est Request) dans nos controllers.

Pour comprendre en profondeur le système de container et d'autowiring je vous conseille la vidéo de Lior CHAMLA : <https://www.youtube.com/watch?v=frAXgi9D6fo>

Dans le twig nous allons rajouter l'affichage de cette variable

{% raw %} 
```twig
{{ variable }}
```
{% endraw %}


Si nous accédons à l'url avec le paramêtre ```$_GET``` : https://localhost/home?variable=test, la variable est affiché dans le template.

Plus simple nous pouvons définir le paramêtre ```$_GET``` dans la route

```php
    /**
     * @Route("/home/{variable}", name="home")
     */
    public function index(string $variable)
    {
        return $this->render('home/index.html.twig', [
            'controller_name' => 'HomeController',
            'variable' => $variable
        ]);
    }
```



## Intégration de Bootstrap

Document : <https://symfony.com/doc/current/form/bootstrap4.html>

Pour relier notre blog au framework css bootstrap, nous allons devoir chercher bootstrap.css, pour faciliter les choses nous allons prendre le CDN

https://www.bootstrapcdn.com

Nous allons intégrer cette ligne à notre balise <head> qui se trouve dans base.html.twig

{% raw %} 
```twig
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Welcome!{% endblock %}</title>
        {% block stylesheets %}{% endblock %}
        <link rel="stylesheet" type="text/css" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3/css/bootstrap.min.css">
    </head>
```
{% endraw %} 


Nous allons après dans le fichier de configuration de twig qui se trouve dans config/package/ rajoutez cette ligne
 
```
    form_themes: ['bootstrap_4_layout.html.twig']
```

Maintenant vos form prennent le thème de Symfony avec les class=‘form-control’ et autre.

Pour faciliter la suite, nous allons la navbar de symfony. (<https://getbootstrap.com/docs/4.3/components/navbar/>)

On va prendre la première qui se trouve dans la documentation et rajouter cela dans notre base.html.twig

Dans le menu du haut je souhaite avoir une page de présentation qui sera gérée dans `HomeController`, un liens vers la liste des Articles et un lien vers la liste des Catégories

Pour rendre le menu plus ergonomique, nous allons faire en sorte de rendre actif le menu sur lequel nous sommes actuellement

```twig
<li class="nav-item {% if app.request.get('_route') == 'home' %}active{% endif %}">
	<a class="nav-link" href="{{ path('home') }}">Home</a>
</li>
```

Le `app.request.get('_route')` va nous permettre d'avoir le nom de la route actuelle et si jamais il correspond à la route souhaitée alors nom mettons active dans la class du li


## ORM

Symfony utilise par défaut l'orm doctrine. Laraval utilise Eloquent par exemple.

Si vous ne savez pas ce qu'est un ORM, prenez quelques instants pour aller vous documentez sur la chose.

Pour utiliser cette ORM, nous allons partir sur une base sqlite, que nous allons définir dans le `.env`, si vous avez un serveur mysql d'installer vous pouvez utilisez la deuxième syntaxe 

```
DATABASE_URL=sqlite:///%kernel.project_dir%/var/data.db
```
ou

```
DATABASE_URL=mysql://root:secret@db:3306/project
```

Nous allons créer nos entités, pour cela rien de plus simple, nous passons toujours par la commande make.

Le but de cet excercice est de faire un blog, ainsi nous allons créer une entités Article

### Création des entités
Nous allons créer une entité avec la commande suivante :

```bash
# php bin/console make:entity
```

La console va nous poser un certain nombre de questions comme nous pouvons le voir ci-dessous:

```
Class name of the entity to create or update (e.g. BraveElephant):
 > Article

 created: src/Entity/Article
 created: src/Repository/ArticleRepository.php
 
 Entity generated! Now let's add some fields!
 You can always add more fields later manually or by re-running this command.

 New property name (press <return> to stop adding fields):
 > title

 Field type (enter ? to see all types) [string]:
 > 

 Field length [255]:
 > 

 Can this field be null in the database (nullable) (yes/no) [no]:
 > 

 updated: src/Entity/Article.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 > content

 Field type (enter ? to see all types) [string]:
 > text

 Can this field be null in the database (nullable) (yes/no) [no]:
 > 

 updated: src/Entity/Article.php

  Add another property? Enter the property name (or press <return> to stop adding fields):
 >
           
  Success! 
           

 Next: When you're ready, create a migration with make:migration
```


La console a créé deux fichiers:

Le fichier dans `Entity` sera notre objet avec des getters et setters

```php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity(repositoryClass="App\Repository\AuthorRepository")
 */
class Article
{
    /**
     * @ORM\Id()
     * @ORM\GeneratedValue()
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(type="string", length=255)
     */
    private $title;

    /**
     * @ORM\Column(type="text")
     */
    private $content;


    public function getId()
    {
        return $this->id;
    }

    public function getTitle(): ?string
    {
        return $this->title;
    }

    public function setTitle(string $title): self
    {
        $this-> title = $title;

        return $this;
    }

    public function getContent(): ?string
    {
        return $this->content;
    }

    public function setContent(string $content): self
    {
        $this->content = $content;

        return $this;
    }

}
```


Nous pouvons voir dans les annotations que nous pouvons définir différents paramètres. C'est cela qui va être pris en compte pour la génération de votre base de données.

Je vous conseille d’aller voir la documentation pour voir les autres paramètres que vous pouvez mettre <https://symfony.com/doc/current/doctrine.html>

L’autre fichier dans `Repository` permettra de faire nos appels spécifiques à la base de données.

```php
namespace App\Repository;

use App\Entity\Article;
use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;
use Symfony\Bridge\Doctrine\RegistryInterface;

/**
 * @method Article|null find($id, $lockMode = null, $lockVersion = null)
 * @method Article |null findOneBy(array $criteria, array $orderBy = null)
 * @method Article[]    findAll()
 * @method Article[]    findBy(array $criteria, array $orderBy = null, $limit = null, $offset = null)
 */
class ArticleRepository extends ServiceEntityRepository
{
    public function __construct(RegistryInterface $registry)
    {
        parent::__construct($registry, Author::class);
    }

//    /**
//     * @return Author[] Returns an array of Author objects
//     */
    /*
    public function findByExampleField($value)
    {
        return $this->createQueryBuilder('a')
            ->andWhere('a.exampleField = :val')
            ->setParameter('val', $value)
            ->orderBy('a.id', 'ASC')
            ->setMaxResults(10)
            ->getQuery()
            ->getResult()
        ;
    }
    */

    /*
    public function findOneBySomeField($value): ?Author
    {
        return $this->createQueryBuilder('a')
            ->andWhere('a.exampleField = :val')
            ->setParameter('val', $value)
            ->getQuery()
            ->getOneOrNullResult()
        ;
    }
    */
}
```

On peut voir que par défaut nous pourrons utiliser quatre fonctions de recherche qui sont assez explicites. Le fichier créé par defaut nous livre aussi des exemples que nous pouvons faire avec query builder.

        
Pour synchroniser notre base de données à notre entité, il va faire faire des migrations

Nous n'avons plus qu'à mettre à jour notre base de données

```php
# php bin/console make:migration
# php bin/console doctrine:migrations:migrate 
```

ou 

```php
# php bin/console doctrine:schema:update —force
```

A chaque modification dans l'entité nous appliquerons ces commandes

Si jamais cette commande vous donne une erreur, c'est que votre lien avec la base de données (mysql surement) ne se fait pas.

Si vous travaillez en local et que le password de mysql est vide veillez procéder ainsi dans votre `.env` qui se trouve à la racine du projet

```
DB_USER=root
DB_PASSWORD=
DB_HOST=localhost
DB_PORT=3306
DB_NAME=blog
DATABASE_URL=mysql://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_NAME}
```

Pour rentrer des exemples en base, je vais appeler une route qui va enregistrer des objects Article. Plus tard nous le ferons avec les fixtures.


```php
	/**
     * @Route("/article/ajout", name="add_article")
     */
    public function add()
    {
        $entityManager = $this->getDoctrine()->getManager();

        for($counter = 0 ; $counter < 10 ; $counter++) {
            $article = new Article();
            $article->setTitle('Mon titre ' . mt_rand());
            $article->setContent('Mon contenu' . mt_rand());
            $entityManager->persist($article);
        }
        $entityManager->flush();
        return new Response('🥳');
    }
```

Maintenant que nous avons une base avec des données, je vais les affichés :

```php
	/**
     * @Route("/articles", name="articles")
     */
    public function index(ArticleRepository $articleRepository)
    {
        $articles = $articleRepository->findAll();

        return $this->render('article/index.html.twig', [
            'articles' => $articles
        ]);
    }
```

Puis rajouter une route pour voir le détail d'un article

```php
 	/**
     * @Route("/article/{id}", name="show_article")
     */
    public function show(Article $article)
    {
        return $this->render('article/show.html.twig', [
            'article' => $article
        ]);
    }
```

Ici symfony va faire automatiquement le liens entre ```{id}``` et l'entity ```Article```
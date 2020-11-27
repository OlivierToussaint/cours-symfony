## L'environnement 

Sur votre poste de travail, il faut vérifier que composer et php sont installés
 
```bash
# php -v
PHP 7.4.12 (cli) (built: Oct 29 2020 18:37:21) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.12, Copyright (c), by Zend Technologies
```
```bash
# composer -v
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 2.0.4 2020-10-30 22:39:11
```

## Installation 

Nous allons ouvrir la console, et taper les lignes suivants dans notre dossier de travail :

```bash
# composer create-project symfony/website-skeleton mon-projet
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

Pour l'ajout en base nous pouvons passer par des fixtures plutot que de faire ça dans un controlleur 

## Fixtures

Pour permettre à un nouveau développeur d'intégrer le projet vite et de lui permettre d'avoir des données de test nous allons utiliser un bundle qui est `doctrine-fixtures-bundle`.

Il va permettre de définir un jeu de donnée initiale

```bash
# composer require --dev doctrine/doctrine-fixtures-bundle
```
Une fois installé nous allons l'initialisé avec make

```bash
# php bin/console make:fixtures
```

Nous l'appelerons AppFixtures dedans nous allons mettre des données par defaut

```php
    public function load(ObjectManager $manager)
    {
-

        $article = new Article();
        $article->setTitle('test');
        $article->setContent('Proinde die funestis interrogationibus praestituto imaginarius iudex equitum resedit magister adhibitis aliis iam quae essent agenda praedoctis, et adsistebant hinc inde notarii, quid quaesitum esset, quidve responsum, cursim ad Caesarem perferentes, cuius imperio truci, stimulis reginae exsertantis aurem subinde per aulaeum, nec diluere obiecta permissi nec defensi periere conplures.');
        $manager->persist($article);
        $manager->flush();
    }
```

Nous utilisons l'objectManager, qui va nous servir à faire la liaison de l'object avec la base, nous y reviendrons plus en details sur le chapitres suivants.
Une fois notre fichier rempli nous allons charger ces données en base.

```
~> php bin/console doctrine:fixtures:load
```

Nous pouvons voir que notre base est remplie.

De ce fait il faudrait rajouter quelque chose dans notre `AppFixtures`, le password.

Pour encoder le password nous aurons besoin d'injecter `UserPasswordEncoderInterface` puis d'utiliser `encodePassword`

Cela nous donnera:

```
	public function __construct(UserPasswordEncoderInterface $encoder)
    {
        $this->encoder = $encoder;
    }

    public function load(ObjectManager $manager)
    {
        $user = new User();
        $user->setEmail('john@doe.fr');
        $password = $this->encoder->encodePassword($user, 'test');
        $user->setPassword($password);
        $user->setName('John Doe');
        $manager->persist($user);
```

Pour nous permettre de changer les données dans la base, nous allons passer par des formulaires

## Les Forms

Documentation : https://symfony.com/doc/current/forms.html

Nous pouvons créer des formulaires via le template ou bien nous pouvons laisser symfony faire tout ça.

```bash
# php bin/console make:form
```
Nous allons lui donner l’entity sur lequel elle va se calquer, puis ensuite nous choisirons de rajouter le bouton submit ou pas dans ce formType.

```php
public function buildForm(FormBuilderInterface $builder, array $options)
{
    $builder
        ->add('title')
        ->add(‘content');
    ;
}
```
Nous pourrions dans ce fichier personnaliser les labels ou autres. Pour tout cela nous irons lire la documentation

```php
public function buildForm(FormBuilderInterface $builder, array $options)
{
    $builder
        ->add(‘title’, TextType::class, [
			‘label’ => ‘Notre titre’
		])
        ->add(‘content');
    ;
}
```

Ainsi, nous allons appeler ce ArticleType dans notre controller:

```php
/**
 * @Route("/article/new", name="article_new")
 */
public function index(Request $request)
{
    $article = new Article();
    $form = $this->createForm(ArticleType::class, $article);

    $form->handleRequest($request);
    if ($form->isSubmitted() && $form->isValid())
    {
        $entityManager = $this->getDoctrine()->getManager();
        $entityManager->persist($article);
        $entityManager->flush();
        $this->addFlash('success', 'Votre article est enregistré');
        return $this->redirectToRoute('articles');
    }

    return $this->render('article/new', [
        'form' => $form->createView(),

    ]);
}
```
On va décomposer :

```php
        $form->handleRequest($request);
```

On injecte tout simplement l'object `Request` pour qu'il fasse le travail tout seul, plus sérieusement la définition de la documentation de cette classe:

>Internally, the request is forwarded to the configured {@link RequestHandlerInterface} instance, which determines whether to submit the form or not.

----

```php
        if ($form->isSubmitted() && $form->isValid()) {
            $entityManager = $this->getDoctrine()->getManager();
            $entityManager->persist($article);
            $entityManager->flush();

            return $this->redirectToRoute('articles');
        }
```

Là, on vérifie que le formulaire est bien submit et que ce dernier est valide, si c'est le cas, on enregistre en base l'object `Post`

*Comment enregistre-t-on en base ?*

Nous avons vu très vite l'entity manager quand nous avons chargé nos fixtures dans l'orm. Nous allons l'utiliser pour enregistrer en base notre object.

À quoi sert cette Entity Manger ? C'est très simple vous n'avez plus à vous soucier de faire un `INSERT` ou un `UPDATE` en base, l'orm va s'occuper de tout, vous n'avez qu'à charger le Manager (à partir de `EntityManagerInterface` pour de l'injection directe) ou bien le récuperer dans votre controller avec `$this->getDoctrine()->getMananger()` 

Pour ce faire, nous allons utiliser la fonction `persist` et `flush` comme montré dans l'exemple:

Je vous renvoie à la documentation qui est vaste <https://symfony.com/doc/current/doctrine.html>

----

```php
        return $this->render('article/new.html.twig', [
            'post' => $post,
            'form' => $form->createView(),
        ]);
```
Le `$form->createView()` servira à construire notre formulaire dans twig qui est représenté par `_form.html.twig`


Dans le twig 

```twig
{{ form_start(form) }}
{{ form_errors(form) }}
{{ form_widget(form) }}
<button type="submit">Save</button>
{{ form_end(form) }}
```

On pourrait personnaliser les forms en détaillant le tout

```twig
{{ form_start(form) }}
{{ form_errors(form) }}
{{ form_widget(form.title) }}
{{ form_widget(form.content) }}
<button type="submit">Save</button>
{{ form_end(form) }}
```

Symfony nous facilite la vie en créant des "templates" à la demande pour faire un CRUD (create, read, update, delete) sur une entity

```bash
# php bin/console make:crud
```

## Security 

Documentation liée.:<https://symfony.com/doc/current/security.html>

Nous allons créer la partie sécurité qui va comporter trois parties.

1. La partie de création de l'entity User associée à sécurity pour l'encodage du password
2. La partie Authentification qui va nous permettre de se connecter sur le site
3. La partie Register qui va nous permettre d'enregistrer nos utilisateurs

```bash
# php bin/console make:user

The name of the security user class (e.g. User) [User]:
 > 

 Do you want to store user data in the database (via Doctrine)? (yes/no) [yes]:
 > 

 Enter a property name that will be the unique "display" name for the user (e.g. email, username, uuid) [email]:
 > 

 Will this app need to hash/check user passwords? Choose No if passwords are not needed or will be checked/hashed by some other system (e.g. a single sign-on server).

 Does this app need to hash/check user passwords? (yes/no) [yes]:
 > 

The newer Argon2i password hasher requires PHP 7.2, libsodium or paragonie/sodium_compat. Your system DOES support this algorithm.
You should use Argon2i unless your production system will not support it.

 Use Argon2i as your password hasher (bcrypt will be used otherwise)? (yes/no) [yes]:
 > 

 created: src/Entity/User.php
 created: src/Repository/UserRepository.php
 updated: src/Entity/User.php
 updated: config/packages/security.yaml

           
  Success! 
           

 Next Steps:
   - Review your new App\Entity\User class.
   - Use make:entity to add more fields to your User entity and then run make:migration.
   - Create a way to authenticate! See https://symfony.com/doc/current/security.html

```

On peut voir que notre entité User implémente une interface (qui définit un 'contrat' avec cette dernière) des choses à implémenter au minimun.

N.B. : Si jamais vous avez une version de Maria dB, ou Mysql qui n'accepte pas le format json, pensez à passer `$role` en array.

 
Dans `security.yaml` le make:user a rajouté un provider et un encoder

```yaml
    encoders:
        App\Entity\User:
            algorithm: argon2i
    providers:
        app_user_provider:
            entity:
                class: App\Entity\User
                property: email
```

Nous allons modifier notre entité `User` pour qu'il puisse avoir par defaut le role `ROLE_USER`

Nous allons rajouter en haut de l'entité deux constantes, puis plus bas dans le construct initialisé la variable `roles` : 

```php
class User implements UserInterface
{
    public const ROLE_USER = 'ROLE_USER';
    public const ROLE_ADMIN = 'ROLE_ADMIN';
    
    ...
        
    public function __construct()
    {
        $this->roles = [self::ROLE_USER];
    
```

Tout nouveau utilisateur aura pour role `ROLE_USER`

--

Nous allons créer le formulaire de login et le guard pour gérer notre authentification

```bash
# php bin/console make:auth
 What style of authentication do you want? [Empty authenticator]:
  [0] Empty authenticator
  [1] Login form authenticator
 > 1

 The class name of the authenticator to create (e.g. AppCustomAuthenticator):
 > App     

 Choose a name for the controller class (e.g. SecurityController) [SecurityController]:
 > 

 created: src/Security/AppAuthenticator.php
 updated: config/packages/security.yaml
 created: src/Controller/SecurityController.php
 created: templates/security/login.html.twig

           
  Success! 
           

 Next:
 - Customize your new authenticator.
 - Finish the redirect "TODO" in the App\Security\AppAuthenticator::onAuthenticationSuccess() method.
 - Review & adapt the login template: templates/security/login.html.twig.

```

Nous allons finir en mettant la redirection dans la méthode indiquée plus haut `onAuthenticationSuccess`

```php
 public function onAuthenticationSuccess(Request $request, TokenInterface $token, $providerKey)
    {
        if ($targetPath = $this->getTargetPath($request->getSession(), $providerKey)) {
            return new RedirectResponse($targetPath);
        }

        // For example : return new RedirectResponse($this->router->generate('some_route'));
        throw new \Exception('TODO: provide a valid redirect inside '.__FILE__);
    }
```

Ici c'est un Service donc nous n'étendons pas de l'AbstractController, on ne peut pas utiliser le raccourci `$this->redirectToRoute('nomdelaroute'). 

Nous pouvons copier coller la ligne commenter juste au dessus `return new RedirectResponse($this->router->generate('some_route'));`

Dans le fichier de configuration de security.yaml, le lieu se fait par `guard`

```
            guard:
                authenticators:
                    - App\Security\AppAuthenticator
```

Ce qui se trouve dans le controller Security est assez classique, je vous laisse allez regarder ce qu'il a dedans.

Maintenant que nous pouvons nous connectez, il va falloir aussi nous déconnecter.

Nous allons créer une méthode dans `SecurityController` qui va nous permettre de déclarer une route pour ce déconnecter

```php
    /**
     * @Route("/logout", name="app_logout")
     */
    public function logout()
    {
    }
```

Cette methode est vide elle nous permet seulement de déclarer la route pour permettre de la mettre dans le fichier de configuration de security. Juste en dessous de guard nous allons rajouter le liens de logout :

```yaml
            guard:
                authenticators:
                    - App\Security\AppCustomAuthenticator

            logout:
              path:   app_logout
```

Ce liens permettra à symfony de gérer la destruction de la session et des cookies associé.

--
Nous allons créer un formulaire pour créer nos utilisateurs


```bash
# php bin/console make:registration-form 

 Creating a registration form for App\Entity\User

 Do you want to add a @UniqueEntity validation annotation on your User class to make sure duplicate accounts aren't created? (yes/no) [yes]:
 > 

 Do you want to automatically authenticate the user after registration? (yes/no) [yes]:
 > 

 updated: src/Entity/User.php
 created: src/Form/RegistrationFormType.php
 created: src/Controller/RegistrationController.php
 created: templates/registration/register.html.twig

           
  Success! 
           

 Next: Go to /register to check out your new form!
 Make any changes you need to the form, controller & template.

```

Nous allons regarder le controller de plus près

```php
/**
     * @Route("/register", name="app_register")
     */
    public function register(Request $request, UserPasswordEncoderInterface $passwordEncoder, GuardAuthenticatorHandler $guardHandler, AppAuthenticator $authenticator): Response
    {
        $user = new User();
        $form = $this->createForm(RegistrationFormType::class, $user);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            // encode the plain password
            $user->setPassword(
                $passwordEncoder->encodePassword(
                    $user,
                    $form->get('plainPassword')->getData()
                )
            );

            $entityManager = $this->getDoctrine()->getManager();
            $entityManager->persist($user);
            $entityManager->flush();

            // do anything else you need here, like send an email

            return $guardHandler->authenticateUserAndHandleSuccess(
                $user,
                $request,
                $authenticator,
                'main' // firewall name in security.yaml
            );
        }

        return $this->render('registration/register.html.twig', [
            'registrationForm' => $form->createView(),
        ]);
    }
```
    
Il y a pas mal de nouvelles choses dans ce formulaire, on peut voir que nous utilisons `UserPasswordEncoderInterface` pour encoder le password donner dans le formulaire.

Nous passons par le `guardHandler` en injection de dépendance pour nous permettre de récupérer la route de la méthode `authenticateUserAndHandleSuccess`

Grace à ces trois commandes, symfony nous a permis d'initialiser un système d'authentification et de l'utiliser presque clé en main, à nous d'analyser ce dernier pour voir ce qu'il y a dedans. **Le fait d'utiliser des outils qui nous facilitent la vie, n'exclut pas de comprendre ces derniers.**

---
Pour continuer dans notre projet, nous allons laisser la possibilité d'attacher un utilisateur (User) à un article.

Il suffira de mettre à jour notre entité `Article` comme nous l'avons fait avec `Category` avec `make:entity`, rajouter le champ `User` et lui dire que c'est une relation ``ManyToOne` avec l'entité `User`

Mettre à jour votre base de donnée.

Puis nous allons rajouter une ligne dans dans notre controlleur `Article` sur la méthode `new`, nous allons utiliser `$this->getUser()` pour nous permettre d'ajouter l'utilisateur connecter à l'article.

```php
 		if ($form->isSubmitted() && $form->isValid()) {
 				// Nous lions l'utilisateur connecter à l'article créé
 				$article->setUser($this->getUser());
            
            	$entityManager = $this->getDoctrine()->getManager();
            	$entityManager->persist($article);
            	$entityManager->flush();

            	return $this->redirectToRoute('article_index');
        }
```

Par la suite nous allons restreindre les accès aux méthodes `new`,`edit` à l'aide de l'annotation `@isGranted`. Rien ne plus simple nous allons le rajouter en annotations.

```php
use Sensio\Bundle\FrameworkExtraBundle\Configuration\IsGranted;

...

    /**
     * @Route("/new", name="article_new", methods={"GET","POST"})
     * @isGranted("ROLE_USER")
     */
     
```
Si jamais nous ne sommes pas connecter avec une utilisateur qui à dans son tableau de role `ROLE_USER`, j'aurais automatiquement un accès denied ou une redirection vers le formulaire de login si je ne suis pas connecté.

Pour les besoins de la suite nous allons faire un crud sur l'entité `User` et nous rajouterons dans la navbar un liens vers l'index.

Nous souhaitons pouvoir modifier les roles d'un utilisateur pour ce faire nous allons éditer le formulaire qui est `Form\UserType.php`. Par default c'est un champs text, nous souhaitons que ce soit une select multiple.

Nous allons supprimer le champs password et modifier le champ roles

```php
	->add('roles', ChoiceType::class, [
	                'choices' => [
	                    'Administrateur' => User::ROLE_ADMIN,
	                    'Utilisateur' => User::ROLE_USER,
	                ],
	                'multiple' => true,
	                'required' => true,
	            ])
```

Si nous retournons sur l'interface, nous pouvons observé la possibilité d'attribué plusieurs ROLE à un utilisateur.


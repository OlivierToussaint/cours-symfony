## Finition du blog

Je vais définir un utilisateur admin et un utilisateur lambda

```php
class AppFixtures extends Fixture
{
    private $encoder;

    public function __construct(UserPasswordEncoderInterface $encoder)
    {
        $this->encoder = $encoder;
    }

    public function load(ObjectManager $manager)
    {
        $user = new User();
        $user->setEmail('olivier@admin.fr');
        $password = $this->encoder->encodePassword($user, 'test');
        $user->setPassword($password);
        $user->setRoles(['ROLE_USER', 'ROLE_ADMIN']);
        $manager->persist($user);

        $user = new User();
        $user->setEmail('olivier@user.fr');
        $password = $this->encoder->encodePassword($user, 'test');
        $user->setPassword($password);
        $user->setRoles(['ROLE_USER']);
        $manager->persist($user);

        $manager->flush();
    }
```

Puis allons définir des accès admin à `Article`, `Category`, `User`

```php
use Sensio\Bundle\FrameworkExtraBundle\Configuration\IsGranted;

/**
 * @Route("/article")
 * @isGranted("ROLE_ADMIN")
 */
class ArticleController extends AbstractController
{

```
Nous reprenons l'annotation isGranted que nous mettons cette fois ci au dessus de la class et non au dessus de la méthode.

Dans notre menu, nous ne souhaitons pas que l'utilisateur anonyme ou bien l'utilisateur normal puisse voir le menu d'accès à `Article`, `Category`, `User`

{% raw %} 
```twig
 {% if is_granted('ROLE_ADMIN') %}
            <li class="nav-item  {% if app.request.get('_route') == 'article_index' or app.request.get('_route') == 'article_new' %}active{% endif %}">
                <a class="nav-link" href="{{ path('article_index') }}">Article</a>
            </li>
            <li class="nav-item {% if app.request.get('_route') == 'category_index' %}active{% endif %}">
                <a class="nav-link" href="{{ path('category_index') }}">Catégorie</a>
            </li>
            <li class="nav-item {% if app.request.get('_route') == 'user_index' %}active{% endif %}">
                <a class="nav-link" href="{{ path('user_index') }}">Liste des Utilisateur</a>
            </li>
{% endif %}
```
{% endraw %} 

Je vais rajouter une date dans mon article pour savoir à quel moment il a été poster. Je mets à jour avec `make:entity` et je créais la variable `createAt`, je la définie comme datetime

A partir de ce moment il y a deux méthodes pour la définir, soit dans la méthode `new` de `ArticleController`, soit dans le `__construct` de l'entité `Article`

Controller/ArticleController.php
```php
	$article->setCreateAt(new \DateTime('now'));
	$entityManager = $this->getDoctrine()->getManager();
	$entityManager->persist($article);
	$entityManager->flush();
```
ou

Entity/Article.php
```php
    public function __construct()
    {
        $this->createAt = new \DateTime('now');
    }
```

Enfin nous allons faire notre partie frontend dans notre `HomeController`, nous définissons deux méthodes `index` et `show`, le premier pour faire la liste des articles de notre blog, le deuxième pour voir le contenu en detail d'un article.

```php
class HomeController extends AbstractController
{
    /**
     * @Route("/", name="home")
     */
    public function index(ArticleRepository $articleRepository)
    {

        return $this->render('home/index.html.twig', [
            'articles' => $articleRepository->findAll(),
        ]);
    }

    /**
     * @Route("/show/{id}", name="home_article_show")
     */
    public function show(Article $article)
    {
        return $this->render('home/article.html.twig', [
            'article' => $article,
        ]);
    }
}
```
{% raw %} 
```twig
    <div class="container">
        {% for article in articles %}
            <a href="{{ path('home_article_show', {'id' : article.id}) }}">{{ article.title }}, par {{ article.user.email }}, le {{ article.createAt|date('d/m/Y') }}</a>
        {% endfor %}
    </div>
```

```twig
    <div class="container">
       <h1>{{ article.title }}</h1>
        <h3>{{ article.user.email }}, le {{ article.createAt|date('d/m/Y') }}</h3>
        <div class="card">
            <div class="card-body">
                {{ article.content }}
            </div>
        </div>
    </div>
```

{% endraw %} 

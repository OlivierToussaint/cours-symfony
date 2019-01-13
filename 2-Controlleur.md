## Le controller

Le rôle du controller va être de nous retourner une réponse par rapport à une route ou une requête donnée.

Nous allons créer un controller avec maker, symfony va nous demander de lui donner un nom. Nous le nommerons BlogController tout simplement.

```bash
# php bin/console make:controller
```

Une fois cette commande faite nous pouvons observer qu’il crée deux fichiers: un fichier dans src/Controller et un autre dans ce nouveau dossier qui est template.

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

Dans cet exemple nous voyons que ce controller nous renvoie un templates html avec en paramètre ```controller_name```. 

Symfony nous a mâché le travail, du coup notre BlogController extend de Controller ce qui permet des raccourcis tels que `$this->render`. Elle permet directement de retourner le rendu de notre template twig en lui passant différents paramètres. 

Nous y reviendrons dans notre controller, dans la partie ORM.

---
### Pour le futur
Sans extends nous aurions pu écrire cette méthode comme cela.

```php
namespace App\Controller;

use Symfony\Component\Routing\Annotation\Route;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Response;
use Twig\Environment;

class BlogController
{
    /**
     * @Route("/home", name="home")
     */
    public function index(Environment $twig)
    {
    	 return new Response(
    	     $twig->render('blog/index.html.twig', [
                'controller_name' => 'BlogController',
            ])
    	 );

    }
}
```

Symfony permet d'injecter automatiquement dans la méthode les classes dont nous avons besoin, si cela vous intéressse référez-vous à la documentation <https://symfony.com/doc/current/service_container/autowiring.html> 

Pour la suite du cours nous utiliserons extends de Controller ainsi que leurs raccourcis mais je vous conseille de travailler comme l'exemple ci-dessus.
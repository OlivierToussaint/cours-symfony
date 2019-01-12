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

Le but de cet excercice est de faire un blog, ainsi nous allons créer deux entités, Author & Post

### Création des entités
Nous allons créer une entité avec la commande suivante :

```bash
# php bin/console make:entity
```

La console va nous poser un certain nombre de questions comme nous pouvons le voir ci-dessous:

```bash
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

use App\Entity\ Article;
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

        
Pour synchroniser notre base de donnée à notre entité, il va faire faire des migrations

Nous n'avons plus qu'a mettre à jour notre base de donnée

```php
# php bin/console make:migration
# php bin/console doctrine:migrations:migrate 
```

ou 

```php
# php bin/console doctrine:schema:update —force
```

A chaque modification dans l'entité nous appliquerons ces commandes

Si jamais cette commande vous donne une erreur, c'est que votre liens avec la base de donnée (mysql surement) ne se fait pas.

Si vous travaillez en local et que le password de mysql est vide veillez procéder ainsi dans votre `.env` qui se trouve à la racine du projet

```
DB_USER=root
DB_PASSWORD=
DB_HOST=localhost
DB_PORT=3306
DB_NAME=blog
DATABASE_URL=mysql://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_NAME}
```

Nous allons créer l'entity Post avec ces différentes columns `title`, `slug`, `body`, `createdAt`, `updatedAt`

Nous allons faire un update pour lier les deux entités:

```
➜  blog git:(master) ✗ php bin/console make:entity

 Class name of the entity to create or update (e.g. BraveElephant):
 > Post

 Your entity already exists! So let's add some new fields!

 New property name (press <return> to stop adding fields):
 > author

 Field type (enter ? to see all types) [string]:
 > ?

Main types
  * string
  * text
  * boolean
  * integer (or smallint, bigint)
  * float

Relationships / Associations
  * relation (a wizard will help you build the relation)
  * ManyToOne
  * OneToMany
  * ManyToMany
  * OneToOne

Array/Object Types
  * array (or simple_array)
  * json (or json_array)
  * object
  * binary
  * blob

Date/Time Types
  * datetime (or datetime_immutable)
  * datetimetz (or datetimetz_immutable)
  * date (or date_immutable)
  * time (or time_immutable)
  * dateinterval

Other Types
  * decimal
  * guid


 Field type (enter ? to see all types) [string]:
 > relation

 What class should this entity be related to?:
 > Author

What type of relationship is this?
 ------------ ------------------------------------------------------------- 
  Type         Description                                                  
 ------------ ------------------------------------------------------------- 
  ManyToOne    Each Blog relates to (has) one Author.                       
               Each Author can relate/has to (have) many Blog objects       
                                                                            
  OneToMany    Each Blog relates can relate to (have) many Author objects.  
               Each Author relates to (has) one Blog                        
                                                                            
  ManyToMany   Each Blog relates can relate to (have) many Author objects.  
               Each Author can also relate to (have) many Blog objects      
                                                                            
  OneToOne     Each Blog relates to (has) exactly one Author.               
               Each Author also relates to (has) exactly one Blog.          
 ------------ ------------------------------------------------------------- 

 Relation type? [ManyToOne, OneToMany, ManyToMany, OneToOne]:
 > ManyToOne

 Is the Post.author property allowed to be null (nullable)? (yes/no) [yes]:
 > no 

 Do you want to add a new property to Author so that you can access/update Post objects from it - e.g. $author->getPosts()? (yes/no) [yes]:
 > 

 A new property will also be added to the Author class so that you can access the related Post objects from it.

 New field name inside Author [posts]:
 > 

 Do you want to activate orphanRemoval on your relationship?
 A Blog is "orphaned" when it is removed from its related Author.
 e.g. $author->removePost($post)
 
 NOTE: If a Blog may *change* from one Author to another, answer "no".

 Do you want to automatically delete orphaned App\Entity\Post.php objects (orphanRemoval)? (yes/no) [no]:
 >  

 updated: src/Entity/Post.php
 updated: src/Entity/Author.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 > 

```

Il fait automatique le rajout dans les deux fichiers.

### 


C’est l’orm qui va s’occuper de la création de l’entité en base. Pour cela nous allons lister les commandes disponibles :

```
~> php bin/console doctrine
```

Vu que nous somme en developpement nous allons utilisé `doctrine:schema:update` mais ne jamais faire ça en production nous préférerons utiliser `doctrine:migrations:diff` et `doctrine:migrations:migrate`

```
~> php bin/console doctrine:schema:create
~> php bin/console doctrine:schema:update —force
```

Nous pouvons voir dans notre base de donnée que les tables et columns ont été crée

Nous allons inséré des choses à l'intérieur pour ce faire nous allons utiliser un bundle qui est `doctrine-fixtures-bundle`

```
~> composer require --dev doctrine/doctrine-fixtures-bundle
```
Une fois installé nous allons l'initialisé avec make

```
~> php bin/console make:fixtures
```

Nous l'appelerons AppFixtures dedans nous allons mettre des données par defaut

```
    public function load(ObjectManager $manager)
    {
        $author = new Author();
        $author->setEmail('john@doe.fr');
        $author->setPassword('password');
        $author->setName('John Doe');
        $author->setUsername('john doe');
        $manager->persist($author);

        $blog = new Blog();
        $blog->setTitle('test');
        $blog->setAuthor($author);
        $blog->setBody('Proinde die funestis interrogationibus praestituto imaginarius iudex equitum resedit magister adhibitis aliis iam quae essent agenda praedoctis, et adsistebant hinc inde notarii, quid quaesitum esset, quidve responsum, cursim ad Caesarem perferentes, cuius imperio truci, stimulis reginae exsertantis aurem subinde per aulaeum, nec diluere obiecta permissi nec defensi periere conplures.');
        $blog->setCreatedAt(new \DateTime());
        $blog->setUpdatedAt(new \DateTime());
        $blog->setSlug('test');
        $manager->persist($blog);
        $manager->flush();
    }
        $manager->flush();
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
        $author = new Author();
        $author->setEmail('john@doe.fr');
        $password = $this->encoder->encodePassword($author, 'test');
        $author->setPassword($password);
        $author->setName('John Doe');
        $author->setUsername('john doe');
        $manager->persist($author);
```

Nous rechargeons nos fixtures avec:

```
php bin/console d:f:l
```

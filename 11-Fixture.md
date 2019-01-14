## Fixtures

Pour permettre à un nouveau développeur d'intégrer le projet vite et de lui permettre d'avoir des données de test nous allons utiliser un bundle qui est `doctrine-fixtures-bundle`.

Il va permettre de définir un jeu de donnée initiale

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
        $author = new User();
        $author->setEmail('john@doe.fr');
        $author->setPassword('password');
        $author->setName('John Doe');
        $author->setUsername('john doe');
        $manager->persist($user);

        $article = new Article();
        $article->setTitle('test');
        $article->setUser($user);
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

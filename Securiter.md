## Security 

Pour faire une partie backend et frontend nous allons mettre en place le componant Security <https://symfony.com/doc/current/security.html>

Cela aura pour effet de permettre de connecter les auteurs à la partie backend. 

Il va falloir rajouter quelques champs dans l'entity Author. Pour cela on peut mettre à jour l'entity avec `make:entity`. Nous allons lui rajouter `password`.

Ensuite nous allons implémenter `UserInterface`, si nous regardons le fichier, ça nous donne ça:

```
namespace Symfony\Component\Security\Core\User;

use Symfony\Component\Security\Core\Role\Role;

/**
 * Represents the interface that all user classes must implement.
 *
 * This interface is useful because the authentication layer can deal with
 * the object through its lifecycle, using the object to get the encoded
 * password (for checking against a submitted password), assigning roles
 * and so on.
 *
 * Regardless of how your user are loaded or where they come from (a database,
 * configuration, web service, etc), you will have a class that implements
 * this interface. Objects that implement this interface are created and
 * loaded by different objects that implement UserProviderInterface
 *
 * @see UserProviderInterface
 * @see AdvancedUserInterface
 *
 * @author Fabien Potencier <fabien@symfony.com>
 */
interface UserInterface
{
    /**
     * Returns the roles granted to the user.
     *
     * <code>
     * public function getRoles()
     * {
     *     return array('ROLE_USER');
     * }
     * </code>
     *
     * Alternatively, the roles might be stored on a ``roles`` property,
     * and populated in any number of different ways when the user object
     * is created.
     *
     * @return (Role|string)[] The user roles
     */
    public function getRoles();

    /**
     * Returns the password used to authenticate the user.
     *
     * This should be the encoded password. On authentication, a plain-text
     * password will be salted, encoded, and then compared to this value.
     *
     * @return string The password
     */
    public function getPassword();

    /**
     * Returns the salt that was originally used to encode the password.
     *
     * This can return null if the password was not encoded using a salt.
     *
     * @return string|null The salt
     */
    public function getSalt();

    /**
     * Returns the username used to authenticate the user.
     *
     * @return string The username
     */
    public function getUsername();

    /**
     * Removes sensitive data from the user.
     *
     * This is important if, at any given point, sensitive information like
     * the plain-text password is stored on this object.
     */
    public function eraseCredentials();
}

```

Ainsi, nous allons rajouter les méthodes manquantes qui sont `eraseCredentials`, `getRoles`, `getSalt` dans notre entité `Author`.

Pour Roles, il faudra aussi mettre à jour la base, et rajouter une column de type json_array

```
    public function getRoles()
    {
        $roles = $this->roles;
        if (!\in_array('ROLE_USER', $roles)) {
            $roles[] = 'ROLE_USER';
        }
        return $roles;
    }

    public function setRoles(array $roles)
    {
        $this->roles = $roles;
    }
```
Pensez à mettre à jour votre base de données.

Après cela nous allons dans notre fichier de configuration de security pour le modifier.

```
providers:
        main:
            entity:
                class: App\Entity\Author
                property: username

encoders:
		 App\Entity\Author:
            algorithm: bcrypt
```

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

Il ne nous manque plus que notre formulaire de login !
 
Pour cela nous allons indiquer dans nos fichiers de configuration le path pour le login:

```
    firewalls:
        main:
            form_login:
                login_path: login
                check_path: login
```
Nous allons créer un Controller spécifique pour gérer ça que nous nommerons `SecurityController` 

Voici la méthode créée pour login :

```
    /**
     * @Route("/login", name="login")
     */
    public function login(AuthenticationUtils $authenticationUtils)
    {
        // get the login error if there is one
        $error = $authenticationUtils->getLastAuthenticationError();

        // last username entered by the user
        $lastUsername = $authenticationUtils->getLastUsername();

        return $this->render('security/login.html.twig', array(
            'last_username' => $lastUsername,
            'error'         => $error,
        ));
    }
```
 
Le fichier twig associé est aussi simple:

```
{% if error %}
    <div>{{ error.messageKey|trans(error.messageData, 'security') }}</div>
{% endif %}

<form action="{{ path('login') }}" method="post">
    <label for="username">Username:</label>
    <input type="text" id="username" name="_username" value="{{ last_username }}" />

    <label for="password">Password:</label>
    <input type="password" id="password" name="_password" />


    <button type="submit">login</button>
</form>
```

Si vous allez sur /login puis que vous utilisez johndoe/test, vous allez être redirigé vers votre index. Dans votre barre de développement vous pouvez voir que vous n'êtes plus ano mais bien authentifié en tant que John Doe.

Nous allons pouvoir sécuriser `PostController`

nous allons rajouter `use Sensio\Bundle\FrameworkExtraBundle\Configuration\Security;` pour nous permettre d'utiliser les annoncations `@Security`

```
/**
 * @Route("/post")
 * @Security("has_role('ROLE_ADMIN')")
 */
```

Si nous tentons d'atteindre /post, nous aurons le droit à un beau Access denied, car par défaut, nous n'avons qu'un ROLE_USER du coup dans notre AppFixtures il faudra rajouter:

```
        $author->setRoles(['ROLE_ADMIN']);
```

On reload les fixtures et voilà, nous avons sécuriser notre zone /post.
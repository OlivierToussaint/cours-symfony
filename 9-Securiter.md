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

```
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

```
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

Il suffira de mettre à jour notre entité Article comme nous l'avons fait avec Category 


## EventDispatcher 


Il va nous permettre de déclencher un évènement (envoie d'un email par exemple) à un moment donné.

Dans le cadre du blog, on va envoyer un message de bienvenue.

Nous allons commencer par télécharger switchmailer qui va nous permettre d'envoyer un email

```composer req mailer```

Nous irons configurer notre email dans notre environement ```.env```

Une fois le service configurer on va mettre en place notre évènement nous allons créer un fichier à la racine de src se nommant ```Events.php```

Dedans nous allons juste déclarer des constantes de nos events

```
namespace App;

final class Events
{
    const USER_REGISTERED = 'user.registered';
}
```

On peut passer par le maker même si il faudra faire des retouches

```
php bin/console make:subscriber

  What event do you want to subscribe to?:
 > user.registered
```

On va voir notre fichier fraichement créer et on va le modifier en fonction.

Nous allons lui mettre notre constantes, et nous allons créer une fonction qui va envoyer un email, dans cette fonction nous lui passerons en paramètre GenericEvent <https://symfony.com/doc/current/components/event_dispatcher/generic_event.html> qui nous permettra de pouvoir recupérer l'email de notre inscrit.

ça va nous donner le fichier suivant :

```
use App\Entity\Author;
use App\Events;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\EventDispatcher\GenericEvent;

class RegisterSubscriber implements EventSubscriberInterface
{
    private $mailer;

    public function __construct(\Swift_Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    public function onUserRegistered(GenericEvent $event)
    {
        $author = $event->getSubject();

        if ($author instanceof Author) {
            $message = (new \Swift_Message("Bienvenue"))
                ->setFrom('olivier@toussaint.fr')
                ->setTo($author->getEmail())
                ->setBody("Vous venez de vous enregistrez sur notre site");
            $this->mailer->send($message);
        }
    }

    public static function getSubscribedEvents()
    {
        return [
           Events::USER_REGISTERED => 'onUserRegistered',
        ];
    }
```

La bonne pratique est de stocker l'email de sender dans les paramètres et de lui donner en parametres.

Nous allons donc le déclarer dans ```config/services.yaml``` dans parameters nous rajouterons :

```
parameters:
    app.sender: olivier@toussaint.fr

services:
    App\EventSubscriber\RegisterSubscriber:
        $sender: '%app.sender%'
```

Du coup nous passerons dans notre construct le ```$sender```

```
    private $mailer;

    private $sender;
    
    public function __construct(\Swift_Mailer $mailer, $sender)
    {
        $this->mailer = $mailer;
        $this->sender = $sender;
    }

    public function onUserRegistered(GenericEvent $event)
    {
        $author = $event->getSubject();

        if ($author instanceof Author) {
            $message = (new \Swift_Message("Bienvenue"))
                ->setFrom('test@toussaint.fr')
                ->setTo($author->getEmail())
                ->setBody("Vous venez de vous enregistrez sur notre site");
            $this->mailer->send($message);
        }
    }
```

Il ne reste plus qu'a faire notre formulaire d'enregistrement, et je vous laisserez la faire :-D.

Néamoins pour enregistrer un user il y a des petites choses à savoir.

```
/**
     * @Route("/register", name="register")
     */
    public function register(Request $request, UserPasswordEncoderInterface $passwordEncoder, EventDispatcherInterface $eventDispatcher)
    {
        $author = new Author();

        $form = $this->createForm(AuthorType::class, $author);

        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $author->setPassword($passwordEncoder->encodePassword($author, $author->getPassword()));
            $entityMananger = $this->getDoctrine()->getManager();
            $entityMananger->persist($author);
            $entityMananger->flush();

            $event = new GenericEvent($author);
            $eventDispatcher->dispatch(Events::USER_REGISTERED, $event);
            return $this->redirectToRoute('login');
        }

        return $this->render('security/register.html.twig', [
            'form' => $form->createView()
        ]);
    }
```
Nous allons utiliser les fonctions de ```UserPasswordEncoderInterface``` pour encoder notre password en base avec la méthode défini dans ```config/security.yaml``` (ici bcrypt).

J'ai rajouté dans l'entity le fait que l'objet author à part defaut le role ```ROLE_USER```

Revenons à notre Event.

Nous utiliser ```GenericEvent``` pour passer l'auteur car notre évènement est simple, sinon nous aurions créer un Objet Event plus complexe comme l'exemple dans la doc <https://symfony.com/doc/current/components/event_dispatcher.html>.

Voila maintenant à la création d'un utilisateur en passant par ```/register``` nous aurons un email de bienvenue.
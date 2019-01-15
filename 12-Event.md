## EventDispatcher 


Il va nous permettre de déclencher un évènement (envoie d'un email par exemple) à un moment donné.

Dans le cadre du blog, on va envoyer un message de bienvenue.

Nous allons commencer par télécharger switchmailer qui va nous permettre d'envoyer un email

```bash
# composer req mailer
```

Nous irons configurer notre email dans notre environement ```.env```

Une fois le service configuré, on va mettre en place notre événement nous allons créer un fichier à la racine de src se nommant ```Events.php```

Dedans nous allons juste déclarer des constantes de nos events

```php
namespace App;

final class Events
{
    const USER_REGISTERED = 'user.registered';
}
```

On peut passer par le maker même s'il faudra faire des retouches

```bash
p# hp bin/console make:subscriber

  What event do you want to subscribe to?:
 > user.registered
```

On va voir notre fichier fraichement créé et on va le modifier pour qu'il puisse envoyer un email.

Nous allons créer une fonction qui va envoyer un email, dans cette fonction nous lui passerons en paramètre GenericEvent <https://symfony.com/doc/current/components/event_dispatcher/generic_event.html> qui nous permettra de pouvoir recupérer l'email de notre inscrit.

ça va nous donner le fichier suivant :

```php
use App\Entity\User;
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
        $user = $event->getSubject();

        if ($user instanceof User) {
            $message = (new \Swift_Message("Bienvenue"))
                ->setFrom('olivier@toussaint.fr')
                ->setTo($user->getEmail())
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

La bonne pratique est de stocker l'email de `sender` dans les paramètres de symfony, puis décrire le service plus bas avec cette variable.

Nous allons donc le déclarer dans ```config/services.yaml``` dans parameters nous rajouterons :

```
parameters:
    app.sender: olivier@toussaint.fr

services:
    App\EventSubscriber\RegisterSubscriber:
        $sender: '%app.sender%'
```

Du coup nous passerons dans notre construct le ```$sender```

```php
    private $mailer;

    private $sender;
    
    public function __construct(\Swift_Mailer $mailer, $sender)
    {
        $this->mailer = $mailer;
        $this->sender = $sender;
    }

    public function onUserRegistered(GenericEvent $event)
    {
        $user = $event->getSubject();

        if ($user instanceof User) {
            $message = (new \Swift_Message("Bienvenue"))
                ->setFrom($this->sender)
                ->setTo($user->getEmail())
                ->setBody("Vous venez de vous enregistrez sur notre site");
            $this->mailer->send($message);
        }
    }
```

Nous allons le rajouter dans notre SecurityController deux lignes

```php
$event = new GenericEvent($user);
$eventDispatcher->dispatch(Events::USER_REGISTERED, $event);
 ```

Nous utiliser ```GenericEvent``` pour passer l'utilisateur car notre évènement est simple, sinon nous aurions créer un Objet Event plus complexe comme l'exemple dans la doc <https://symfony.com/doc/current/components/event_dispatcher.html>.

Voila maintenant à la création d'un utilisateur en passant par ```/register``` nous aurons un email de bienvenue.
## Les Forms

Documentation : https://symfony.com/doc/current/forms.html

Nous pouvons créer des formulaires via le template ou bien nous pouvons laisser symfony faire tout ça.

```
 ~> php bin/console make:form
```
Nous allons lui donner l’entity sur lequel elle va se calquer, puis ensuite nous choisirons de rajouter le bouton submit ou pas dans ce formType.

```
public function buildForm(FormBuilderInterface $builder, array $options)
{
    $builder
        ->add('title')
        ->add(‘content');
    ;
}
```
Nous pourrions dans ce fichier personnaliser les labels ou autres. Pour tout cela nous irons lire la documentation

```
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

```
/**
 * @Route("/home", name="home")
 */
public function index(Request $request)
{
    $article = new Article();
    $form = $this->createForm(ArticleType::class, $article);

    $form->handleRequest($request);
    if ($form->isSubmitted() && $form->isValid())
    {
        $em = $this->getDoctrine()->getManager();
        $em->persist($article);
        $em->flush();
        $this->addFlash('success', 'Votre article est enregistré');
        return $this->redirectToRoute('home');
    }

    return $this->render('home/index.html.twig', [
        'controller_name' => 'HomeController',
        'form' => $form->createView(),

    ]);
}
```

----
Nous initialiserons la création du formulaire avec:

``` 
       $form = $this->createForm(PostType::class, $post);
```

On peut remarquer qu'il fait appel à PostType qui est stocké dans un nouveau répertoire Form dans notre src:

```
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('title')
            ->add('slug')
            ->add('body')
            ->add('createdAt')
            ->add('updatedAt')
            ->add('author')
        ;
    }
```

Ce FormType sert à définir les champs qui vont apparaître dans le formulaire, on pourra le générer avec la commande suivante:

```
php bin/console make:form
```

Il faudra la lier à l'entity souhaitée:


Malheureusement pour nous, il nous fera pas en automatique la relation ManyToOne, nous allons donc nous en charger pour faire un select sur les authors

```
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('title')
            ->add('slug')
            ->add('body')
            ->add('createdAt')
            ->add('updatedAt')
            ->add('author', EntityType::class, [
                'class' => Author::class,
                'choice_label' => 'username'])
        ;
    }
```

J'ai rajouté des spécifications au champs author, je pourrais faire de même sur tous les champs si j'en avais envie.

Ce `PostType` va nous servir pour l'édition et la création de l'objet `Post` dans notre base.

----

```
        $form->handleRequest($request);
```

On injecte tout simplement l'object `Request` pour qu'il fasse le travail tout seul, plus sérieusement la définition de la documentation de cette classe:

>Internally, the request is forwarded to the configured {@link RequestHandlerInterface} instance, which determines whether to submit the form or not.

----

```
        if ($form->isSubmitted() && $form->isValid()) {
            $em = $this->getDoctrine()->getManager();
            $em->persist($post);
            $em->flush();

            return $this->redirectToRoute('post_index');
        }
```

Là, on vérifie que le formulaire est bien submit et que ce dernier est valide, si c'est le cas, on enregistre en base l'object `Post`

*Comment enregistre-t-on en base ?*

Nous avons vu très vite l'entity manager quand nous avons chargé nos fixtures dans l'orm. Nous allons l'utiliser pour enregistrer en base notre object.

À quoi sert cette Entity Manger ? C'est très simple vous n'avez plus à vous soucier de faire un `INSERT` ou un `UPDATE` en base, l'orm va s'occuper de tout, vous n'avez qu'à charger le Manager (à partir de `EntityManagerInterface` pour de l'injection directe) ou bien le récuperer dans votre controller avec `$this->getDoctrine()->getMananger()` 

Pour ce faire, nous allons utiliser la fonction `persist` et `flush` comme montré dans l'exemple:

Je vous renvoie à la documentation qui est vaste <https://symfony.com/doc/current/doctrine.html>

----

```
        return $this->render('post/new.html.twig', [
            'post' => $post,
            'form' => $form->createView(),
        ]);
```
Le `$form->createView()` servira à construire notre formulaire dans twig qui est représenté par `_form.html.twig`


Dans le twig 

```
{{ form_start(form) }}
{{ form_errors(form) }}
{{ form_widget(form) }}
<button type="submit">Save</button>
{{ form_end(form) }}
```

On pourrait personnalisé les forms en détaillant le tout

```
{{ form_start(form) }}
{{ form_errors(form) }}
{{ form_widget(form.title) }}
{{ form_widget(form.content) }}
<button type="submit">Save</button>
{{ form_end(form) }}
```


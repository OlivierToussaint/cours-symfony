## Bootstrap

Document : <https://symfony.com/doc/current/form/bootstrap4.html>

Pour relier notre blog au framework css bootstrap, nous allons devoir chercher bootstrap.css, pour faciliter les choses nous allons prendre le CDN

https://www.bootstrapcdn.com

Nous allons intégrer cette ligne à notre balise <head> qui se trouve dans base.html.twig

{% raw %} 
```twig
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Welcome!{% endblock %}</title>
        {% block stylesheets %}{% endblock %}
        <link rel="stylesheet" type="text/css" href="https://stackpath.bootstrapcdn.com/bootstrap/4.2.1/css/bootstrap.min.css">
    </head>
```
{% endraw %} 


Nous allons après dans le fichier de configuration de twig qui se trouve dans config/package/ rajoutez cette ligne
 
```
    form_themes: ['bootstrap_4_layout.html.twig']
```

Maintenant vos form prennent le thème de Symfony avec les class=‘form-control’ et autre.

Pour faciliter la suite, nous allons la navbar de symfony. (<https://getbootstrap.com/docs/4.2/components/navbar/>)

On va prendre la première qui se trouve dans la documentation et rajouter cela dans notre base.html.twig

Dans le menu du haut je souhaite avoir une page de présentation qui sera gérée dans `HomeController`, un liens vers la liste des Articles et un lien vers la liste des Catégories

Pour rendre le menu plus ergonomique, nous allons faire en sorte de rendre actif le menu sur lequel nous sommes actuellement

```twig
<li class="nav-item {% if app.request.get('_route') == 'app_logout' %}active{% endif %}">
	<a class="nav-link" href="{{ path('app_logout') }}">Logout</a>
</li>
```

Le `app.request.get('_route')` va nous permettre d'avoir le nom de la route actuelle et si jamais il correspond à la route souhaitée alors nom mettons active dans la class du li
## Twig 

Nous allons au fichier précréé `blog/index.html.twig` par maker

{% raw %} 
```twig
{% extends 'base.html.twig' %}

{% block title %}Hello {{ controller_name }}!{% endblock %}

{% block body %}
<style>
    .example-wrapper { margin: 1em auto; max-width: 800px; width: 95%; font: 18px/1.5 sans-serif; }
    .example-wrapper code { background: #F5F5F5; padding: 2px 6px; }
</style>

<div class="example-wrapper">
    <h1>Hello {{ controller_name }}! </h1>

    This friendly message is coming from:
    <ul>
        <li>Your controller at <code><a href="{{ 'src/Controller/AuthorController.php'|file_link(0) }}">src/Controller/AuthorController.php</a></code></li>
        <li>Your template at <code><a href="{{ 'templates/author/index.html.twig'|file_link(0) }}">templates/author/index.html.twig</a></code></li>
    </ul>
</div>
{% endblock %}
```
{% endraw %}
 
Nous retrouvons notre variable qui est `controller_name` qui est entourée d'accolade.

Nous pouvons observer différents `block`, c'est ce que l'on va trouver dans le fichier qu'il extend `base.html.twig`. Dans ce fichier il y a tout simplement la structure de base de notre site.

Pour le blog nous allons la remanier, nous allons y rajouter bootstrap, mais juste le css.

Ca donne ça 

```twig
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Bienvenue sur notre blog{% endblock %}</title>
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css" integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
        {% block stylesheets %}{% endblock %}
    </head>
    <body>
        {% block body %}{% endblock %}
        {% block javascripts %}{% endblock %}
    </body>
</html>
```

C'est dans ce fichier que nous allons créer nos blocks que l'on remplira par des données.

Plus tard nous pourrons avoir besoin des variables globales telles que `{{ app.request }}`, `{{ app.session }}`, `{{app.user }}`

Twig à une documentation juste pour lui <https://twig.symfony.com/doc/2.x/>.

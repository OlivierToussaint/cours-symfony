## Twig 

Nous retrouvons notre variable qui est `controller_name` qui est entourée d'accolade.

Nous pouvons observer différents `{% block %}`, c'est ce que l'on va trouver dans le fichier qu'il extend `base.html.twig`. Dans ce fichier il y a tout simplement la structure de base de notre site.

Pour le blog nous allons la remanier, nous allons y rajouter bootstrap, mais juste le css.

Ca donne ça 

```
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

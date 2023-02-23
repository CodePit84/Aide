Pour le stylesheet bootswatch et le script JS bootstrap :

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Welcome!{% endblock %}</title>
        <link rel="icon" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 128 128%22><text y=%221.2em%22 font-size=%2296%22>⚫️</text></svg>">
        {# Run `composer require symfony/webpack-encore-bundle` to start using Symfony UX #}
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bootswatch/5.2.3/zephyr/bootstrap.min.css" integrity="sha512-dcTg+pv6j02FTyko5ua8nsnARs/l4u43vmnbeVgkFWB5wdLgfUq4CEotFWOlTE4XK7FfVriWj7BrpqET/a+SJQ==" crossorigin="anonymous" referrerpolicy="no-referrer" />
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/js/bootstrap.bundle.min.js" integrity="sha384-w76AqPfDkMBDXo30jS1Sgez6pr3x5MlQ1ZAGC+nuZB+EYdgRZgiwxhTBTkF7CXvN" crossorigin="anonymous" defer></script>
        {% block stylesheets %}
        {% endblock %}

        {% block javascripts %}
        {% endblock %}
    </head>
    <body>
        {% block header %}
            {% include "partials/_header.html.twig" %}
        {% endblock %}
        
        {% block body %}{% endblock %}

        {% block footer %}
            {% include "partials/_footer.html.twig" %}
        {% endblock %}
    </body>
</html>
```

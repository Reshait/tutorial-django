.. _reference-logout_usuario:

Logout de un usuario
====================

Ahora nos queda que el usuario pueda desloguearse del sitio, de lo que llevamos hecho, esto es lo mas fácil y sobre todo, como ya debes entender el funcionamiento de las vistas y plantillas, pues todo sera rápido y sin dolor!.

Como siempre, se ha de crear una vista y añadir la ``url()`` al URLconf, pero en esta ocasión, no vamos a crear una plantilla, a cambio, lo mostraremos con un mensaje para así echar un ojo al sistema ``django.contrib.messages`` (hasta los haremos un poco bonitos :P).

Empecemos con la vista, es tan simple que ni la comentare.

.. code-block:: python

    # accounts/views.py

    # Añadir import logout y messages
    from django.contrib.auth import authenticate, login, logout
    from django.contrib import messages


    def logout_view(request):
        logout(request)
        messages.success(request, 'Te has desconectado con exito.')
        return redirect(reverse('accounts.login'))

Antes de continuar, vamos a crear un pequeño sistema que muestra los mensajes con un poco de estilo css gracias a bootstrap.

.. code-block:: html

    <!-- templates/base.html ->

    <!-- Antes de {% block content %}{% endblock content %} añadimos -->
    {% include '_messages.html' %}
    {% block content %}{% endblock content %}

Con la etiqueta {% include '_nombre_plantilla.html' %} importa un archivo .html y lo 'vuelca' en el lugar donde es llamado.

El archivo ``_messages.html`` es una plantilla que ahora vamos a crear y lo creamos en ``templates`` de la raiz del proyecto.

.. code-block:: html

    <!-- templates/_messages.html -->

    {% if messages %}
        <div class="row dj-messages">
            <div class="col-md-6 col-md-offset-3">
                {% for message in messages %}
                    {% if message.tags == 'error' %}
                    <div class="alert alert-danger" role="alert">
                    {% else %}
                    <div class="alert alert-{{ message.tags }}" role="alert">
                    {% endif %}
                        {{ message }}<button type="button" class="close" data-dismiss="alert">&times;</button>
                    </div>
                {% endfor %}
            </div>
        </div>
    {% endif %}

Primero comprueba si existe una variable ``messages`` y en caso de existir, mostrara el mensaje con un estilo (usando bootstrap), con un diseño decente, si no existe ``messages``, no insertara nada.

También vamos a poner en la barra de navegación, un link para que un usuario pueda hacer login/logout.

.. code-block:: html

    <!-- templates/base.html -->

    <!DOCTYPE html>
    {% load staticfiles %}
    <html lang="es">
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>{% block title %}{% endblock title %}</title>
        <link rel="stylesheet" href="{% static 'bootstrap/css/bootstrap.css' %}">
        <link rel="stylesheet" href="{% static 'css/main.css' %}">
    </head>
    <body>

        <nav class="navbar navbar-inverse navbar-fixed-top">
          <div class="container">
            <div class="navbar-header">
              <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar">
                <span class="sr-only">Toggle navigation</span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
              </button>
              <a class="navbar-brand" href="#">Project name</a>
            </div>
            <div id="navbar" class="collapse navbar-collapse">
              <ul class="nav navbar-nav">
                <li class="active"><a href="#">Home</a></li>
                <li><a href="#about">About</a></li>
                <li><a href="#contact">Contact</a></li>
              </ul>
              <ul class="nav navbar-nav navbar-right">
                  {% if user.is_authenticated %}
                    <li><a href="{% url 'accounts.index' %}">{{ user.username }}</a></li>
                    <li><a href="{% url 'accounts.logout' %}">Logout</a></li>
                  {% else %}
                    <li><a href="{% url 'accounts.registro' %}">Registro</a></li>
                    <li><a href="{% url 'accounts.login' %}">Login</a></li>
                  {% endif %}
              </ul>
            </div><!--/.nav-collapse -->
          </div>
        </nav>

        <div class="container">
            {% include "_messages.html" %}
            {% block content %}{% endblock content %}
        </div><!-- /.container -->

        <script src="{% static 'jquery/jquery.js' %}"></script>
        <script src="{% static 'bootstrap/js/bootstrap.js' %}"></script>
        <script src="{% static 'js/main.js' %}"></script>
    </body>
    </html>

Resalto la parte añadida:

.. code-block:: html

    <!-- templates/base.html -->

    <ul class="nav navbar-nav navbar-right">
      {% if user.is_authenticated %}
        <li><a href="{% url 'accounts.index' %}">{{ user.username }}</a></li>
        <li><a href="{% url 'accounts.logout' %}">Logout</a></li>
      {% else %}
        <li><a href="{% url 'accounts.registro' %}">Registro</a></li>
        <li><a href="{% url 'accounts.login' %}">Login</a></li>
      {% endif %}
    </ul>

Como se vio anteriormente, el objeto ``user``, esta disponible en las plantillas y un método es ``is_autenthicate`` (como también se vio en una vista), por lo que si esta autenticado, mostrara el botón para acceder al indice de la cuenta con el nombre del usuario y ``Logout`` para darle la posibilidad de desloguearse del sistema, en caso contrario, mostrara los botones ``Registro`` y ``Login`` para que pueda registrarse o loguearse respectivamente.

También se puede ver, que cuando haces logout, mostrara el mensaje ``Te has desconectado con éxito`` con una ``x`` para cerrar el mensaje, el mensaje solo lo mostrara una vez (si actualizas la pagina, no aparecerá) y solo lo ve el usuario que ha hecho logout.

Ya tenemos casi terminado el sistema de usuarios (una versión simple), en la siguiente sección, veremos como actualizar algunos datos del usuario.

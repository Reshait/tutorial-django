.. _reference-archivos_static:

Archivos static
===============

Ahora es el momento de ver como incluir los archivos estáticos (**css, js, imagenes**) al proyecto.

Si abres el archivo de configuración ``tutorial_django/settings.py`` y bajas al final del archivo, vemos una variable de configuración ``STATIC_URL = '/static/'``, de esta manera le esta diciendo que, cuando en las plantillas llamamos a un archivo estático, anteponga en la **URI** ``/static/`` (ahora veremos como funciona), a parte tenemos que decirle a Django donde estarán esos archivos estáticos, así que, debajo de ``STATIC_URL`` ponemos lo siguiente:

.. code-block:: python

    # tutorial_django/settings.py

    STATIC_URL = '/static/'
    STATICFILES_DIRS = (
        os.path.join(BASE_DIR, 'static'),
    )

``STATICFILES_DIRS`` es una tupla, que dice las rutas absolutas del sistema para buscar archivos estáticos, en nuestro caso, solo hemos añadido uno.

Ahora, ya sabrá Django que los archivos estarán en la ``src/static`` (con una ruta absoluta), pero ese directorio no existe, así que lo creamos y dentro creamos tres directorios mas, ``js``, ``css`` e ``img``

.. code-block:: bash

    mkdir -p static/{js,css,img}

dentro de **css** creamos un archivo ``main.css`` y dentro de **js** un archivo ``main.js``. Ya para 'terminar', los incluimos en ``templates/base.html``

.. code-block:: html

    <!-- templates/base.html -->

    <!DOCTYPE html>
    {% load staticfiles %}
    <html lang="es">
    <head>
        <meta charset="utf-8">
        <title>{% block title %}{% endblock title %}</title>
        <link rel="stylesheet" href="{% static 'css/main.css' %}">
    </head>
    <body>
        <h1>Pagina principal</h1>
        {% block content %}{% endblock content %}

        <script src="{% static 'js/main.js' %}"></script>
    </body>
    </html>

Hemos incluido una **tag** o etiqueta ``{% load staticfiles %}`` (si no, dará error al cargar la pagina), cuando llama un archivo estático como ``{% static 'js/main.js' %}``, Django cambiara ``{% static 'js/main.js' %}`` por el valor de ``settings.STATIC_URL + js/main.js`` que en este caso sera ``<script src="/static/js/main.js"></script>``

Dentro de ``static/css/main.css`` añadimos:

.. code-block:: css

    /* static/css/main.css */

    body { background-color: blue; }

Ahora, reiniciamos el servidor (por si acaso) y vamos a `http://127.0.0.1:8000/ <http://127.0.0.1:8000/>`_ y vemos que el fondo es azul, lo importante es ver el código **html** generado, si nos fijamos ha puesto la ruta ``<link rel="stylesheet" href="/static/css/main.css">``, eso significa que podríamos cambiar el directorio a otro ruta y cambiando en **settings** la ruta, sin modificar nada mas, seguiremos teniendo a nuestra disposición los archivos estáticos.

Para el resto del tutorial, vamos a poner `Bootstrap <http://getbootstrap.com/>`_ y `JQuery <https://jquery.com/>`_, descargamos ambas librerías y las descomprimimos dentro del directorio **static** (**jquery** no lo usaremos), quedando de esta manera la estructura.

.. code-block:: bash

    static/
    ├── bootstrap
    │   ├── css
    │   │   ├── bootstrap.css
    │   │   ├── bootstrap.css.map
    │   │   ├── bootstrap.min.css
    │   │   ├── bootstrap-theme.css
    │   │   ├── bootstrap-theme.css.map
    │   │   └── bootstrap-theme.min.css
    │   ├── fonts
    │   │   ├── glyphicons-halflings-regular.eot
    │   │   ├── glyphicons-halflings-regular.svg
    │   │   ├── glyphicons-halflings-regular.ttf
    │   │   ├── glyphicons-halflings-regular.woff
    │   │   └── glyphicons-halflings-regular.woff2
    │   └── js
    │       ├── bootstrap.js
    │       ├── bootstrap.min.js
    │       └── npm.js
    ├── css
    │   ├── main.css
    ├── img
    ├── jquery
    │   └── jquery.js
    └── js
        └── main.js

    8 directories, 18 files

E incluimos en nuestro ``templates/base.html``. Ademas añadimos el típico **navbar** superior de **Bootstrap**, quedando de la siguiente manera nuestro archivo ``base.html``

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
            </div><!--/.nav-collapse -->
          </div>
        </nav>

        <div class="container">

          <div class="starter-template">
            {% block content %}{% endblock content %}
          </div>

        </div><!-- /.container -->

        <script src="{% static 'jquery/jquery.js' %}"></script>
        <script src="{% static 'bootstrap/js/bootstrap.js' %}"></script>
        <script src="{% static 'js/main.js' %}"></script>
    </body>
    </html>

El archivo **css** también le vamos a quitar ese color azul tan molón :P y ponerle el margen de la barra.

.. code-block:: css

    /* static/css/main.css */

    body {
        margin: 70px 0 20px 0;
    }

Si actualizamos la pagina, ahora se ve que la cosa cambia :) y es hora de empezar a profundizar mas en todos los componentes que hasta ahora hemos tocado, a parte de que queda mucho por ver como los modelos.

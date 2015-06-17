.. _reference-templates_and_static:

Templates
=========

Hasta ahora, ya sabemos como crear un proyecto, una app, configurar URLconfs y enlazarlas a las vistas, ahora es hora de renderizar desde las vistas a los templates o plantillas e incluir los archivos.

Empezamos con el archivo de configuración ``tutorial_django/settings.py``, buscamos la lista ``TEMPLATES`` y vemos que contiene un diccionario, un elemento de ese diccionario es ``'DIRS': [],`` una clave ``DIRS`` con un valor que es una lista vacía. Dentro de la lista vamos a decir donde guardaremos los archivos de plantillas, quedando de la siguiente manera.

.. code-block:: python

    # tutorial_django/settings.py

    TEMPLATES = [
        {
            # ...
            'DIRS': [os.path.join(BASE_DIR, 'templates')],
            # ...
        }
    ]

Ahora, tenemos que crear el directorio ``templates`` en el directorio raiz del proyecto

.. code-block:: bash

    mkdir templates

Como puedes ver dentro del diccionario dentro de ``TEMPLATES`` hay un elemento ``'APP_DIRS': True,`` que le dice si buscar dentro de las apps un directorio ``templates`` donde contendrán plantillas que pertenecen a esa app (mas adelante lo veremos como funciona) y terminare de explicar como busca las plantillas Django, por ahora, quédate con ``'APP_DIRS': True,``.

Creamos dentro del directorio ``templates`` un archivo ``.html`` que sera el archivo base, el que se leerá en todas nuestras paginas con el nombre ``base.html`` y añadimos el siguiente codigo html.

.. code-block:: html

    <!-- templates/base.html -->

    <!DOCTYPE html>
    <html lang="es">
    <head>
        <meta charset="utf-8">
        <title></title>
    </head>
    <body>
        <h1>Pagina principal</h1>
    </body>
    </html>

Volvemos al archivo de vista en la app home y cambiamos todo el código por el siguiente:

.. code-block:: python

    # home/views.py

    from django.shortcuts import render


    def index_view(request):
        return render(request, 'base.html')

Como se pude ver, se ha cambiado el from... y el return..., ahora vemos que en vez de devolver ``HttpResponse`` como hacíamos antes, ahora devolvemos ``render`` que es un atajo donde volvemos a pasar el ``HttpRequest``, el nombre de plantilla a usar y mas adelante, veremos como pasar un diccionario con datos (un diccionario con pares clave/valor) para poderlos mostrar desde las plantillas.

Ahora si iniciamos el servidor ``./manage.py runserver`` y vamos a `http:/127.0.0.1:8000 <http:/127.0.0.1:8000>`_ veremos que la salida, es la del html creado.

Vamos a empezar con la herencia de plantillas, para ver como podemos crear bloques, vamos a editar el archivo ``base.html`` y creamos un bloque.

.. code-block:: html

    <!-- templates/base.html -->

    <!DOCTYPE html>
    <html lang="es">
    <head>
        <meta charset="utf-8">
        <title>{% block title %}{% endblock title %}</title>
    </head>
    <body>
        <h1>Pagina principal</h1>
        {% block content %}{% endblock content %}
    </body>
    </html>

Podemos observar que hemos creado dos bloques, ¿para que sirven?, pues bien, cuando una plantilla extiende de otra, la plantilla que llama sustituye el contenido del bloque por la de la plantilla 'padre', para verlo, dentro de ``home``, creamos un directorio ``templates`` y dentro creamos otro directorio con el nombre ``home`` y dentro un archivo .html con el nombre ``index.html``.

.. code-block:: bash

    mkdir -p home/templates/home
    touch home/templates/home/index.html

y ponemos este código:

.. code-block:: html

    <!-- home/templates/home/index.html -->

    {% extends 'base.html' %}

    {% block title %}home{% endblock title %}

    {% block content %}
        <h2>Home page</h2>
    {% endblock content %}

La manera de extender una plantilla es con ``{% extends 'base.html' %}`` donde le estamos diciendo que extienda la plantilla a ``base.html`` y a partir de hay, los bloques (block) de código, cambiaran los datos de la plantilla actual con la de la plantilla extendida, en este caso ``base.html``.

Como se puede apreciar, puede parecer un poco follón crear un directorio ``template`` y dentro otro con el mismo nombre de la app, en este caso ``home``, ¿porque este lío?, sencillo, imagina que creas diez apps en un proyecto y tres de ellos tienen una plantilla ``index.html``, ¿como sabe Django que plantilla cargar?, de hay que se crea siempre un directorio con el nombre de la app (los nombres de las apps, son siempre únicos).

Otra manera o estructura de crear las plantillas, es dentro del directorio ``templates`` de la raiz del proyecto, es crear directorios con el mismo nombre que la apps y dentro las plantillas, pero yo por costumbre, siempre los creo en los directorios de la app.

Bien, continuemos... ahora, vamos a cambiar en la vista ``index_view`` la plantilla que requiere.

.. code-block:: python

    def index_view(request):
        return render(request, 'home/index.html')

Vemos que hemos añadido la ruta ``home/index.html`` y lo mejor de todo, vemos que ahora imprime lo de ``base.html`` y lo de ``index.html``, ahora la pagina muestra un titulo y debajo de ``Pagina principal`` inserta el contenido que hay dentro de 'block' en ``home/index.html``.

Para terminar con las plantillas (seguiremos a lo largo del tutorial), vamos a ver como pasar un contexto de la vista a la plantilla.

Volvemos a editar ``home/views.py``

.. code-block:: python

    # home/views.py

    from django.shortcuts import render
    from django.utils import timezone

    def index_view(request):
        context = {
            'ahora': timezone.now()
        }
        return render(request, 'home/index.html', context)

y ahora, en ``home/templates/home/index.html``

.. code-block:: html

    <!-- home/templates/home/index.html -->

    {% extends 'base.html' %}

    {% block title %}home{% endblock title %}

    {% block content %}
        <h2>Home page</h2>
        <p>Ahora es {{ ahora }}</p>
    {% endblock content %}

Y vemos que hemos pasado la fecha y hora de la maquina en la que estamos.

.. note::

    Si te sale la fecha en ingles y una hora que no corresponde a la de tu sistema, ves a ``tutorial_django/settings.py`` para editar las variables de configuración ``LANGUAGE_CODE = 'es-es'`` `ver identificadores <http://www.i18nguy.com/unicode/language-identifiers.html>`_ y ``TIME_ZONE = 'Europe/Madrid'`` `ver timezones <https://en.wikipedia.org/wiki/List_of_tz_database_time_zones>`_

En la siguiente sección, veremos como incluir archivos estáticos a nuestro proyecto.

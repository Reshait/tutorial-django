.. _reference-blog-creacion_app:

Creación APP Blog
=================

Hasta ahora, tenemos creado toda la parte de gestión de usuario, registro, login, logout el usuario puede cambiar algunos datos de su perfil.

Comprendemos como estructurar las plantillas, como incluir archivos **media** y **static**, comprendemos las relaciones **model, view, template** **MVT** (o **MVC**), como añadir patrones en el **router**, etc.

A partir de ahora, cambiare un poco las maneras de hacer el tutorial, ya que si no, seria repetirse un poco.

En vez de usar vistas basadas en funciones (Function Based Views FBV), usaremos vistas basadas en clases (Class Based Views CBV), intentare explicar los fundamentos del ORM de Django, los fundamentos de las plantillas, ``inclusion tags`` y otras cosas.

De momento, solo vamos a crear la **app** y añadirla en ``INSTALLED_APPS`` y e incluir las **urls**.

.. code-block:: bash

    ./manage.py startapp blog

    touch blog/urls.py

    mkdir -p blog/templates/blog


.. code-block:: python

    # tutorial_django/settings.py

    INSTALLED_APPS = (
        # ...

        'home',
        'accounts',
        'blog',
    )

La **app** ya esta creada, en la siguiente sección, empezamos con los modelos.

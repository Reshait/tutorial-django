.. _reference-creacion_app:

.. |br| raw:: html

   <br />

Crear nuestra primera app
=========================

Una aplicación (**app**), no es mas que un paquete Python con una estructura básica para hacer algo concreto, por ejemplo **blog**, **about**, **contact** seria cada una de ellas una aplicación. Las aplicaciones en Django, están (en un principio) totalmente desacopladas y puedes tener **apps** en varios proyectos sin modificar ni una sola linea de código, el único requisito es que este en la variable de entorno ``PYTHONPATH``

.. note::

    Cuando ejecutamos un archivo ``.py`` Python añade el directorio actual en ``PYTHONPATH``, por lo tanto, durante el desarrollo, cuando levantamos el servidor, al hacerlo a través de ``manage.py`` ``src`` estará en ``PYTHONPATH``.

Vamos a crear nuestra primera **app**, crearemos el **home** de nuestro sitio, para crear nuestra **app**, abriremos la terminal e iremos hasta ``src`` y ejecutaremos:

.. code-block:: bash

    ./manage.py startapp home

.. note::

    Si el comando anterior da algun error (en Windows creo), usar ``python manage.py startapp home`` o en windows ``.\manage.py startapp home``.

El comando anterior es posible crearlo con ``django-admin``, muy útil cuando se crean **apps** en otros directorios que no sean en la raíz del proyecto. Lo que el comando ``manage.py startapp``  hace es crear una estructura, que tranquilamente se podría crear los directorios y archivos a mano. El comando anterior crea la siguiente estructura:

.. code-block:: bash

    home/
    ├── migrations
    │   └── __init__.py
    ├── __init__.py
    ├── admin.py
    ├── models.py
    ├── tests.py
    └── views.py

Examinemos los distintos archivos que ha creado el comando anterior, nos crea un directorio ``migrations`` que contiene un archivo ``__init__.py`` (para decirle a Python que es un paquete), ``migrations``

    Las migraciones son manera de propagar los cambios que realice en sus modelos (la adición de un campo, la eliminación de un modelo , etc.) en el esquema de base de datos de Django. Están diseñados para ser en su mayoría automática.

    `fuente <https://docs.djangoproject.com/en/1.8/topics/migrations/>`_

``admin.py`` donde crearemos la forma que se vera la **app** en la administración de Django. |br|
``models.py`` donde crearemos los modelos y es el 'puente' entre Django y la base de datos. |br|
``test.py`` para test unitarios, no se trata el tema en el tutorial. |br|
``views.py`` donde crearemos las vistas (o controladores, para los que vengan de otros Frameworks).

Con todo esto en mente, vamos a crear el primer **Hello world** :), en primer lugar, tenemos que decirle a Django que incluya la aplicación, para ello, editamos ``tutotial_django/settings.py``

Vamos a ``INSTALLED_APPS`` y añadimos la siguiente linea:

.. code-block:: python

    # tutotial_django/settings.py

    INSTALLED_APPS = (
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',

        'home',
    )

Ahora, Django conocerá que tenemos una aplicación y buscara templates (plantillas), archivos de migración, etc, dentro de esa **app**.

En la siguiente sección, modificaremos las **urls** para poder acceder a las vistas.

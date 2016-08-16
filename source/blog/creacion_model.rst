.. _reference-blog-creacion_model:

Creación del modelo
===================

Una aplicación siempre que use una base de datos, se comienza por el modelo, la representación de la base de datos, sus tablas y sus campos (clases y propiedades en Python), el blog se compone de dos tablas, una para las etiquetas **Tag** y otra para los artículos **Article**.

**Tag** tendra tres campos

* ``id`` Primary Key creado de manera implícita.
* ``name`` string, único.
* ``slug`` string, nombre amigable de name, único.

**Article** tendrá mas campos, el **id, titulo, slug, owner** (propietario del articulo)

* ``id`` **Primary Key** creado de manera implícita.
* ``title`` **string**, único.
* ``slug`` **string**, nombre amigable de **title**, único.
* ``body`` **text**, cuerpo del articulo.
* ``owner`` **ForeignKey** (**User**), propietario del articulo.
* ``tags`` **ManyToMany** (**Tag**), lista de etiquetas.
* ``create_at`` **DateTime**, fecha y hora de creación.
* ``update_at`` **DateTime**, fecha y hora de ultima modificación.

Los ``Primary Key``, los crea de manera implícita a no ser que se cree explícitamente, ``owner`` es una relación **un usuario sera propietario de muchos artículos** o **muchos artículos pueden pertenecer a un mismo propietario**, entonces desde la perspectiva ``Article`` es una relación muchos a uno y desde la perspectiva ``User`` uno a muchos (corríjanme si estoy equivocado). En cuanto a ``tags`` es una relación muchos a muchos, una articulo puede pertenecer a muchas ``Tag`` y un ``Tag`` pueden contener muchos ``Article``.

Cuando se crea un campo **Foreign Key** añade un campo extra en la base de datos, en el caso de ``owner`` la tabla ``article`` creara un campo ``owner_id`` que sera la relación con ``auth_user.id``, en cuanto a ``tags``, creara una nueva tabla ``blog_article_tag`` con tres columnas, ``id``, ``article_id`` y ``tag_id``

Con esto en mente, vamos a crear el modelo en ``blog/models.py``, lo mas simple posible, ya mas adelante iremos añadiendo mas código.

.. code-block:: python

    from django.db import models
    from django.contrib.auth.models import User


    class Tag(models.Model):

        name = models.CharField(max_length=100, unique=True)
        slug = models.CharField(max_length=100, unique=True)


    class Article(models.Model):

        title = models.CharField(max_length=100, unique=True)
        slug = models.CharField(max_length=100, unique=True)
        body = models.TextField()
        owner = models.ForeignKey(User, related_name='article_owner')
        tags = models.ManyToManyField(Tag, related_name='article_tags')
        create_at = models.DateTimeField(auto_now_add=True)
        # update_at = models.DateTimeField(auto_now=True)

.. note::

    He dejado un campo comentado a propósito ``update_at = models.DateTimeField(auto_now=True)`` que mas tarde añadiremos, lo hago para mostrar como actúan las migraciones.

Creo que con lo que se comento anteriormente, mas lo que llevamos hecho con **accounts**, es bastante claro lo que hace, pero quizá haya un par de cosas a comentar, los argumentos de las propiedades, algunos argumentos como ``max_length`` en los clases ``CharField`` son obligatorios, y especifica el tamaño del campo en la base de datos, ademas, en los formularios, comprobara que el texto insertado por el usuario, no pase de la cantidad puesta. En cuanto a ``unique``, declaramos que el campo sera único.

Otro apunte son los campos ``owner`` y ``tags`` de la clase ``Article``, el primer argumento, hace referencia a la relación, en este caso la clase a la que tiene relación y ``related_name`` es un nombre al que podremos acceder cuando hagamos relación inversa, en el caso de ``Tag``, cuando queramos acceder al titulo, lo haremos de la siguiente manera ``tag_object.article_tags.title``. ``related_name``, es opcional y en caso de omitirlo, para acceder al campo ``title`` se haria ``tag_object.article.title``, es decir, nombre de clase relacional en minúsculas.

``auto_now_add`` y ``auto_now``, argumentos dice: ``auto_now_add`` inserta de manera implícita la fecha y hora (en este caso 'DateTime') cuando se crea un articulo y ``auto_now``, actualiza la fecha y hora, cuando se actualice un articulo.

.. note::

    En este caso, La clase ``Article`` esta debajo de ``Tag`` en el código Python y por eso ``tags = models.ManyToManyField(Tag)`` es posible poner ``Tag``, es decir el literal de clase, en caso de que la clase ``Tag`` estuviera debajo de ``Article``, para decirle que la relación es la clase ``Tag``, se deberá usar entre comillas, ``tags = models.ManyToManyField('Tag')``.

El siguiente paso es crear una migración (preparar los cambios) y migrar (crear los cambios en la base de datos).

.. code-block:: bash

    ./manage.py makemigrations blog

    Migrations for 'blog':
      0001_initial.py:
        - Create model Article
        - Create model Tag
        - Add field tags to article


Antes de crear la migración, vamos a ver que código **SQL** nos va a generar (el código es el generado para **SQLite**).

.. code-block:: sql

    ./manage.py sqlmigrate blog 0001_initial

    BEGIN;
    CREATE TABLE "blog_article" (
        "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
        "title" varchar(100) NOT NULL UNIQUE,
        "slug" varchar(100) NOT NULL UNIQUE,
        "body" text NOT NULL,
        "create_at" datetime NOT NULL,
        "owner_id" integer NOT NULL REFERENCES "auth_user" ("id")
    );
    CREATE TABLE "blog_tag" (
        "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
        "name" varchar(100) NOT NULL UNIQUE,
        "slug" varchar(100) NOT NULL UNIQUE
    );
    CREATE TABLE "blog_article_tags" (
        "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
        "article_id" integer NOT NULL REFERENCES "blog_article" ("id"),
        "tag_id" integer NOT NULL REFERENCES "blog_tag" ("id"), UNIQUE ("article_id", "tag_id")
    );
    CREATE INDEX "blog_article_5e7b1936" ON "blog_article" ("owner_id");
    CREATE INDEX "blog_article_tags_a00c1b00" ON "blog_article_tags" ("article_id");
    CREATE INDEX "blog_article_tags_76f094bc" ON "blog_article_tags" ("tag_id");

    COMMIT;

Si nos parece bien, ejecutamos la migración para hacer los cambios (en este caso crear las tablas) en la base de datos.

.. code-block:: bash

    ./manage.py migrate

    Operations to perform:
      Synchronize unmigrated apps: messages, staticfiles
      Apply all migrations: accounts, contenttypes, admin, auth, blog, sessions
    Synchronizing apps without migrations:
      Creating tables...
        Running deferred SQL...
      Installing custom SQL...
    Running migrations:
      Rendering model states... DONE
      Applying blog.0001_initial... OK

Como se puede ver en la base de datos, ahora se han creado las tres tablas. Antes comentemos un campo (propiedad) en la clase ``Article``, la descomentamos y ejecutamos ``makemigrations blog`` y ``migrate``.

.. code-block:: bash

    ./manage.py makemigrations blog

    You are trying to add a non-nullable field 'update_at' to article without a default; we can't do that (the database needs something to populate existing rows).
    Please select a fix:
     1) Provide a one-off default now (will be set on all existing rows)
     2) Quit, and let me add a default in models.py
    Select an option:

Nos esta diciendo que el campo ``update_at`` es un campo ``not null`` por lo que no puede añadir un campo sin datos (en este caso no hay filas, pero bueno :)), así que nos pregunta si queremos añadir un dato ahora o cambiar el modelo y añadir un dato por defecto.

En este caso, vamos a añadir la opcion **1º** y añadimos ``timezone.now()``.

.. code-block:: bash

    Select an option: 1
    Please enter the default value now, as valid Python
    The datetime and django.utils.timezone modules are available, so you can do e.g. timezone.now()
    >>> timezone.now()
    Migrations for 'blog':
      0002_article_update_at.py:
        - Add field update_at to article

Y hacemos la migración para actualizar la base de datos:

.. code-block:: bash

    ./manage.py migrate

Ahora el campo ``update_at`` ya esta en la base de datos :)

Antes de continuar, explicar que las clases de modelo creadas ``Tag`` y ``Article`` son subclases de ``django.db.models.Model``, que tienen una propiedad ``objects`` (por lo tanto ``Tag`` y ``Article`` también tienen la propiedad ``objects``), que es una clase ``django.db.models.Manager``. Es una interface a través de la cual las operaciones de consulta de base de datos se proporcionan a los modelos de Django, en resumen, el **manager** (gestor) de un modelo es un objeto a través del cual los modelos de Django realizan consultas de bases de datos. Cada modelo de Django tiene al menos un **manager**, y puedes crear **managers** personalizados con el fin de personalizar el acceso de base de datos.

Django tiene un argumento con ``./manage.py`` que es ``shell`` y es lo mismo que la consola interactiva de Python, pero que añade al **path** el proyecto (ejecuta internamente ``django.setup()``) y tenemos a nuestra disposición los modelos del proyecto.

Desde la consola vamos a ver 4 cosillas para interactuar con el ORM incorporado de Django y añadir, actualizar, etc filas en la base de datos.

.. code-block:: shell

    ./manage.py shell

Ahora, como si estuviéramos en un archivo **.py**, vamos a crear artículos, primero importamos los módulos y creamos un par de **tags**, después creamos algunos artículos.

Iré ``#`` comentando paso a paso las instrucciones, ``>>>`` es indicativo de que es una instrucción y se ha de omitir en la escritura en la terminal, por ultimo también mostrare la salida.

.. code-block:: python

    >>> from blog.models import Tag, Article

    # Comprobar cuantas etiquetas hay, para ello se usa el metodo all()
    # que obtiene una lista con todos los elementos existentes en la db (si los hay)
    >>> Tag.objects.all()
    []

    # Creamos un objeto Tag, insertamos datos y guardamos con save() el objeto
    # Con save(), guardara en la base de datos la fila
    >>> tag = Tag()
    >>> tag.name = 'Linux'
    >>> tag.slug = 'linux'
    >>> tag.save()
    >>> Tag.objects.all()
    [<Tag: Tag object>]

Como se puede ver, nos devuelve ``[<Tag: Tag object>]``, una lista con un elemento, vamos a modificar ``blog/models.py``

.. code-block:: python

    # blog/models.py

    class Tag(models.Model):

        # .....

        def __str__(self):
            return self.name


    class Article(models.Model):

        # .....

        def __str__(self):
            return self.title

Al haber realizado un cambio en el archivo ``models.py``, se ha de salir del interprete

.. code-block:: python

    >>> exit()

    ./manage.py shell

    >>> from blog.models import Tag, Article
    >>> Tag.objects.all()
    [<Tag: Linux>]

    # Vamos a crear 2 tags mas, pasando los datos en el 'constructor' de Tag
    >>> tag1 = Tag(name='Windows', slug='windows')
    >>> tag1.save()
    >>> Tag.objects.all()
    [<Tag: Linux>, <Tag: Windows>]
    >>> tag2 = Tag(name='Mac OS X', slug='mac-os-x')
    >>> tag2.save()
    >>> Tag.objects.all()
    [<Tag: Linux>, <Tag: Windows>, <Tag: Mac OS X>]

Ahora vamos a modificar un elemento, primero obtendremos el elemento que queremos modificar con ``filter(nombre_campo='valor_campo')``, donde ``nombre_campo``, es el nombre de la propiedad ``Tag`` y ``valor_campo`` es un valor, por ejemplo **Windows**. Nos devolverá siempre, una lista con 0 o mas elementos, tantos como campos tengan un valor **Windows** (en este caso, la propiedad ``name`` es ``unique``, por lo que obtendremos 0 o 1 elemento) y en este caso, si no existe, no lanzara una excepción (mas tarde veremos lo de **en este caso**)

Después de obtener el elemento (que sabemos de antemano, que sera 0 o 1 elemento), lo modificaremos y por ultimo actualizaremos los datos en la base de datos.

.. code-block:: python

    >>> Tag.objects.all()
    [<Tag: Linux>, <Tag: Windows>, <Tag: Mac OS X>]
    # Obtener el primer elemento
    >>> w = Tag.objects.filter(name='Windows')[0]
    >>> w
    <Tag: Windows>
    >>> type(w)
    <class 'blog.models.Tag'>
    >>> w.name = 'Microsoft Windows'
    >>> w.slug = 'microsoft-windows'
    >>> w.save()
    >>> w
    <Tag: Microsoft Windows>
    >>> Tag.objects.all()
    [<Tag: Linux>, <Tag: Microsoft Windows>, <Tag: Mac OS X>]

Si observamos en el filtro ``Tag.objects.filter(name='Windows')[0]`` el ``[0]``, es puro Python, la manera de obtener **x** elementos, Django lo traduce como:

* ``Tag.objects.filter(field='valor')[0]`` ``LIMIT 1`` El elemento que corresponda a X, el elemento 0 es el primer elemento.
* ``Tag.objects.filter(field='valor')[:5]`` ``LIMIT 5`` Los 5 primeros elementos
* ``Tag.objects.filter(field='valor')[5:10]`` ``OFFSET 5 LIMIT 5`` Cinco elementos a partir del 5 elemento.

Por lo tanto, solo devuelve 1 elemento y es el objeto, en caso de no utilizar ``[x]`` o ``[x:x]``, siempre devolverá una lista.

Así que ``w`` es un objeto ``Tag`` por lo que se puede acceder a sus propiedades y métodos directamente y lo que hacemos es modificar el ``name`` y ``slug``, por ultimo, guardamos los cambios en la base de datos.

Vamos primero a modificar el modelo, la clase ``django.db.models.Model`` tiene un método ``save()``, que se ejecuta justo antes de guardar/actualizar datos en la base de datos. Vamos a aprovecharlo para cambiar el **slug** dinamicamente, asi solo sera necesario cambiar/poner el ``name`` y justo antes de guardar/actualizar, Django(Python), nos cambiara el **slug**. A la vez, Django tiene una función para generar **slugs** validos que se encuentra en ``django.utils.text.slugify``.

.. code-block:: python

    # blog/models.py

    # Añadir al inicio
    from django.utils.text import slugify

    class Tag(models.Model):

        # ...

        def save(self, *args, **kwargs):
            self.slug = slugify(self.name)
            return super().save(*args, **kwargs)


    class Article(models.Model):

        # ...

        def save(self, *args, **kwargs):
            self.slug = slugify(self.title)
            return super().save(*args, **kwargs)

Vamos a cambiar de nuevo 'Microsoft Windows' por 'Windows' y a ver que pasa

.. code-block:: python

    >>> w = Tag.objects.filter(name='Microsoft Windows')[0]
    >>> w
    <Tag: Microsoft Windows>
    >>> w.name = 'Windows'
    >>> w.save()
    >>> Tag.objects.filter(name='Windows')[0].slug
    'windows'

Se puede observar, que ahora el slug es generado dinamicamente :)

Otro método de **Manager**, es ``get(**kwargs)``, como argumentos, acepta pares clave/valor, como ``filter()`` a excepción que siempre devuelve un solo elemento y si hay mas de un elemento lanzara ``MultipleObjectsReturned`` y que si no hay coincidencia, lanzara ``DoesNotExist``.

.. code-block:: python

    >>> Tag.objects.get(pk=1)
    <Tag: Linux>
    >>> Tag.objects.get(slug='windows')
    <Tag: Windows>
    >>> Tag.objects.get(id=1)
    <Tag: Linux>

    >>> Tag.objects.get(id=10)
    Traceback (most recent call last):
      File "<console>", line 1, in <module>
      File "/home/snicoper/.virtualenvs/tutorial_django/lib/python3.4/site-packages/django/db/models/manager.py", line 127, in manager_method
        return getattr(self.get_queryset(), name)(*args, **kwargs)
      File "/home/snicoper/.virtualenvs/tutorial_django/lib/python3.4/site-packages/django/db/models/query.py", line 334, in get
        self.model._meta.object_name
    blog.models.DoesNotExist: Tag matching query does not exist.

    >>> Tag.objects.get(name__icontains='i')
    Traceback (most recent call last):
      File "<console>", line 1, in <module>
      File "/home/snicoper/.virtualenvs/tutorial_django/lib/python3.4/site-packages/django/db/models/manager.py", line 127, in manager_method
        return getattr(self.get_queryset(), name)(*args, **kwargs)
      File "/home/snicoper/.virtualenvs/tutorial_django/lib/python3.4/site-packages/django/db/models/query.py", line 338, in get
        (self.model._meta.object_name, num)
    blog.models.MultipleObjectsReturned: get() returned more than one Tag -- it returned 2!



Aunque el campo de clave primaria ``pk`` es **id**, también es posible usar ``pk`` en los campos y Django, usara el nombre del campo que sea ``PRIMARY KEY`` (que por defecto es siempre ``id``).

``name__icontains`` es un **Field lookups** (Búsquedas de campo), contiene el nombre de la propiedad y ``__tipodebusqueda=valor``(dos guiones bajos), en este caso ``icontains`` y la ``i`` de **insensitive**, hace una búsqueda insensitiva (me van a matar los puristas del castellano, sry ^^), es decir en SQL seria algo así:

``Tag.objects.filter(name__icontains='algo')`` se traduciría a ``SELECT * FROM blog_tag WHERE name ILIKE '%algo%'``

Hay `muchas búsquedas de campo y le puedes echar un ojo en la documentación de Django <https://docs.djangoproject.com/en/1.8/ref/models/querysets/#field-lookups>`_ .

Ahora, vamos a crear algunas entradas.

.. code-block:: python

    # Importar Tag y Article y el usuario creado
    >>> from blog.models import Tag, Article
    >>> from django.contrib.auth.models import User
    >>> u = User.objects.get(pk=1)
    >>> u
    <User: snicoper>
    >>> u.email
    'snicoper@gmail.com'

    # Obtenemos 2 de 3 elementos tags que hay en la db
    >>> ts = Tag.objects.all()[1:]
    >>> ts
    [<Tag: Windows>, <Tag: Mac OS X>]

    # Creamos un articulo
    >>> a = Article()
    >>> a.title = 'Primer articulo'
    >>> a.body = 'Contenido del articulo'
    >>> a.owner = u
    # Es necesario guardar el objeto antes de añadir relaciones many to many
    >>> a.save()
    # y añadimos las relaciones, una lista con 2 elementos
    >>> a.tags.add(*ts)
    # Guardamos los cambios
    >>> a.save()

    # Comprobamos los resultados
    >>> article = Article.objects.get(title='Primer articulo')
    >>> article.owner
    <User: snicoper>
    >>> article.tags
    <django.db.models.fields.related.create_many_related_manager.<locals>.ManyRelatedManager object at 0x7fe19886a400>
    >>> article.tags.all()
    [<Tag: Windows>, <Tag: Mac OS X>]
    >>> article.slug
    'primer-articulo'

Ahora, vamos a ver cuantos artículos ha publicado el usuario

.. code-block:: python

    >>> from django.contrib.auth.models import User
    >>> u = User.objects.get(pk=1)

    # Accedemos al modelo Article, con el nombre de related_name que pusimos
    >>> u.article_owner
    <django.db.models.fields.related.create_foreign_related_manager.<locals>.RelatedManager object at 0x7f26c9422f98>
    >>> u.article_owner.all()
    [<Article: Primer articulo>]
    >>> u.article_owner.all()[0].title
    'Primer articulo'

    # Rizar el rizo
    # obtener el primer articulo de todos los que haya publicado el usuario
    # obtener la primera tag, de todas las que tenga el articulo y mostrar el name
    >>> u.article_owner.all()[0].tags.all()[0].name
    'Windows'

Poco a poco iremos viendo y comentado nuevos métodos que tiene ``Manager``, pero para empezar, creo que se hace uno una idea del tema, para ver mas sobre el tema, te recomiendo la `documentación <https://docs.djangoproject.com/en/1.8/topics/db/queries/>`_

En la siguiente sección, veremos un poco por encima la administración Django.

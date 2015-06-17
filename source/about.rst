.. _reference-about:

Pagina About
============

About (Sobre mi o Acerca de), la vamos a crear con la vista mas simple, ``View`` para mostrarla, tanto **about** como **contact**, crearemos las vistas en la **app home**.

Vamos a empezar como siempre, si, la vista...

.. code-block:: python

    # home/views.py

    # Añadimos al inicio
    from django.views import generic

    # Creamos la vista
    class AboutView(generic.View):

        def get(self, request, *args, **kwargs):
            return render(request, 'home/about.html')

La clase ``View`` es la clase de vista mas simple, la que menos propiedades y métodos tiene, 2 de los métodos que podemos usar es ``get()`` y ``post()`` que ejecutara uno u otro dependiendo del **method**. En este caso, ``get()`` simplemente renderiza una pagina **html** (una simple función lo podría a ver hecho, pero quería mostrar la clase ``View``).

Ahora ponemos la **url** en **URLconf**

.. code-block:: python

    # home/urls.py

    urlpatterns = [
        # ...
        url(regex=r'^about/$', view=views.AboutView.as_view(), name='about'),
    ]

la plantilla

.. code-block:: html

    <!-- blog/templates/blog/about.html -->

    {% extends 'base.html' %}

    {% block title %}About{% endblock title %}

    {% block content %}
        <h2 class="page-header">About</h2>
    {% endblock content %}

Y añadimos en ``base.html`` el link

.. code-block:: html

    <!-- templates/base.html -->

    <!-- buscar -->
    <li><a href="#about">About</a></li>

    <!-- cambiar por -->
    <li><a href="{% url 'about' %}">About</a></li>

El contenido de **about** es cosa de cada uno :)

Bueno, pues ya solo queda **contact** donde daremos la posibilidad a los usuarios que puedan contactar con nosotros, eso sera en la próxima sección.

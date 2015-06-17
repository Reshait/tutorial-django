.. _reference-blog-detalles_articulo:

Detalles articulo DetailView
============================

Ahora, vamos a añadir la pagina de detalles del articulo, donde mostrar el articulo en una pagina separada.

Los pasos, los de siempre la vista, la plantilla y crear la **url** en URLconf.

La vista, al igual que hicimos con ``ArticleListView`` que es una subclase de ``ListView``, ahora vamos hacer prácticamente lo mismo, ``ArticleDetailView`` que sera una subclase de ``DetailView``.

.. code-block:: python

    # blog/views.py

    class ArticleDetailView(generic.DetailView):

        template_model = 'blog/article_detail.html'
        model = Article
        context_object_name = 'article'

El template

.. code-block:: html

    <!-- blog/templates/blog/article_detail.html -->

    {% extends 'blog/base_blog.html' %}

    {% block title %}{{ article.title }}{% endblock title %}

    {% block blog_content %}
        <h2>{{ article.title }}</h2>
        <div class="article-info">
            <small>
                <strong>Por: </strong>{{ article.owner.username }}
                <strong>Hace: </strong>{{ article.create_at|timesince }}
                <strong>El: </strong>{{ article.create_at|date:'d F Y' }}
            </small><hr>
        </div>
        <p>{{ article.body|safe }}</p>
        <div class="article-footer">
            <strong>Etiquetas: </strong>{{ article.get_string_tags }}
        </div>

        <div class="comentarios">
            <!-- Aqui los comentarios -->
        </div>
    {% endblock blog_content %}

Se puede ver que es prácticamente igual que ``article_list.html`` omitiendo la paginacion, dentro de un rato, volveremos con este tema (para no repetirnos) y lo modificaremos.

en URLconf añadimos la siguiente **url**

.. code-block:: python

    # blog/urls.py

    urlpatterns = [
        # ...

        # /blog/detalles/:<string>slug/
        url(r'^detalle/(?P<slug>[-\w]+)/$', views.ArticleDetailView.as_view(), name='blog.article_detail'),
    ]

Por ultimo, tenemos que añadir un link en la lista de artículos para acceder a los detalles del articulo.

.. code-block:: html

    <!-- blog/templates/blog/article_list.html

    <!-- Cambiar <h2>{{ article.title }}</h2> por -->
    <h2><a href="{% url 'blog.article_detail' article.slug %}">{{ article.title }}</a></h2>

Como se puede observar, en ``{% url 'blog.article_detail' article.slug %}``, le pasamos el **slug** como parámetro, que lo requiere en la **URLconf** ``(?P<slug>[-\w]+)``. Por defecto, la vista ``ArticleDetailView``, para obtener el **item** a obtener del modelo exige la ``id`` o el ``slug`` (se puede cambiar con ``slug_url_kwarg``,  ``slug_field`` y ``pk_url_kwarg``)

Si vamos a la pagina, podemos ver que la lista de artículos, el titulo es un link que nos mandara al detalle del articulo.

Si recordamos, la platilla se repite, prácticamente es igual a la plantilla ``article_list.html``, la única diferencia es el link para ver el articulo en detalles.

Primero vamos a crear una nueva plantilla, ``_article.html`` y añadimos.

.. code-block:: html

    <!-- blog/templates/blog/_article.html -->

    {% if articles %}
        <h2><a href="{% url 'blog.article_detail' article.slug %}">{{ article.title }}</a></h2>
    {% else %}
        <h2>{{ article.title }}</h2>
    {% endif %}

    <div class="article-info">
        <small>
            <strong>Por: </strong>{{ article.owner.username }}
            <strong>Hace: </strong>{{ article.create_at|timesince }}
            <strong>El: </strong>{{ article.create_at|date:'d F Y' }}
        </small><hr>
    </div>
    <p>{{ article.body|safe }}</p>
    <div class="article-footer">
        <strong>Etiquetas: </strong>{{ article.get_string_tags }}
    </div>


Y modificamos ``article_detail.html`` y ``article_list.html``

.. code-block:: html

    <!-- blog/templates/blog/article_list.html -->

    <!-- la parte del {% for article in articles %} -->
    {% for article in articles %}
        {% include 'blog/_article.html' %}
    {% endfor %}

.. code-block:: html

    <!-- blog/templates/blog/article_detail.html -->

    {% extends 'blog/base_blog.html' %}

    {% block title %}{{ article.title }}{% endblock title %}

    {% block blog_content %}
        {% include 'blog/_article.html' %}

        <div class="comentarios">
            <!-- Aquí los comentarios -->
        </div>
    {% endblock blog_content %}

¿Como funciona?, cuando añadimos ``{% include 'blog/_article.html' %}`` importamos parte de un documento **html** en el mismo punto donde lo incluimos, por lo tanto, la plantilla incluida, tiene acceso al mismo **contexto**, por lo tanto en ``article_list.html`` tiene una variable de contexto ``articles`` mientras que en ``article_detail``, no. Ambos contextos, tienen la variable ``article`` que es el objeto de un modelo ``Article``, en el caso de ``article_list.html`` se genera dinamicamente dentro de **for** por lo que incluira tantas plantillas ``_article.html`` como artículos muestre y el objeto ``Article`` varia en cada **loop**

De esta manera, podemos tener una plantilla y un cambio se reflejara en ambas plantillas ``article_list.html`` y ``article_detail``.

Puedes observar, que he creado un comentario ``<!-- Aquí los comentarios -->`` en ``article_detail``, que seria para añadir el sistema `Disqus <https://disqus.com/>`_, pero no lo voy a incluir en el tutorial, te recomiendo un `articulo <http://www.snicoper.com/blog/article/anadir-sistema-de-comentarios-disqus/>`_ que cree en mi blog `www.snicoper.com <http://www.snicoper.com>`_

En la siguiente sección, vamos a crear el típico **leer mas...** y así crearemos nuestro primer **filtro**.

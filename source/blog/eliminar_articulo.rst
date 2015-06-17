.. _reference-blog-eliminar_articulo:

Eliminar articulo DeleteView
============================

Eliminar un articulo, también es muy fácil:

**La vista:**

.. code-block:: python

    # blog/views.py

    # Añadir al inicio
    from django.core.urlresolvers import reverse_lazy

    class ArticleDeleteView(generic.DeleteView):

        template_name = 'blog/confirmar_eliminacion.html'
        success_url = reverse_lazy('blog.article_list')
        model = Article

        def dispatch(self, request, *args, **kwargs):
            if not request.user.has_perms('blog.delete_article'):
                return redirect(settings.LOGIN_URL)
            return super().dispatch(request, *args, **kwargs)

Aquí se puede ver que hemos usado ``success_url``, por que si lo dejamos sin poner como anteriormente, cuando lea del model ``get_absolute_url``, dará un error, por que el articulo no existe.

``reverse_lazy`` es igual que ``reverse`` que ya hemos usado varias veces, la diferencia, es que ``reverse_lazy`` es útil usarla cuando URLconf aun no se ha cargado.

**URLconf:**

.. code-block:: python

    # blog/urls.py

    urlpatterns = [
        url(r'^eliminar/(?P<slug>[-\w]+)/$', views.ArticleDeleteView.as_view(), name='blog.eliminar'),
    ]

**La plantila:**

.. code-block:: html

    <!-- blog/templates/blog/confirmar_eliminacion.html -->

    {% extends 'blog/base_blog.html' %}

    {% block title %}Confirmar la eliminación{% endblock title %}

    {% block blog_content %}
        <h2 class="page-header">Confirmacion para eliminar {{ article.title }}</h2>
        <form method="post" action="">
            {% csrf_token %}
            <p>¿Seguro que quieres eliminar el articulo {{ article.title }}?</p>
            <button type="submit" class="btn btn-danger">Si, eliminar</button>
            <a href="{% url 'blog.article_detail' article.slug %} class="btn btn-success">Cancelar</a>
        </form>
    {% endblock blog_content %}

Y añadimos un enlace en detalles del articulo.

.. code-block:: html

    <!-- blog/templates/blog/article_detail.html -->

    <!-- añadir antes de <div class="comentarios"> -->
    {% if perms.article.can_delete %}
        <a href="{% url 'blog.eliminar' article.slug %}" class="btn btn-danger">Eliminar</a>
    {% endif %}

Ahora, nos queda la pagina **about** y **contact** que veremos en las próximas secciones.

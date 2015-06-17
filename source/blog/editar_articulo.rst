.. _reference-blog-editar_articulo:

Editar articulo UpdateView
==========================

Con editar ya si que no vamos a ver nada nuevo, prácticamente es lo mismo que la creación, lo único que cambia es (internamente), es que el formulario, muestra los datos de un articulo a modificar.

Tampoco necesitamos saber que usuario lo ha creado (no lo cambiaremos), así que pondré los pasos e iré mas rápido y pondré las capturas para una vista rápida.

**La vista**

.. code-block:: python

    # blog/views.py

    class ArticleUpdateView(generic.UpdateView):

        template_model = 'blog/article_form.html'
        model = Article
        form_class = ArticleCreateForm

        def dispatch(self, request, *args, **kwargs):
            # Al igual que con ArticleCreateView, dejo al lector
            # que cambie el comportamiento de este método para saber
            # si esta logueado y tiene permisos.
            # Ver el comentario de ArticleCreateView en el método
            # dispatch
            if not request.user.has_perms('blog.change_article'):
                return redirect(settings.LOGIN_URL)
            return super().dispatch(request, *args, **kwargs)

        def get_context_data(self, **kwargs):
            # Obtenemos el contexto de la clase base
            context = super().get_context_data(**kwargs)
            # añadimos nuevas variables de contexto al diccionario
            context['title'] = 'Editar articulo'
            context['nombre_btn'] = 'Editar'
            # devolvemos el contexto
            return context

``dispatch`` cambia el tipo de permisos, ahora comprueba que el usuario pueda editar y en el contexto ``get_context_data``, cambia el **title** y **nombre_btn**.

La plantilla es la misma que la de crear articulo, así que pasamos a la **URLconf**

.. code-block:: python

    # blog/urls.py

    # Añadir

    urlspatterns = [
        # ...
        url(r'^editar/(?P<slug>[-\w]+)/$', views.ArticleUpdateView.as_view(), name='blog.editar'),
    ]

Vamos a añadir un link en los detalles del articulo y probar si puede editar, que muestre el botón para editar.

.. code-block:: html

    <!-- blog/templates/blog/article_detail.html -->

    <!-- añadimos debajo de {% include 'blog/_article.html' %} -->

    {% if perms.article.can_change %}
        <a href="{% url 'blog.editar' article.slug %}" class="btn btn-primary">Editar</a>
    {% endif %}

Y ya esta!! nada mas :)

**Como ejercicio para el lector:** Te propongo que cuando creas y editas un articulo, si el formulario es valido (si se ha creado o editado un articulo), muestre un mensaje al usuario como lo hace la función ``logout_view`` de ``accounts/views.py`` cuando te deslogueas.

En la siguiente sección ya terminamos el sistema **CRUD** del blog y prácticamente terminamos este pequeño tutorial.

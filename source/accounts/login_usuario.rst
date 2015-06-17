.. _reference-login_usuario:

Login usuario
=============

Django ya incorpora funciones para hacer **login** (e incluso vistas y formularios, pero los vamos a omitir en este tutorial).

En primer lugar creamos la vista

.. code-block:: python

    # accounts/views.py

    # Añadimos
    from django.contrib.auth import authenticate, login
    from django.contrib.auth.decorators import login_required


    @login_required
    def index_view(request):
        return render(request, 'accounts/index.html')


    def login_view(request):
    # Si el usuario esta ya logueado, lo redireccionamos a index_view
    if request.user.is_authenticated():
        return redirect(reverse('accounts.index'))

    mensaje = ''
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        user = authenticate(username=username, password=password)
        if user is not None:
            if user.is_active:
                login(request, user)
                return redirect(reverse('accounts.index'))
            else:
                # Redireccionar informando que la cuenta esta inactiva
                # Lo dejo como ejercicio al lector :)
                pass
        mensaje = 'Nombre de usuario o contraseña no valido'
    return render(request, 'accounts/login.html', {'mensaje': mensaje})

La vista, lo primero que comprueba es si esta autenticado, si lo esta, mostrar el formulario es **tontería** y lo redirecciona a la pagina principal ``index_view``, después comprueba si la respuesta viene por el método **POST**, si no lo es, renderiza la plantilla ``accounts/login.html``, que seria la primea vez que se accede a la pagina.

En caso contrario, significa que la respuesta es por método **POST**, entonces obtiene los valores de los campos ``username`` y ``password`` del formulario con ``request.POST.get('nombre_campo')``

Ahora, comprueba una autenticación, que lo que hace es comprobar si **username** y **password** coinciden con un usuario en la base de datos y en caso de éxito devolverá un objeto con el usuario, en caso contrario devolverá ``None`` con lo que podemos redirecionar otra vez al formulario con el mensaje **Nombre de usuario o contraseña no valido.**

Si ``user`` no es ``None``, significa que es un objeto ``User`` y comprueba si el usuario tiene una cuanta activa con ``if user.is_active``, si no la tiene, se debería cambiar el mensaje, para que el usuario sea consciente de por que no puede hacer login. De lo contrario, si todo ha ido bien, el usuario se loguea y lo redirecciona a la vista ``index_view``.

La vista ``index_view``, tiene un decorador que comprueba si esta logueado o no, si no lo esta, redireccionara a la pagina de login, de lo contrario, podrá acceder a la vista.

.. note::

    el decorador acepta un parámetro (entre otros) ``@login_required(login_url='')``, en caso de omitir el parámetro redireccionara al valor de ``tutorial_django.settings.LOGIN_URL``.

Ahora vamos a crear las dos plantillas, la del formulario para hacer login y la pagina **index** de **accounts**, esta ultima pagina, simplemente mostrara un mensaje de bienvenida y la foto del usuario.

.. code-block:: bash

    touch accounts/templates/accounts/{index.html,login.html}

.. code-block:: html

    # accounts/templates/accounts/index.html

    {% extends 'base.html' %}

    {% block title %}Perfil de usuario{% endblock title %}

    {% block content %}
    <div class="page-header"><h2><h2>Perfil de usuario</h2></div>
        Hola de nuevo {{ user.username }}<br>
        <div>
            <img src="{{ MEDIA_URL }}{{ user.userprofile.photo }}" alt="Imagen de {{ user.username }}" />
        </div>
    {% endblock content %}

Se puede observar que cuando queremos mostrar un archivo **media**, lo hacemos con ``{{ MEDIA_URL }}``, con esto, obtiene la **url** de ``settings.MEDIA_URL``, el resto, lo obtiene con ``{{ user.userprofile.photo }}``. El objeto **user** esta accesible en todos las plantillas, accede a ``UserProfile`` con el mismo nombre pero en minúsculas y finalmente accede a la propiedad ``photo``

.. code-block:: html

    # accounts/templates/accounts/login.html

    {% extends 'base.html' %}

    {% block title %}Login{% endblock title %}

    {% block content %}
        <div class="row">
            <div class="col-md-4 col-md-offset-4">
                <div class="page-header"><h2>Login</h2></div>
                {% if mensaje %}
                    <div class="alert alert-danger">
                        {{ mensaje }}
                    </div>
                {% endif %}

                <form method="post" action="">
                    {% csrf_token %}
                    <div class="form-group">
                        <label class="control-label" for="username">Nombre de usuario</label>
                        <input type="text" id="username" name="username" class="form-control" value="{{ username }}">
                    </div>
                    <div class="form-group">
                        <label for="password">Contraseña</label>
                        <input type="password" name="password" id="password" class="form-control">
                    </div>
                    <button type="submit" class="btn btn-primary">Login</button>
                </form>
            </div>
        </div>
    {% endblock content %}

Por ultimo, tenemos que añadir las dos **urls** en el **URLconf**.

.. code-block:: python

    # accounts/urls.py

    # Añadir dentro de urlpatterns

    urlpatterns = [
        url(r'^$', views.index_view, name='accounts.index'),
        url(r'^login/$', views.login_view, name='accounts.login'),

        # ...
    ]

Vamos a ver si todo funciona mas o menos bien :P, para ello, si estas logueado (hasta ahora, la única manera de hacerlo era a través de la administración) y entras a `http://127.0.0.1:8000/accounts/login/ <http://127.0.0.1:8000/accounts/login/>`_, veras que te redirecciona a `http://127.0.0.1:8000/accounts/ <http://127.0.0.1:8000/accounts/>`_ (así que, deslogueate desde la administración y prueba de nuevo).

Y si lo haces al revés, si no estas logueado e intentas acceder a `http://127.0.0.1:8000/accounts/ <http://127.0.0.1:8000/accounts/>`_, te redireccionara a `http://127.0.0.1:8000/accounts/login/ <http://127.0.0.1:8000/accounts/login/>`_.

¿No puedes ver la imagen?, :), primero en ``tutorial_django/settings`` en la lista ``TEMPLATES``, hay otra lista ``context_processors``, asegúrate que ``'django.template.context_processors.media',`` esta incluido en la lista (en Django 1.8, no viene por defecto), a parte, cuando estés con el servidor de desarrollo, en ``totorial_django/urls.py`` inserta lo siguiente:

.. code-block:: python

    # tutorial_django/urls.py

    # Añade esto, al inicio del documento
    from django.conf import settings

    # Añade esto, al final del documento
    if settings.DEBUG:
        # static files (images, css, javascript, etc.)
        urlpatterns.append(
            # /media/:<mixed>path/
            url(
                regex=r'^media/(?P<path>.*)$',
                view='django.views.static.serve',
                kwargs={'document_root': settings.MEDIA_ROOT}))

Prueba ahora a ver si puedes ver la imagen!

Pues ya tenemos casi terminado el sistema de usuarios, queda el contrario, poder hacer logout, que sera en la próxima sección y la manera de que el usuario pueda modificar sus datos.


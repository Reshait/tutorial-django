.. _reference-editar_perfil_usuario:

Editar perfil de usuario
========================

Ahora tenemos que darle la oportunidad al usuario de poder modificar datos, como cambiar el email, contraseña y la foto.

Para empezar, vamos a modificar la pagina principal de **accounts** para añadir links a la edición por separado de los datos.

Vamos a añadir un menú lateral, para poner los links y después, añadiremos tres plantillas, ``editar_contrasena.html``, ``editar_foto.html``, ``editar_email.html``.

Como las tres paginas (a parte de **index.html**), incluirán el menú lateral, seria repetirse cuatro veces la parte del menú, por lo que vamos a crear una plantilla **_menu.html**.

Aparte, necesitamos una plantilla para representa el menú en un lado y el contenido, en el resto de la pagina, crearemos también una platilla **base_accounts.html** que represente ambos espacios.

Vamos a crear los archivos:

.. code-block:: bash

    touch accounts/templates/accounts/{base_accounts.html,editar_contrasena.html,editar_foto.html,editar_email.html}

Editamos ``accounts/templates/accounts/base_accounts.html``

.. code-block:: html

    <!-- accounts/templates/accounts/base_accounts.html -->

    {% extends 'base.html' %}

    {% block content %}
        <div class="row">
            <div class="col-md-2">
                <div class="list-group">
                    <a href="#" class="list-group-item">Editar email</a>
                    <a href="#" class="list-group-item">Editar contraseña</a>
                    <a href="#" class="list-group-item">Editar foto</a>
                </div>
            </div>
            <div class="col-md-10">
                {% block accounts_content %}{% endblock accounts_content %}
            </div>
        </div>
    {% endblock content %}

Mas tarde, ya añadiremos los links, por ahora, para hacerse una idea ya vale.

Editamos ``accounts/templates/accounts/index.html``

.. code-block:: html

    <!-- accounts/templates/accounts/index.html -->

    {% extends 'accounts/base_accounts.html' %}

    {% block title %}Perfil de usuario{% endblock title %}

    {% block accounts_content %}
    <div class="page-header"><h2><h2>Perfil de usuario</h2></div>
        Hola de nuevo {{ user.username }}<br>
        <div>
            <img src="{{ MEDIA_URL }}{{ user.userprofile.photo }}" alt="Imagen de {{ user.username }}" />
        </div>
    {% endblock accounts_content %}

Los cambios (``{% extends 'accounts/base_accounts.html' %}`` y los blocks ``{% block accounts_content %}`` y ``{% endblock accounts_content %}``), son fáciles de entender, en vez de usar ``extends`` de ``base.html`` (``base_accounts.html`` usa ``extends`` a ``base.html``, por lo que no perderemos el diseño anterior) usara uno para que las cuatro paginas, tengan el mismo aspecto.

Editar Email
************

Empecemos con editar email, necesitamos un formulario de un solo campo, después el email ha de ser único en la base de datos (si lo ha cambiado, comprobar que no exista uno igual), en caso de éxito, redireccionamos otra vez a **index**, en caso contrario, se lo notificaremos a usuario.

El formulario

.. code-block:: python

    # accounts/forms.py

    class EditarEmailForm(forms.Form):

        email = forms.EmailField(
            widget=forms.EmailInput(attrs={'class': 'form-control'}))

        def __init__(self, *args, **kwargs):
            """Obtener request"""
            self.request = kwargs.pop('request')
            return super().__init__(*args, **kwargs)

        def clean_email(self):
            email = self.cleaned_data['email']
            # Comprobar si ha cambiado el email
            actual_email = self.request.user.email
            username = self.request.user.username
            if email != actual_email:
                # Si lo ha cambiado, comprobar que no exista en la db.
                # Exluye el usuario actual.
                existe = User.objects.filter(email=email).exclude(username=username)
                if existe:
                    raise forms.ValidationError('Ya existe un email igual en la db.')
            return email

La vista

.. code-block:: python

    # accounts/views.py

    @login_required
    def editar_email(request):
        if request.method == 'POST':
            form = EditarEmailForm(request.POST, request=request)
            if form.is_valid():
                request.user.email = form.cleaned_data['email']
                request.user.save()
                messages.success(request, 'El email ha sido cambiado con exito!.')
                return redirect(reverse('accounts.index'))
        else:
            form = EditarEmailForm(
                request=request,
                initial={'email': request.user.email})
        return render(request, 'accounts/editar_email.html', {'form': form})

Es la misma rutina de siempre, comprueba el método, y actúa en consecuencia, lo único distinto, es ver como pasamos el objeto ``HttpRequest`` al instanciar el formulario y como rellenamos datos en el formulario con el argumento ``initial``.

La plantilla

.. code-block:: html

    <!-- accounts/templates/accounts/editar_email.html -->

    {% extends 'accounts/base_accounts.html' %}

    {% block title %}Editar email{% endblock title %}

    {% block accounts_content %}
        <h2 class="page-header">Editar Email</h2>
        <form method="post" action="">
            {% csrf_token %}
            {{ form.as_p }}
            <button class="btn btn-primary" type="submit">Actualizar Email</button>
            <a href="{% url 'accounts.index' %}" class="btn btn-warning" type="submit">Cancelar</a>
        </form>
    {% endblock accounts_content %}

URLconf

.. code-block:: python

    # accounts/urls.py

    # Añadimos en urlpatterns
    url(r'^editar_email/$', views.editar_email, name='accounts.editar_email'),

Y actualizamos el link en ``accounts/templates/accounts/base_accounts.html``

.. code-block:: html

    <!-- accounts/templates/accounts/base_accounts.html -->

    <a href="{% url 'accounts.editar_email' %}" class="list-group-item">Editar email</a>

Editar contraseña
*****************

La contraseña requiere de tres campos, uno con la contraseña actual, otro para insertar nueva contraseña y un ultimo para repetir de nueva contraseña.

El formulario

.. code-block:: python

    # accounts/forms.py

    class EditarContrasenaForm(forms.Form):

        actual_password = forms.CharField(
            label='Contraseña actual',
            min_length=5,
            widget=forms.PasswordInput(attrs={'class': 'form-control'}))

        password = forms.CharField(
            label='Nueva contraseña',
            min_length=5,
            widget=forms.PasswordInput(attrs={'class': 'form-control'}))

        password2 = forms.CharField(
            label='Repetir contraseña',
            min_length=5,
            widget=forms.PasswordInput(attrs={'class': 'form-control'}))

        def clean_password2(self):
            """Comprueba que password y password2 sean iguales."""
            password = self.cleaned_data['password']
            password2 = self.cleaned_data['password2']
            if password != password2:
                raise forms.ValidationError('Las contraseñas no coinciden.')
            return password2

La vista

.. code-block:: python

    # accounts/views.py

    # Añadir al inicio
    from django.contrib.auth.hashers import make_password

    # Modificar al inicio
    from .forms import (
        RegistroUserForm, EditarEmailForm, EditarContrasenaForm)

    # Añadir al final
    @login_required
    def editar_contrasena(request):
        if request.method == 'POST':
            form = EditarContrasenaForm(request.POST)
            if form.is_valid():
                request.user.password = make_password(form.cleaned_data['password'])
                request.user.save()
                messages.success(request, 'La contraseña ha sido cambiado con exito!.')
                messages.success(request, 'Es necesario introducir los datos para entrar.')
                return redirect(reverse('accounts.index'))
        else:
            form = EditarContrasenaForm()
        return render(request, 'accounts/editar_contrasena.html', {'form': form})


Observa como usamos ``make_password()`` para generar un **password** con **hash** (no se traducir esto, lo siento :)), es muy importante, ya que si no, guardara la contraseña en texto plano y es un gran error por motivos de seguridad!.

(Lo pongo aquí, aunque seria parte del ``EditarContrasenaForm``), también hay una función ``check_password() <https://docs.djangoproject.com/en/1.4/topics/auth/#django.contrib.auth.hashers.check_password>`_, que podríamos a ver comprobado en un método ``clean_actual_password()`` y comprobar si ``actual_password`` es igual a **password** informar al usuario que esta usando la misma contraseña que la actual (lo dejo como ejercicio para el lector).

La plantilla

.. code-block:: html

    <!-- accounts/templates/accounts/editar_contrasena.html -->

    {% extends 'accounts/base_accounts.html' %}

    {% block title %}Editar email{% endblock title %}

    {% block accounts_content %}
        <h2 class="page-header">Editar contraseña</h2>
        <form method="post" action="">
            {% csrf_token %}
            {{ form.as_p }}
            <button class="btn btn-primary" type="submit">Actualizar contraseña</button>
            <a href="{% url 'accounts.index' %}" class="btn btn-warning" type="submit">Cancelar</a>
        </form>
    {% endblock accounts_content %}


El URLconf

.. code-block:: python

    # accounts/urls.py

    # Añadir a urlpatterns
    url(r'^editar_contrasena/$', views.editar_contrasena, name='accounts.editar_contrasena'),

Actualizar ``base_accounts.html``

.. code-block:: html

    <!-- accounts/templates/accounts/base_accounts.html -->

    <a href="{% url 'accounts.editar_contrasena' %}" class="list-group-item">Editar contraseña</a>

Ya solo queda editar la imagen, pero lo dejo como ejercicio para el lector, ademas, también dejo como ejercicio, si nos fijamos, las plantillas ``editar_x`` son prácticamente iguales, es decir nos repetimos demasiado!!, intenta que con una plantilla muestra los datos que quieres mostrar. Como pista podrías crear una sola pagina de formulario y dentro de la pagina añadir variables de contexto ``{{ titulo }}``, etc y pasarlas de las vistas a las plantillas.

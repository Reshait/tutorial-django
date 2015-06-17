.. _reference-contact:

Pagina Contacto
===============

Django proporciona una envoltura de **smtplib** para que el envió de correos electrónicos sea una tarea muy sencilla. Para empezar, es necesario añadir variables en el archivo de configuración ``tutorial_django/settings.py``

Al igual que **about**, lo haremos en la **app home**. Añadimos al final:

.. code-block:: python

    # tutorial_django/settings.py

    EMAIL_USE_TLS = True
    EMAIL_HOST = 'smtp.gmail.com'
    EMAIL_PORT = 587
    EMAIL_HOST_USER = 'usuario@gmail.com'
    EMAIL_HOST_PASSWORD = 'contraseña'

Esta es la configuración típica para usar **Gmail** como **SMTP**, modifica los datos de tu cuenta **Gmail**.

Vamos a empezar con los formularios. La idea es la siguiente, si un usuario esta logueado (tenemos su email de contacto), no mostrara el campo de email, en caso contrario, le pediremos su email por si tenemos que contactar con el.

.. code-block:: bash

    touch home/forms.py

    from django import forms


    class ContactUsuarioAnonimoForm(forms.Form):

        email = forms.EmailField(
            label='Email',
            widget=forms.EmailInput(attrs={'class': 'form-control'})
        )
        subject = forms.CharField(
            label='Asunto',
            widget=forms.TextInput(attrs={'class': 'form-control'})
        )
        body = forms.CharField(
            label='Mensaje',
            widget=forms.Textarea(attrs={'class': 'form-control'})
        )


    class ContactUsuarioLoginForm(forms.Form):

        subject = forms.CharField(
            label='Asunto',
            widget=forms.TextInput(attrs={'class': 'form-control'})
        )
        body = forms.CharField(
            label='Mensaje',
            widget=forms.Textarea(attrs={'class': 'form-control'})
        )


Creamos 2 formularios, uno para el usuario logueado y otro para el usuario anónimo.

La vista, en esta ocasión, vamos a usar otra **CBV** que es un ``FormView``

.. code-block:: python

    # home/views.py

    # Añadimos en el inicio
    from django.contrib import messages
    from django.core.urlresolvers import reverse_lazy
    from django.core.mail import send_mail
    from .forms import ContactUsuarioAnonimoForm, ContactUsuarioLoginForm

    # Creamos una función para el envió de email
    # (es muy simple, solo para demostrar como enviar un email)
    def send_email_contact(email_usuario, subject, body):
        body = '{} ha enviado un email de contacto\n\n{}\n\n{}'.format(email_usuario, subject, body)
        send_mail(
            subject='Nuevo email de contacto',
            message=body,
            from_email='contact@example.com',
            recipient_list=['usuario@example.com']
        )

    class ContactView(generic.FormView):

        template_name = 'home/contact.html'
        success_url = reverse_lazy('home')

        def dispatch(self, request, *args, **kwargs):
            if request.user.is_authenticated():
                self.form_class = ContactUsuarioLoginForm
            else:
                self.form_class = ContactUsuarioAnonimoForm
            return super().dispatch(request, *args, **kwargs)

        def form_valid(self, form):
            subject = form.cleaned_data.get('subject')
            body = form.cleaned_data.get('body')
            if self.request.user.is_authenticated():
                email_usuario = self.request.user.email
                send_email_contact(email_usuario, subject, body)
            else:
                email_usuario = form.cleaned_data.get('email')
                send_email_contact(email_usuario, subject, body)
            messages.success(self.request, 'Email enviado con exito')
            return super().form_valid(form)

Por defecto, un ``FormView`` requiere una propiedad ``form_class``, en este caso, como no sabemos que formulario vamos a usar, en el método ``dispatch`` (es el método encargado en saber el **method** del tipo de respuesta) comprobamos si el usuario esta logueado o no y dependiendo de si lo esta o no, asignaremos a ``form_class`` un formulario u otro.

Después ya la clase ``ContactView`` se encarga de renderizar la plantilla, que la lee de ``template_name``.

Ahora, tenemos que generar el método ``form_valid`` y decirle que envié el email con ``django.core.mail.send_mail``, una función que acepta los siguientes parámetros

.. code-block:: python

    send_mail(subject, message, from_email, recipient_list, fail_silently=False, auth_user=None, auth_password=None, connection=None, html_message=None)

Volvemos hacer la comprobación de si es un usuario autenticado, si lo esta, recuperamos el email con ``self.request.user.email`` y si no lo es, entonces el usuario ha introducido el email en el formulario y lo recuperamos con ``form.cleaned_data.get('email')``.

Después pasamos los datos a la función ``send_email_contact``, que prepara el email (de una manera muy simple, solo para mostrar como funciona) y lo envía de una manera muy sencilla (recuerda configurar las variables de configuración en ``tutorial_django/settings.py``)

Pues yo creo que ya tenemos un blog (básico) creado, con la funcionalidad básica de cualquier formulario, y por ahora ya lo dejamos, espero que lo hayas disfrutado y te haya servido de algo el tutorial.

Ya solo unas palabras de despedida en la siguiente sección.

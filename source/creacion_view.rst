.. _reference-creacion_view:

Creación de la Vista
====================

Las ``views`` (controladores en otros Frameworks), es el puente entre los ``templates`` (vistas en otros Frameworks) y el modelo.

En este primer contacto, no vamos a usar los modelos para mantener las cosas lo mas simples posible, así que vamos a escribir la vista mas sencilla posible.

Abrimos ``home/views.py`` y escribimos lo siguiente:

.. code-block:: python

    # home/views.py

    from django.http import HttpResponse


    def index_view(request):
        return HttpResponse("Hello world")

Tenemos que escribir una ``url`` para que al acceder ella, Django y el sistema de enrrutamiento  sepa a que ``view`` ha de acceder.



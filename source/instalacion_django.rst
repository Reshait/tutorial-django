.. _reference-instalacion_django:

Instalación Django
==================

Antes de instalar Django, se ha de tener Python en el sistema, para no repetirme dejo aquí unas guías para instalar Python y Django en varios sistemas.

* `Instalación Python en Windows <http://snicoper.readthedocs.org/es/latest/windows/instalacion_python_windows.html>`_
* `Instalación Python en Fedora <http://snicoper.readthedocs.org/es/latest/linux/python/instalacion_python_fedora.html>`_
* `Instalación Python en Ubuntu <http://snicoper.readthedocs.org/es/latest/linux/python/instalar_python_ubuntu.html>`_

Crear un entorno virtual
************************

La mejor manera de desarrollar con Python, es usando entornos virtuales, de esta manera podemos tener versiones de paquetes sin que se mezclen.

Para crear un entorno virtual, abrimos la terminal y escribimos:

.. code-block:: bash

    mkvirtualenv tutorial_django

Ahora en la terminal, podemos ver que pone ``(terminal_django)``, eso nos indica que entorno estamos usando.

Para salir del entorno virtual, simplemente con ``deactivate`` en la terminal, saldremos y para entrar o cambiar en entorno, usamos ``workon nombre_entorno``.

Instalación de Django
*********************

Para instalar Django, usando el entorno virtual, escribimos:

.. note::

    Omito ``(terminal_django)snicoper@lxmaq1 ~/projects/tutorial_django`` en la terminal cuando pongo los comandos, asumo que a partir de ahora siempre se estará usando el entorno virtual ``terminal_django``.

.. code-block:: bash

    pip install django

Una vez instalado, comprobamos si todo ha salido bien:

.. code-block:: bash

    python -c 'import django; print(django.get_version())'
    # 1.8.2

Si nos muestra la versión, es que todo ha salido correcto y ahora podemos crear nuestro primer proyecto.

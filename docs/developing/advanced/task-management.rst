###############
Task Management
###############

Dependencies
============

Task management using Celery is available in Arches if you have a message broker like RabbitMQ or Redis. The Celery documentation
provides information on `broker installation <https://docs.celeryproject.org/en/stable/getting-started/first-steps-with-celery.html#choosing-a-broker>`_.

Configuration
=============

Once you have your broker available, you will need to configure your settings. You will find this in your project's
settings.py file. Each setting that begins with the CELERY prefix will be used as a celery config, so you can configure
celery by adding the configs you need

.. code-block:: python

    CELERY_BROKER_URL = 'amqp://guest:guest@localhost'
    CELERY_RESULT_BACKEND = 'django-db'  # Use 'django-cache' if you want to use your cache as your backend

The settings you are likely to want to modify right away are the CELERY_BROKER_URL and the CELERY_RESULT_BACKEND.
Your CELERY_BROKER_URL should point to your broker's service URL. If you are using RabbitMQ, you will probably want to
create a new user and password and replace 'guest:guest' with the new users credentials.

Your CELERY_RESULT_BACKEND is set to the Django ORM by default. If your task will be run frequently, you may want to
consider a more performant option. If you've configured a cache for django, you could use that, or you could use another
`backend option <https://docs.celeryproject.org/en/latest/userguide/configuration.html#result-backend>`_ that Celery supports.


Adding Tasks to Your Project
============================

To add additional tasks you your project, all you need to do is add a tasks.py file to your project. This could be placed
in your project's root directory (next to manage.py) or any sub-directory. However, you will likely want to put it - at
least to start in the directory below root (next to urls.py).

Here's an example of a very simple task in tasks.py:

.. code-block:: python


    from __future__ import absolute_import, unicode_literals
    from celery import shared_task

    @shared_task
    def add(x, y):
        return x + y


To call your task, just import tasks into the module that will run it.


Running Celery
==============

For your tasks to run both your broker service and a Celery worker need to be running. For production you will likely
want to run a worker as service. One way to do this is to use supervisord. For more information see: :ref:`setting-up-supervisord-for-celery`

However for development, you probably want to run your worker in a terminal. To do so just cd into your project's root
directory with your virtual environment activated and run:

.. code-block:: bash

    python manage.py celery start

Users of Apple Silicon Macs may encounter ``billiard.exceptions.WorkerLostError`` following a warning
emitted by the OS explaining it is "crashing instead" rather than unsafely calling ``fork()``. Until this
`issue <https://github.com/celery/celery/issues/7324>`_ is resolved by ``celery``, launching like so will
silence the errors:

.. code-block:: bash

    OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES python manage.py celery start

Note that in the event of celery errors which do not clearly indicate what is breaking or preventing your tasks from succeeding, we recommend running the following command instead for the purpose of debugging and isolating any unhandled exceptions:

.. code-block:: bash

    celery -A [app_name] worker --loglevel=warning


Once the worker is running you should be able check your database and see you task result output with the following
query:

.. code-block:: bash

    select * from django_celery_results_taskresult;

If you want to monitor your tasks with a realtime console, you can use `Flower <https://flower.readthedocs.io/en/latest/>`_.

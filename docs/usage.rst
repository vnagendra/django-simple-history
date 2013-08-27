Usage
=====

Install
-------

This package is available on `PyPI`_ and `Crate.io`_.

Install from PyPI with ``pip``:


.. code-block:: bash

    $ pip install django-simple-history

.. _pypi: https://pypi.python.org/pypi/django-simple-history/
.. _crate.io: https://crate.io/packages/django-simple-history/


Quickstart
----------

Add simple_history to your ``INSTALLED_APPS`` in the settings.py file for the project

.. code-block:: python

	INSTALLED_APPS = (
    	...,
    	...,
    	'polls',
    	'simple_history',
	)

To track history for a model, create an instance of
``simple_history.models.HistoricalRecords`` on the model.

An example for tracking changes on the ``Poll`` and ``Choice`` models in the
Django tutorial:

.. code-block:: python

    from django.db import models
    from simple_history.models import HistoricalRecords

    class Poll(models.Model):
        question = models.CharField(max_length=200)
        pub_date = models.DateTimeField('date published')
        history = HistoricalRecords()

    class Choice(models.Model):
        poll = models.ForeignKey(Poll)
        choice_text = models.CharField(max_length=200)
        votes = models.IntegerField(default=0)
        history = HistoricalRecords()

Now all changes to ``Poll`` and ``Choice`` model instances will be tracked in
the database.


Integration with Django Admin
-----------------------------

To allow viewing previous model versions on the Django admin site, inherit from
the ``simple_history.admin.SimpleHistoryAdmin`` class when registering your
model with the admin site.

This will replace the history object page on the admin site and allow viewing
and reverting to previous model versions.  Changes made in admin change forms
will also accurately note the user who made the change.

An example of admin integration for the ``Poll`` and ``Choice`` models:

.. code-block:: python

    from django.contrib import admin
    from simple_history.admin import SimpleHistoryAdmin
    from .models import Poll, Choice

    admin.site.register(Poll, SimpleHistoryAdmin)
    admin.site.register(Choice, SimpleHistoryAdmin)


Querying history
----------------

Querying history on a model instance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``HistoricalRecords`` object on a model instance can be used in the same
way as a model manager:

.. code-block:: pycon

    >>> from polls.models import Poll, Choice
    >>> from datetime import datetime
    >>> poll = Poll.objects.create(question="what's up?", pub_date=datetime.now())
    >>>
    >>> poll.history.all()
    [<HistoricalPoll: Poll object as of 2010-10-25 18:03:29.855689>]

Whenever a model instance is saved a new historical record is created:

.. code-block:: pycon

    >>> poll.pub_date = datetime(2007, 4, 1, 0, 0)
    >>> poll.save()
    >>> poll.history.all()
    [<HistoricalPoll: Poll object as of 2010-10-25 18:04:13.814128>, <HistoricalPoll: Poll object as of 2010-10-25 18:03:29.855689>]

Querying history on a model class
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Historical records for all instances of a model can be queried by using the
``HistoricalRecords`` manager on the model class.  For example historical
records for all ``Choice`` instances can be queried by using the manager on the
``Choice`` model class:

.. code-block:: pycon

    >>> choice1 = poll.choice_set.create(choice_text='Not Much', votes=0)
    >>> choice2 = poll.choice_set.create(choice_text='The sky', votes=0)
    >>>
    >>> Choice.history
    <simple_history.manager.HistoryManager object at 0x1cc4290>
    >>> Choice.history.all()
    [<HistoricalChoice: Choice object as of 2010-10-25 18:05:12.183340>, <HistoricalChoice: Choice object as of 2010-10-25 18:04:59.047351>]


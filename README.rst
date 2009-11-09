-------------
django-db-log
-------------

Logs Django exceptions to your database handler.

=======
Install
=======

The easiest way to install the package is via setuptools::

	easy_install django-db-log

Once installed, update your settings.py and add the middleware and installed apps settings::

	MIDDLEWARE_CLASSES = (
	    'django.middleware.common.CommonMiddleware',
	    'django.contrib.sessions.middleware.SessionMiddleware',
	    'django.contrib.auth.middleware.AuthenticationMiddleware',
	    ...
	    'djangodblog.middleware.DBLogMiddleware',
	)

	INSTALLED_APPS = (
	    'django.contrib.admin',
	    'django.contrib.auth',
	    'django.contrib.contenttypes',
	    'django.contrib.sessions',
	    'djangodblog',
	    ...
	)

Finally, run ``python manage.py syncdb`` to create the database tables.

=============
Configuration
=============

Several options exist to configure django-db-log via your ``settings.py``:

######################
DBLOG_CATCH_404_ERRORS
######################

Enable catching of 404 errors in the logs. Default value is ``False``.

##############
DBLOG_DATABASE
##############

Warning: This feature is currently in the testing phase.

Use a secondary database to store error logs. This is useful if you have several websites and want to aggregate error logs onto one database server::

	DBLOG_DATABASE = dict(
	    DATABASE_ENGINE='mysql', # defaults to settings.DATABASE_ENGINE
	    DATABASE_NAME='my_db_name',
	    DATABASE_USER='db_user',
	    DATABASE_PASSWORD='db_pass',
	    DATABASE_HOST='localhost', # defaults to localhost
	    DATABASE_PORT='', # defaults to [default port]
	    DATABASE_OPTIONS={}
	)

Some things to note:

* You will need to create the tables by hand if you use this option. Use ``python manage.py sql djangodblog`` and dump that SQL into the correct server.
* This functionality does not yet support Django 1.2.

=====
Usage
=====

You will find two new admin panels in the automatically built Django administration:

* Errors (Error)
* Error batches (ErrorBatch)

It will store every single error inside of the `Errors` model, and it will store a collective, or summary, of errors inside of `Error batches` (this is more useful for most cases). If you are using this on multiple sites with the same database, the `Errors` table also contains the SITE_ID for which it the error appeared on.

If you wish to access these within your own views and models, you may do so via the standard model API::

	from djangodblog.models import Error, ErrorBatch
	
	ErrorBatch.objects.all().order_by('-last_seen')

You can also record errors outside of middleware if you want::

	from djangodblog.models import Error
	
	try:
		...
	except Exception, exc:
		Error.objects.create_from_exception(exc, [url=None])

=====
Notes
=====

* django-db-log will automatically integrate with django-idmapper.
* Multi-db support (via ``DBLOG_DATABASE``) will most likely not work in Django 1.2
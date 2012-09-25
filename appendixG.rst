====================================
Appendix G: The django-admin Utility
====================================

``django-admin.py`` is Django's command-line utility for administrative tasks.
This appendix explains its many powers.

You'll usually access ``django-admin.py`` through a project's ``manage.py``
wrapper. ``manage.py`` is automatically created in each Django project and is a
thin wrapper around ``django-admin.py``. It takes care of two things for you
before delegating to ``django-admin.py``:

    * It puts your project's package on ``sys.path``.

    * It sets the ``DJANGO_SETTINGS_MODULE`` environment variable so that it
      points to your project's ``settings.py`` file.

The ``django-admin.py`` script should be on your system path if you installed
Django via its ``setup.py`` utility. If it's not on your path, you can find it in
``site-packages/django/bin`` within your Python installation. Consider
symlinking it from some place on your path, such as ``/usr/local/bin``.

Windows users, who do not have symlinking functionality available,
can copy ``django-admin.py`` to a location on their existing path or edit the
``PATH`` settings (under Settings ~TRA Control Panel ~TRA System ~TRA Advanced ~TRA 
Environment) to point to its installed location.

Generally, when working on a single Django project, it's easier to use
``manage.py``. Use ``django-admin.py`` with ``DJANGO_SETTINGS_MODULE`` or the
``--settings`` command-line option, if you need to switch between multiple
Django settings files.

The command-line examples throughout this appendix use ``django-admin.py`` to
be consistent, but any example can use ``manage.py`` just as well.

Usage
=====

The basic usage is::

    django-admin.py action [options]

or::

    manage.py action [options]

``action`` should be one of the actions listed in this document. ``options``,
which is optional, should be zero or more of the options listed in this
document.

Run ``django-admin.py --help`` to display a help message that includes a terse
list of all available actions and options.

Most actions take a list of app names. An *app name* is the base name of the
package containing your models. For example, if your ``INSTALLED_APPS`` contains
the string ``'mysite.blog'``, the app name is ``blog``.

Available Actions
=================
The following sections cover the actions available to you.

adminindex [appname appname ...]
--------------------------------

Prints the admin-index template snippet for the given application names. Use
admin-index template snippets if you want to customize the look and feel of your
admin's index page.

createcachetable [tablename]
----------------------------

Creates a cache table named ``tablename`` for use with the database cache
back-end. See Chapter 13 for more about caching.

dbshell
-------

Runs the command-line client for the database engine specified in your
``DATABASE_ENGINE`` setting, with the connection parameters specified in the settings
``DATABASE_USER``, ``DATABASE_PASSWORD``, and so forth.

    * For PostgreSQL, this runs the ``psql`` command-line client.
    
    * For MySQL, this runs the ``mysql`` command-line client.
    
    * For SQLite, this runs the ``sqlite3`` command-line client.

This command assumes the programs are on your ``PATH`` so that a simple call to
the program name (``psql``, ``mysql``, or ``sqlite3``) will find the program in the
right place. There's no way to specify the location of the program manually.

diffsettings
------------

Displays differences between the current settings file and Django's default
settings.

Settings that don't appear in the defaults are followed by ``"###"``. For
example, the default settings don't define ``ROOT_URLCONF``, so
``ROOT_URLCONF`` is followed by ``"###"`` in the output of ``diffsettings``.

Note that Django's default settings live in ``django.conf.global_settings``,
if you're ever curious to see the full list of defaults.

dumpdata [appname appname ...]
------------------------------

Outputs to standard output all data in the database associated with the named
application(s).

By default, the database will be dumped in JSON format. If you want the output
to be in another format, use the ``--format`` option (e.g., ``format=xml``).
You may specify any Django serialization back-end (including any user-specified
serialization back-ends named in the ``SERIALIZATION_MODULES`` setting). The
``--indent`` option can be used to pretty-print the output.

If no application name is provided, all installed applications will be dumped.

The output of ``dumpdata`` can be used as input for ``loaddata``.

flush
-----

Returns the database to the state it was in immediately after syncdb was
executed. This means that all data will be removed from the database, any
postsynchronization handlers will be re-executed, and the ``initial_data``
fixture will be reinstalled.

inspectdb
---------

Introspects the database tables in the database pointed to by the
``DATABASE_NAME`` setting and outputs a Django model module (a ``models.py``
file) to standard output.

Use this if you have a legacy database with which you'd like to use Django.
The script will inspect the database and create a model for each table within
it.

As you might expect, the created models will have an attribute for every field
in the table. Note that ``inspectdb`` has a few special cases in its field name
output:

    * If ``inspectdb`` cannot map a column's type to a model field type, it will
      use ``TextField`` and will insert the Python comment
      ``'This field type is a guess.'`` next to the field in the generated
      model.

    * If the database column name is a Python reserved word (such as
      ``'pass'``, ``'class'``, or ``'for'``), ``inspectdb`` will append
      ``'_field'`` to the attribute name. For example, if a table has a column
      ``'for'``, the generated model will have a field ``'for_field'``, with
      the ``db_column`` attribute set to ``'for'``. ``inspectdb`` will insert
      the Python comment
      ``'Field renamed because it was a Python reserved word.'`` next to the
      field.

This feature is meant as a shortcut, not as definitive model generation. After
you run it, you'll want to look over the generated models yourself to make
customizations. In particular, you'll need to rearrange the models so that
models with relationships are ordered properly.

Primary keys are automatically introspected for PostgreSQL, MySQL, and
SQLite, in which case Django puts in the ``primary_key=True`` where
needed.

``inspectdb`` works with PostgreSQL, MySQL, and SQLite. Foreign key detection
only works in PostgreSQL and with certain types of MySQL tables.

loaddata [fixture fixture ...]
------------------------------

Searches for and loads the contents of the named fixture into the database.

A *fixture* is a collection of files that contain the serialized contents of
the database. Each fixture has a unique name; however, the files that
comprise the fixture can be distributed over multiple directories, in
multiple applications.

Django will search in three locations for fixtures:

   * In the ``fixtures`` directory of every installed application
   * In any directory named in the ``FIXTURE_DIRS`` setting
   * In the literal path named by the fixture

Django will load any and all fixtures it finds in these locations that match
the provided fixture names.

If the named fixture has a file extension, only fixtures of that type
will be loaded. For example, the following::

    django-admin.py loaddata mydata.json

will only load JSON fixtures called ``mydata``. The fixture extension
must correspond to the registered name of a serializer (e.g., ``json`` or
``xml``).

If you omit the extension, Django will search all available fixture types
for a matching fixture. For example, the following::

    django-admin.py loaddata mydata

will look for any fixture of any fixture type called ``mydata``. If a fixture
directory contained ``mydata.json``, that fixture would be loaded
as a JSON fixture. However, if two fixtures with the same name but different
fixture types are discovered (e.g., if ``mydata.json`` and
``mydata.xml`` were found in the same fixture directory), fixture
installation will be aborted, and any data installed in the call to
``loaddata`` will be removed from the database.

The fixtures that are named can include directory components. These
directories will be included in the search path. The following, for example::

    django-admin.py loaddata foo/bar/mydata.json

will search ``<appname>/fixtures/foo/bar/mydata.json`` for each installed
application,  ``<dirname>/foo/bar/mydata.json`` for each directory in
``FIXTURE_DIRS``, and the literal path ``foo/bar/mydata.json``.

Note that the order in which fixture files are processed is undefined. However,
all fixture data is installed as a single transaction, so data in
one fixture can reference data in another fixture. If the database back-end
supports row-level constraints, these constraints will be checked at the
end of the transaction.

The ``dumpdata`` command can be used to generate input for ``loaddata``.

.. admonition:: MySQL and Fixtures

    Unfortunately, MySQL isn't capable of completely supporting all the
    features of Django fixtures. If you use MyISAM tables, MySQL doesn't
    support transactions or constraints, so you won't get a rollback if
    multiple transaction files are found, or validation of fixture data.
    If you use InnoDB tables, you won't be able to have any forward
    references in your data files -- MySQL doesn't provide a mechanism to
    defer checking of row constraints until a transaction is committed.

reset [appname appname ...]
---------------------------
Executes the equivalent of ``sqlreset`` for the given app names.

runfcgi [options]
-----------------
Starts a set of FastCGI processes suitable for use with any Web server
that supports the FastCGI protocol. See Chapter 20 for more about deploying under FastCGI.

This command requires the Python FastCGI module from ``flup``
(http://www.djangoproject.com/r/flup/).

runserver [optional port number, or ipaddr:port]
------------------------------------------------

Starts a lightweight development Web server on the local machine. By default,
the server runs on port 8000 on the IP address 127.0.0.1. You can pass in an
IP address and port number explicitly.

If you run this script as a user with normal privileges (recommended), you
might not have access to start a port on a low port number. Low port numbers
are reserved for the superuser (root).

.. warning::

    **Do not use this server in a production setting**. It has not gone through
    security audits or performance tests, and there are no plans to change that
    fact. Django's developers are in the business of making Web frameworks, not
    Web servers, so improving this server to be able to handle a production
    environment is outside the scope of Django.

The development server automatically reloads Python code for each request, as
needed. You don't need to restart the server for code changes to take effect.

When you start the server, and each time you change Python code while the
server is running, the server will validate all of your installed models. (See
the upcoming section on the ``validate`` command.) If the validator finds errors, 
it will print them to standard output, but it won't stop the server.

You can run as many servers as you want, as long as they're on separate ports.
Just execute ``django-admin.py runserver`` more than once.

Note that the default IP address, 127.0.0.1, is not accessible from other
machines on your network. To make your development server viewable to other
machines on the network, use its own IP address (e.g., 192.168.2.1) or
0.0.0.0.

For example, to run the server on port 7000 on IP address 127.0.0.1, use this::

    django-admin.py runserver 7000

Or to run the server on port 7000 on IP address 1.2.3.4, use this::

    django-admin.py runserver 1.2.3.4:7000

Serving Static Files with the Development Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

By default, the development server doesn't serve any static files for your site
(such as CSS files, images, things under ``MEDIA_ROOT_URL``, etc.). If
you want to configure Django to serve static media, read about serving static
media at http://www.djangoproject.com/documentation/0.96/static_files/.

Turning Off Autoreload
~~~~~~~~~~~~~~~~~~~~~~~

To disable autoreloading of code while the development server is running, use the
``--noreload`` option, like so::

    django-admin.py runserver --noreload

shell
-----

Starts the Python interactive interpreter.

Django will use IPython (http://ipython.scipy.org/) if it's installed. If you
have IPython installed and want to force use of the "plain" Python interpreter,
use the ``--plain`` option, like so::

    django-admin.py shell --plain

sql [appname appname ...]
-------------------------

Prints the ``CREATE TABLE`` SQL statements for the given app names.

sqlall [appname appname ...]
----------------------------

Prints the ``CREATE TABLE`` and initial-data SQL statements for the given app names.

Refer to the description of ``sqlcustom`` for an explanation of how to
specify initial data.

sqlclear [appname appname ...]
------------------------------

Prints the ``DROP TABLE`` SQL statements for the given app names.

sqlcustom [appname appname ...]
-------------------------------

Prints the custom SQL statements for the given app names.

For each model in each specified app, this command looks for the file
``<appname>/sql/<modelname>.sql``, where ``<appname>`` is the given app name and
``<modelname>`` is the model's name in lowercase. For example, if you have an
app ``news`` that includes a ``Story`` model, ``sqlcustom`` will attempt
to read a file ``news/sql/story.sql`` and append it to the output of this
command.

Each of the SQL files, if given, is expected to contain valid SQL. The SQL
files are piped directly into the database after all of the models'
table-creation statements have been executed. Use this SQL hook to make any
table modifications, or insert any SQL functions into the database.

Note that the order in which the SQL files are processed is undefined.

sqlindexes [appname appname ...]
--------------------------------

Prints the ``CREATE INDEX`` SQL statements for the given app names.

sqlreset [appname appname ...]
------------------------------

Prints the ``DROP TABLE`` SQL, and then the ``CREATE TABLE`` SQL, for the given app
names.

sqlsequencereset [appname appname ...]
--------------------------------------

Prints the SQL statements for resetting sequences for the given app names. 

You'll need this SQL only if you're using PostgreSQL and have inserted data by
hand. When you do that, PostgreSQL's primary key sequences can get out of sync
from what's in the database, and the SQL emitted by this command will clear it
up.

startapp [appname]
------------------

Creates a Django application directory structure for the given app name in the current
directory.

startproject [projectname]
--------------------------

Creates a Django project directory structure for the given project name in the
current directory.

syncdb
------

Creates the database tables for all applications in ``INSTALLED_APPS`` whose tables have
not already been created.

Use this command when you've added new applications to your project and want to
install them in the database. This includes any applications shipped with Django that
might be in ``INSTALLED_APPS`` by default. When you start a new project, run
this command to install the default applications.

If you're installing the ``django.contrib.auth`` application, ``syncdb`` will
give you the option of creating a superuser immediately. ``syncdb`` will also
search for and install any fixture named ``initial_data``. See the documentation
for ``loaddata`` for details on the specification of fixture data files.

test
----

Discovers and runs tests for all installed models. Testing was still under
development when this book was being written, so to learn more you'll need to
read the documentation online at
http://www.djangoproject.com/documentation/0.96/testing/.

validate
--------

Validates all installed models (according to the ``INSTALLED_APPS`` setting) and
prints validation errors to standard output.

Available Options
=================

The sections that follow outline the options that ``django-admin.py`` can take.

--settings
----------

Example usage::

    django-admin.py syncdb --settings=mysite.settings

Explicitly specifies the settings module to use. The settings module should be
in Python package syntax (e.g., ``mysite.settings``). If this isn't provided,
``django-admin.py`` will use the ``DJANGO_SETTINGS_MODULE`` environment
variable.

Note that this option is unnecessary in ``manage.py``, because it takes care of
setting ``DJANGO_SETTINGS_MODULE`` for you.

--pythonpath
------------

Example usage::

    django-admin.py syncdb --pythonpath='/home/djangoprojects/myproject'

Adds the given filesystem path to the Python import search path. If this isn't
provided, ``django-admin.py`` will use the ``PYTHONPATH`` environment variable.

Note that this option is unnecessary in ``manage.py``, because it takes care of
setting the Python path for you.

--format
--------

Example usage::

    django-admin.py dumpdata --format=xml

Specifies the output format that will be used. The name provided must be the name
of a registered serializer.

--help
------

Displays a help message that includes a terse list of all available actions and
options.

--indent
--------

Example usage::

    django-admin.py dumpdata --indent=4

Specifies the number of spaces that will be used for indentation when
pretty-printing output. By default, output will *not* be pretty-printed.
Pretty-printing will only be enabled if the indent option is provided.

--noinput
---------

Indicates you will not be prompted for any input. This is useful if the 
``django-admin`` script will be executed as an unattended, automated script.

--noreload
----------

Disables the use of the autoreloader when running the development server.

--version
---------

Displays the current Django version.

Example output::

    0.9.1
    0.9.1 (SVN)

--verbosity
-----------

Example usage::

    django-admin.py syncdb --verbosity=2

Determines the amount of notification and debug information that
will be printed to the console. ``0`` is no output, ``1`` is normal output,
and ``2`` is verbose output.

--adminmedia
------------

Example usage::

    django-admin.py --adminmedia=/tmp/new-admin-style/

Tells Django where to find the various CSS and JavaScript files for the admin
interface when running the development server. Normally these files are served
out of the Django source tree, but because some designers customize these files
for their site, this option allows you to test against custom versions.
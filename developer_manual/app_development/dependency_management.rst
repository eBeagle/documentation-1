.. _app-dependencies:

=====================
Dependency management
=====================

It's possible to build a Nextcloud app with existing software packages.

.. _app-composer:

Composer
--------

You can add 3rd party php packages with `Composer`_. Composer will download the specified packages to a directory of your choice, typically to ``/vendor``. In order to benefit from Composer's autoloader, you'll want to add a ``require_once`` to the ``register`` method of your ``Application`` class in the :ref:`bootstrapping<Bootstrapping>` code of your app.

.. _app-composer-dependency-hell:

Dependency hell
^^^^^^^^^^^^^^^

Be careful with which packages you add to an app. PHP can not load two version of the same class twice, hence there can be conflicts between Nextcloud Server and an app or between two or more apps if they require the same package. So try to keep the number of production dependencies to a minimum and see :ref:`app-composer-bin-tools`.

Conflict example
****************

To illustrate the problem imagine app *A* depends on package *foo* in version 1, app *B* depends on package *foo* in version 2. The package *foo* had a breaking change to which app *B* has been adjusted, *A* uses the old API.

Both apps ship a Composer autoloader that autoloads the *foo* functions and classes. There is a race between the two autoloaders. If *A*'s autoloader is asked to load the class first, then v1 will be used. If *B*'s autoloader loads functions and classes first it will be v2. In some scenarios there might be classes of v1 and v2 when autoloaders are invoked without a defined order.

Depending on which functions and classes are loaded, app *A* might work or break. The same applies to *B*.

.. _app-composer-bin-tools:

Development tools
^^^^^^^^^^^^^^^^^

It is very common for an app to use CLI tools for syntax checks, testing and building. Since many tools depend on common Composer packages like ``psr/*`` and ``symfony/console``, it is likely that apps produce a :ref:`dependency hell <app-composer-dependency-hell>` on development environments.

The dependency hell for CLI tools can be avoided by using the *Composer bin plugin*. It's a composer plugin that puts development dependencies into sub directorory with a dedicated autoloader. That autoloader is only used if the CLI tool is used. For Nextcloud apps this means two apps can use conflicting versions of one tool. Moreover dependency conflicts between the tools of one app are no longer an issue.

Tools known to be problematic that should be moved into bin plugin directories include

* ``friendsofphp/php-cs-fixer``
* ``phpunit/phpunit``
* ``vimeo/psalm``

Please see the `package page <https://packagist.org/packages/bamarni/composer-bin-plugin>`_ for up-to-date installation instructions.

.. _Composer: https://getcomposer.org/

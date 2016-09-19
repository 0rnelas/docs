3.4 Migration Guide
###################

CakePHP 3.4 is an API compatible upgrade from 3.3. This page outlines the
changes and improvements made in 3.4.

Deprecations
============

The following is a list of deprecated methods, properties and behaviors. These
elements will continue to function until 4.0.0 after which they will be removed.

* The public properties on ``Cake\Event\Event`` are deprecated, new methods have
  been added to read/write the relevant properties.
* Several properties on ``Cake\Network\Request`` have been deprecated:

  * ``Request::$params`` is deprecated. Use ``Request::param()`` instead.
  * ``Request::$data`` is deprecated. Use ``Request::data()`` instead.
  * ``Request::$query`` is deprecated. Use ``Request::query()`` instead.
  * ``Request::$cookies`` is deprecated. Use ``Request::cookie()`` instead.
  * ``Request::$base`` is deprecated. Use ``Request::getAttribute('base')`` instead.
  * ``Request::$webroot`` is deprecated. Use ``Request::getAttribute('webroot')`` instead.
  * ``Request::$here`` is deprecated. Use ``Request::here()`` instead.
  * ``Request::$_session`` was renamed to ``Request::$session``.

* A number of methods on ``Cake\Network\Request`` have been deprecated:

  * The setter modes for ``query()``, ``data()`` and ``param()`` are deprecated.
  * ``__get()`` & ``__isset()`` methods are deprecated. Use ``param()`` instead.
  * ``method()`` is deprecated. Use ``getMethod()`` instead.
  * ``setInput()`` is deprecated. Use ``withBody()`` instead.
  * The ``ArrayAccess`` methods have all been deprecated.

* The ``Auth.redirect`` session variable is no longer used.

Behavior Changes
================

While these changes are API compatible, they represent minor variances in
behavior that may effect your application:

* ORM\Query results will not typecast aliased columns based on the original
  columns type. For example if you alias ``created`` to ``created_time`` you
  will now get a ``Time`` object back instead of a string.
* ``AuthComponent`` now uses a query string to store the redirect URL when an
  unauthenticated user is redirected to the login page. Previously, this redirect
  was stored in the session. Using the query string allows for better
  multi-browser experience.

Event
=====

* ``Event::data()`` was added.
* ``Event::setData()`` was added.
* ``Event::result()`` was added.
* ``Event::setResult()`` was added.


I18n
====

* You can now customize the behavior of the fallback message loader. See
  :ref:`creating-generic-translators` for more information.

PaginatorHelper
===============

* ``PaginatorHelper::numbers()`` now uses an HTML elipsis instead of '...' in
  the default templates.

FormHelper
==========

* You can now configure the sources which FormHelper reads from. This makes
  creating GET forms simpler. See :ref:`form-values-from-query-string` for more
  information.

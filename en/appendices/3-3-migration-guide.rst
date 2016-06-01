3.3 Migration Guide
###################

CakePHP 3.3 is an API compatible upgrade from 3.2. This page outlines the
changes and improvements made in 3.3.

Deprecations
============

* ``Router::mapResources()`` is deprecated. Use routing scopes and
  ``$routes->resources()`` instead.
* ``Router::redirect()`` is deprecated. Use routing scopes and
  ``$routes->redirect()`` instead.
* ``Router::parseNamedParams()`` is deprecated. Named parameter backwards
  compatibility will be removed in 4.0.0
* ``Cake\Http\Response`` has had the following methods deprecated because they
  overlap with PSR7 interface methods:

  * ``statusCode()`` use ``getStatusCode()`` instead.
  * ``encoding()`` use ``getEncoding()`` instead.
  * ``header()`` use ``getHeaderLine()`` instead.
  * ``cookie()`` use ``getCookie()`` instead.
  * ``version()`` use ``getProtocolVersion()`` instead.

Behavior Changes
================

While these changes are API compatible, they represent minor variances in
behavior that may effect your application:

* The default JSON encode format for Date and DateTime instances is now
  ISO-8601. This means that the timezone value contains a ``:``.
  For example ``2015-11-06T00:00:00+03:00``
* ``Controller::referer()`` now consistently omits the application base path
  when generating application local URLs. Previously string URLs would have the
  base path prepended to them, while array URLs would not.

Http Client is now PSR7 Compatible
==================================

``Cake\Network\Http\Client`` has been moved to ``Cake\Http\Client``. Its request
and response objects now implement the
`PSR7 interfaces <http://www.php-fig.org/psr/psr-7/>`__. Several methods on
``Cake\Http\Client\Response`` are now deprecated, see above for more information.

Routing
=======

* ``Router::parse()``, ``RouteCollection::parse()`` and ``Route::parse()`` had
  a ``$method`` argument added. It defaults to 'GET'. This new parameter reduces
  reliance on global state, and necessary for the PSR7 work integration to be done.
* When building resource routes, you can now define a prefix. This is useful
  when defining nested resources as you can create specialized controllers for
  nested resources.

Console
=======

* Shell tasks that are invoked directly from the CLI no longer have their
  ``_welcome`` method invoked. They will also have the ``requested`` parameter
  set now.
* ``Shell::err()`` will now apply the 'error' style to text. The default
  styling is red text.

Request
=======

* ``Request::is()`` and ``Request::addDetector()`` now supports additional
  arguments in detectors. This allows detector callables to operate on
  additional parameters.

ORM
===

* Additional support has been added for mapping complex data types. This makes
  it easier to work with geo-spatial types, and data that cannot be represented
  by strings in SQL queries. See the
  :ref:`mapping-custom-datatypes-to-sql-expressions` for more information.
* A new ``JsonType`` was added. This new type lets you use the native JSON types
  available in MySQL and Postgres. In other database providers the ``json`` type
  will map to ``TEXT`` columns.
* ``Association::unique()`` was added. This method proxies the target table's
  ``unique()`` method, but ensures that association conditions are applied.
* ``isUnique`` rules now apply association conditions.
* When entities are converted into JSON, the associated objects are no longer
  converted to an array first using ``toArray()``. Instead, the
  ``jsonSerialize()`` method will be invoked on all associated entities. This
  gives you more flexibility and control on which properties are exposed in JSON
  representations of your entities.
* ``Table::newEntity()`` and ``Table::patchEntity()`` will now raise an
  exception when an unknown association is in the 'associated' key.

Validation
==========

* ``Validator::requirePresence()`` now accepts a list of fields. This allows you
  to more concisely define the fields that are required.
* ``Validator::requirePresence()``, ``Validator::allowEmpty()`` and
  ``Validator::notEmpty()`` now accept a list of fields. This allows you
  to more concisely define the fields that are required.

Debugging Functions
===================

* The ``pr()``, ``debug()``, and ``pj()`` functions now return the value being
  dumped. This makes them easier to use when values are being returned.

Cache Shell Added
=================

To help you better manage cached data from a CLI environment, a shell command
has been added that exposes methods for clearing cached data::

    // Clear one cache config
    bin/cake cache clear <configname>

    // Clear all cache configs
    bin/cake cache clear_all

3.6 Migration Guide
###################

CakePHP 3.6 est une mise à jour de CakePHP 3.5 dont la compatibilité API est
complète. Cette page souligne les changements et améliorations faits dans 3.6.

Pour mettre à jour vers 3.6.x, lancez la commande suivante :

.. code-block:: bash

    php composer.phar require --update-with-dependencies "cakephp/cakephp:3.6.*"

Deprecations
============

La liste qui suit regroupe les méthodes, les propriétés et les comportements
dépréciés. Ces différents éléments continueront de fonctionner jusqu'à la
version 4.0.0, à partir de laquelle ils seront supprimés.

* ``bin/cake orm_cache`` est maintenant ``bin/cake schema_cache``.

Changement de Comportements
============================

Les changements suivants sont compatibles avec l'API, mais elles représentent
des écarts mineurs de comportement qui pourrait affecter votre application:

* ``Cake\Utility\Security::randomBytes()`` lancera maintenant une exception
  lorsqu'une source sécurisée d'entropie ne peut pas être trouvée en PHP5.

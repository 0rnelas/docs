Behavior CounterCache
#####################

.. php:namespace:: Cake\ORM\Behavior

.. php:class:: CounterCacheBehavior

Souvent les applications web doivent afficher le nombre d'objets liés. Par
exemple, quand vous montrez une liste d'articles, vous voulez peut-être
afficher combien de commentaires ils ont. Ou quand vous montrez un utilisateur,
vous voulez montrer le nombre d'amis/de followers qu'il a. Le behavior
CounterCache est présent pour ces situations.

Usage Basique
=============

Vous activez le behavior CounterCache comme tous les autres behaviors, mais
il ne fera rien jusqu'à ce que vous configuriez quelques relations et le
nombre de champs qui doivent être stockés sur chacun d'eux. Utiliser notre
exemple ci-dessous, nous pourrions mettre en cache le nombre de commentaires
pour chaque article avec ce qui suit::

    class CommentsTable extends Table
    {
        public function initialize(array $config)
        {
            $this->addBehavior('CounterCache', [
                'Articles' => ['comment_count']
            ]);
        }
    }

La configuration de CounterCache doit être composée de noms de relations et
de la configuration spécifique pour cette relation.

La valeur du compteur sera mise à jour à chaque fois qu'une entity est
sauvegardée ou supprimée. Le compteur **ne va pas** être mis à jour lorsque
vous utilisez ``updateAll()`` ou ``deleteAll()``, ou que vous executez du SQL
que vous avez écrit.

Usage Avancée
=============

Si vous avez besoin de garder un compteur mis en cache pour moins que tous les
enregistrements liés, vous pouvez fournir des conditions supplémentaires ou
des méthodes finder pour générer une valeur du compteur::

    // Utilise une méthode find spécifique.
    // Dans ce cas find(published)
    $this->addBehavior('CounterCache', [
        'Articles' => [
            'comment_count' => [
                'findType' => 'published'
            ]
        ]
    ]);

Si vous n'avez pas de méthode de finder personnalisé, vous pouvez fournir
un tableau de conditions pour trouver les enregistrements à la place::

    $this->addBehavior('CounterCache', [
        'Articles' => [
            'comment_count' => [
                'conditions' => ['Comments.spam' => false]
            ]
        ]
    ]);

Enfin, si un finder personnalisé et les conditions ne sont pas réunies, vous
pouvez fournir une méthode de callback. Cette méthode retourne la valeur du
compteur à stocker::

    $this->addBehavior('CounterCache', [
        'Articles' => [
            'rating_avg' => function ($event, $entity, $table) {
                return 4.5;
            }
        ]
    ]);

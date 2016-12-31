Pagination
##########

One of the main obstacles of creating flexible and user-friendly web
applications is designing an intuitive UI. Many applications tend to
grow in size and complexity quickly, and designers and programmers alike
find they are unable to cope with displaying hundreds or thousands of
records. Refactoring takes time, and performance and user satisfaction
can suffer.

Displaying a reasonable number of records per page has always been a
critical part of every application and used to cause many headaches for
developers. CakePHP eases the burden on the developer by providing a
quick, easy way to paginate data.

The PaginatorHelper offers a great solution because it's so easy to use.
Apart from pagination, it bundles some very easy-to-use sorting
features. Last but not least, Ajax sorting and pagination are supported
as well.

Controller Setup
================

In the controller, we start by defining the pagination defaults in the
*$paginate* controller variable. It is important to note here that the
order key must be defined in the array structure given.

::

    class RecipesController extends AppController {

        var $paginate = array(
            'limit' => 25,
            'order' => array(
                'Post.title' => 'asc'
            )
        );
    }

You can also include other find() options, such as *fields*:

::

    class RecipesController extends AppController {

        var $paginate = array(
            'fields' => array('Post.id', 'Post.created'),
            'limit' => 25,        
            'order' => array(
                'Post.title' => 'asc'
            )
        );
    }

Other keys that can be included in the *$paginate* array are similar to
the parameters of the *Model->find('all')* method, that is:
*conditions*, *fields*, *order*, *limit*, *page*, *contain*, *joins*,
and *recursive*. In fact, you can define more than one set of pagination
defaults in the controller, you just name the pieces of the array after
the model you wish to configure:

::

    class RecipesController extends AppController {

        var $paginate = array(
            'Recipe' => array (...),
            'Author' => array (...)
        );
    }

Example of syntax using Containable Behavior:

::

    class RecipesController extends AppController {

        var $paginate = array(
            'limit' => 25,
            'contain' => array('Article')
        );
    }

Once the *$paginate* variable has been defined, we can call the
*paginate()* method in controller actions. This method returns paged
*find()* results from the model, and grabs some additional paging
statistics, which are passed to the View behind the scenes. This method
also adds PaginatorHelper to the list of helpers in your controller, if
it has not been added already.

::

    function list_recipes() {
        // similar to findAll(), but fetches paged results
        $data = $this->paginate('Recipe');
        $this->set('data', $data);
    }

You can filter the records by passing conditions as second parameter to
the ``paginate()`` function.

::

    $data = $this->paginate('Recipe', array('Recipe.title LIKE' => 'a%'));

Or you can also set *conditions* and other keys in the ``$paginate``
array inside your action.

::

    function list_recipes() {
        $this->paginate = array(
            'conditions' => array('Recipe.title LIKE' => 'a%'),
            'limit' => 10
        );
        $data = $this->paginate('Recipe');
        $this->set(compact('data'));
    );

Pagination in Views
===================

It's up to you to decide how to show records to the user, but most often
this will be done inside HTML tables. The examples below assume a
tabular layout, but the PaginatorHelper available in views doesn't
always need to be restricted as such.

See the details on
`PaginatorHelper <https://api.cakephp.org/class/paginator-helper>`_ in
the API.

As mentioned, the PaginatorHelper also offers sorting features which can
be easily integrated into your table column headers:

::

    // app/views/recipes/list_recipes.ctp
    <table>
        <tr> 
            <th><?php echo $this->Paginator->sort('ID', 'id'); ?></th> 
            <th><?php echo $this->Paginator->sort('Title', 'title'); ?></th> 
        </tr> 
           <?php foreach($data as $recipe): ?> 
        <tr> 
            <td><?php echo $recipe['Recipe']['id']; ?> </td> 
            <td><?php echo $recipe['Recipe']['title']; ?> </td> 
        </tr> 
        <?php endforeach; ?> 
    </table> 

The links output from the sort() method of the PaginatorHelper allow
users to click on table headers to toggle the sorting of the data by a
given field.

It is also possible to sort a column based on associations:

::

    <table>
        <tr> 
            <th><?php echo $this->Paginator->sort('Title', 'title'); ?></th> 
            <th><?php echo $this->Paginator->sort('Author', 'Author.name'); ?></th> 
        </tr> 
           <?php foreach($data as $recipe): ?> 
        <tr> 
            <td><?php echo $recipe['Recipe']['title']; ?> </td> 
            <td><?php echo $recipe['Author']['name']; ?> </td> 
        </tr> 
        <?php endforeach; ?> 
    </table> 

The final ingredient to pagination display in views is the addition of
page navigation, also supplied by the PaginationHelper.

::

    <!-- Shows the page numbers -->
    <?php echo $this->Paginator->numbers(); ?>
    <!-- Shows the next and previous links -->
    <?php echo $this->Paginator->prev('« Previous', null, null, array('class' => 'disabled')); ?>
    <?php echo $this->Paginator->next('Next »', null, null, array('class' => 'disabled')); ?> 
    <!-- prints X of Y, where X is current page and Y is number of pages -->
    <?php echo $this->Paginator->counter(); ?>

The wording output by the counter() method can also be customized using
special markers:

::

    <?php
    echo $this->Paginator->counter(array(
        'format' => 'Page %page% of %pages%, showing %current% records out of
                 %count% total, starting on record %start%, ending on %end%'
    )); 
    ?>

To pass all URL arguments to paginator functions, add the following to
your view:

::

    $this->Paginator->options(array('url' => $this->passedArgs));

Route elements that are not named arguments should manually be merged
with ``$this->passedArgs``:

::

    //for urls like http://www.example.com/en/controller/action
    //that are routed as Router::connect('/:lang/:controller/:action/*', array(), array('lang' => 'ta|en'));
    $this->Paginator->options(array('url' => array_merge(array('lang' => $lang), $this->passedArgs)));

Or you can specify which params to pass manually:

::

    $this->Paginator->options(array('url' => array("0", "1")));

AJAX Pagination
===============

It's very easy to incorporate Ajax functionality into pagination. Using
the JsHelper and RequestHandlerComponent you can easily add Ajax
pagination to your application. :doc:`/The-Manual/Core-Helpers/Js`

Configuring the PaginatorHelper to use a custom helper
------------------------------------------------------

By default in 1.3 the ``PaginatorHelper`` uses JsHelper to do ajax
features. However, if you don't want that and want to use the
``AjaxHelper`` or a custom helper for ajax links, you can do so by
changing the ``$helpers`` array in your controller. After running
``paginate()`` do the following.

::

    $this->set('posts', $this->paginate());
    $this->helpers['Paginator'] = array('ajax' => 'Ajax');

Will change the ``PaginatorHelper`` to use the ``AjaxHelper`` for ajax
operations. You could also set the 'ajax' key to be any helper, as long
as that class implements a ``link()`` method that behaves like
``HtmlHelper::link()``

Custom Query Pagination
=======================

Fix me: Please add an example where overriding paginate is justified

A good example of when you would need this is if the underlying DB does
not support the SQL LIMIT syntax. This is true of IBM's DB2. You can
still use the CakePHP pagination by adding the custom query to the
model.

Should you need to create custom queries to generate the data you want
to paginate, you can override the ``paginate()`` and ``paginateCount()``
model methods used by the pagination controller logic.

Before continuing check you can't achieve your goal with the core model
methods.

The ``paginate()`` method uses the same parameters as ``Model::find()``.
To use your own method/logic override it in the model you wish to get
the data from.

::

    /**
     * Overridden paginate method - group by week, away_team_id and home_team_id
     */
    function paginate($conditions, $fields, $order, $limit, $page = 1, $recursive = null, $extra = array()) {
        $recursive = -1;
        $group = $fields = array('week', 'away_team_id', 'home_team_id');
         return $this->find('all', compact('conditions', 'fields', 'order', 'limit', 'page', 'recursive', 'group'));
    }

You also need to override the core ``paginateCount()``, this method
expects the same arguments as ``Model::find('count')``. The example
below uses some Postgres-specifc features, so please adjust accordingly
depending on what database you are using.

::

    /**
     * Overridden paginateCount method
     */
    function paginateCount($conditions = null, $recursive = 0, $extra = array()) {
        $sql = "SELECT DISTINCT ON(week, home_team_id, away_team_id) week, home_team_id, away_team_id FROM games";
        $this->recursive = $recursive;
        $results = $this->query($sql);
        return count($results);
    }

The observant reader will have noticed that the paginate method we've
defined wasn't actually necessary - All you have to do is add the
keyword in controller's ``$paginate`` class variable.

::

    /**
    * Add GROUP BY clause
    */
    var $paginate = array(
        'MyModel' => array('limit' => 20, 
                               'order' => array('week' => 'desc'),
                               'group' => array('week', 'home_team_id', 'away_team_id'))
                              );
    /**
    * Or on-the-fly from within the action
    */
    function index() {
        $this->paginate = array(
        'MyModel' => array('limit' => 20, 
                               'order' => array('week' => 'desc'),
                               'group' => array('week', 'home_team_id', 'away_team_id'))
                              );

However, it will still be necessary to override the ``paginateCount()``
method to get an accurate value.

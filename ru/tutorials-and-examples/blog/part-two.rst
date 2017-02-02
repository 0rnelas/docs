Пример создания блога - Часть 2
###############################

Создание модели статей
======================

Модели - это всё в CakePHP. Создавая модель для работы с базой данных, мы
создаем фундамент для будущих операций просмотра, редактирования, удаления
данных.

Файлы классов модели CakePHP подразделяются на объекты ``Таблица`` и
``Сущность`` (Entity).
Объекты ``Таблица`` предоставляют доступ к коллекции сущностей размещенных в
определенной таблице. Они находятся в папке **src/Model/Table**. Класс,
который мы создадим, будет храниться в файле **src/Model/Table/ArticlesTable.php**.
Окончательный его вариант будет иметь следующий вид::

    // src/Model/Table/ArticlesTable.php

    namespace App\Model\Table;

    use Cake\ORM\Table;

    class ArticlesTable extends Table
    {
        public function initialize(array $config)
        {
            $this->addBehavior('Timestamp');
        }
    }
    
Соглашения об именовании очень важны в CakePHP. В случае, если мы называем наш
объект таблицы ``ArticlesTable``, CakePHP может автоматически предположить, что
этот объект будет использоваться контроллером ``ArticlesController``, и будет
связан с таблицей базы данных ``articles``.

.. note::

    CakePHP динамически создаст объект модели для Вас если не сможет
    найти нужный файл в  **src/Model/Table**. Это также означает, что
    если Вы случайно неправильно назовете файл (например articlestable.php или
    ArticleTable.php), CakePHP не распознает их и будет использовать вместо
    этого сгенерированную модель.
    
Для более подробной информации о моделях (например валидация данных) ознакомьтесь
с главой руководства :doc:`/orm`.

.. note::

    Если Вы завершили работу с :doc:`Частью 1 данного руководства
    </tutorials-and-examples/blog/blog>` и создали таблицу ``articles`` в
    базе данных нашего блога для большего удобства Вы можете воспользоваться
    консолью bake CakePHP и ее возможностями генерации кода для создания
    модели ``ArticlesTable``::
    
        bin/cake bake model Articles
        
Чтобы получить больше информации о консоли bake, пожалуйста ознакомьтесь с
:doc:`/bake/usage`.

Создание контроллера статей
===========================

Теперь мы создадим контроллер для наших статей. Контроллер - это то место, где
происходит все взаимодействие со статьями. В двух словах - это место, где Вы
взаимодействуете с бизнес-логикой, содержащейся в моделях и получаете готовый
результат после обработки статей. Мы разместим этот новый контроллер в файле
**ArticlesController.php** внутри папки **src/Controller**. Вот какой должна
быть начальная структура контроллера::

    // src/Controller/ArticlesController.php

    namespace App\Controller;

    class ArticlesController extends AppController
    {
    }
    
Давайте добавим экшен в наш контроллер. Экшены часто представляют из себя
отдельные функции или интерфейсы в приложении. Например, когда пользователь
запрашивает адрес www.example.com/articles/index (то же самое, что и 
www.example.com/articles/), он ожидает увидеть список статей. Код этого
экшена должен выглядеть так::

    // src/Controller/ArticlesController.php

    namespace App\Controller;

    class ArticlesController extends AppController
    {

        public function index()
        {
            // запрашиваем из модели данные о всех статьях
            $articles = $this->Articles->find('all');
            // передаем полученные данные в вид
            $this->set(compact('articles'));
        }
    }
    
Объявляя метод ``index()`` в нашем контроллере ``ArticlesController``,
мы даем возможность пользователям получить доступ к логике для
обработки запроса www.example.com/articles/index из адресной строки.
Схожим образом, если бы мы объявили метод ``foobar()``, пользователям
стал бы доступен адрес www.example.com/articles/foobar.

.. warning::

    Вы можете поддаться искушению называть Ваши контроллеры и экшены
    определенным образом, для получения определеных URL. Боритесь с
    этим искушением. Следуйте :doc:`/intro/conventions` (написание с
    заглавной буквы, форма множественного числа и т.д.) и создавайте
    читаемые, понятные имена экшенов. Вы можете назначить правила
    маршрутизации для Ваших адресов используя :doc:`/development/routing`
    рассматриваемый позже.
    
Единственная инструкция в экшене - ``set()`` передает данные от
контроллера в вид (который мы создадим позже). Эта строка назначает
переменную вида, называемую 'articles' совпадающую по названию с
результатом, возвращаемым методом ``find('all')`` объекта ``ArticlesTable``.

.. note::

    Если Вы завершили работу с :doc:`Частью 1 данного руководства
    </tutorials-and-examples/blog/blog>` и создали таблицу ``articles`` в
    базе данных нашего блога для большего удобства Вы можете воспользоваться
    консолью bake CakePHP и ее возможностями генерации кода для создания
    класса контроллера ArticlesController::

        bin/cake bake controller Articles

Чтобы получить больше информации о консоли bake, пожалуйста ознакомьтесь с
:doc:`/bake/usage`.

Чтобы узнать больше о контроллерах CakePHP ознакомьтесь с главой
документации :doc:`/controllers`.

Создание вида статей
====================

Теперь, когда унас есть наши данные, получаемые из модели и наша логика,
определенная в контроллере, давайте создадим вид для нашего экшена index,
созданного ранее.

.. note::
    The documentation is not currently supported in Russian language for this
    page.

    Please feel free to send us a pull request on
    `Github <https://github.com/cakephp/docs>`_ or use the **Improve This Doc**
    button to directly propose your changes.

    You can refer to the english version in the select top menu to have
    information about this page's topic.

Рекомендуется к прочтению
-------------------------

These are common tasks people learning CakePHP usually want to study next:

1. :ref:`view-layouts`: Customizing your website layout
2. :ref:`view-elements`: Including and reusing view snippets
3. :doc:`/bake/usage`: Generating basic CRUD code
4. :doc:`/tutorials-and-examples/blog-auth-example/auth`: User authentication
   and authorization tutorial


.. meta::
    :title lang=ru: Blog Tutorial Adding a Layer
    :keywords lang=ru: doc models,validation check,controller actions,model post,php class,model class,model object,business logic,database table,naming convention,bread and butter,callbacks,prefixes,nutshell,interaction,array,cakephp,interface,applications,delete

Salvando dados
##############

.. php:namespace:: Cake\ORM

.. php:class:: Table
    :noindex:

Depois que você :doc:`carregou seus dados</orm/retrieving-data-and-resultsets>`
provavelmente vai querer atualizar e salvar as alterações.

Visão geral sobre Salvando Dados
=========================

Aplicações geralmente terá algumas maneiras de como os dados são salvos. A
primeira é, obviamente, atravéz de formulários web e a outra é por geração direta
ou alterando dados no código para enviar ao banco de dados.

Inserindo dados
--------------

A maneira mais fácil de inserir dados no banco de dados é criando uma nova entidade
e passando ela pro método ``save()`` na class ``Table`` class::

    use Cake\ORM\TableRegistry;

    $articlesTable = TableRegistry::get('Articles');
    $article = $articlesTable->newEntity();

    $article->title = 'A New Article';
    $article->body = 'This is the body of the article';

    if ($articlesTable->save($article)) {
        // The $article entity contains the id now
        $id = $article->id;
    }
Atualizando dados
-------------

Atualizar seus dados é igualmente fácil, e o método ``save()`` também é usado para
esse propósito::

    use Cake\ORM\TableRegistry;

    $articlesTable = TableRegistry::get('Articles');
    $article = $articlesTable->get(12); // Return article with id 12

    $article->title = 'CakePHP is THE best PHP framework!';
    $articlesTable->save($article);

CakePHP saberá quando deve realizar uma inserção ou atualização com base no valor
de retorno do método ``isNew()``. Entidades que foram obtidas com ``get()`` ou
``find()`` sempre retornará ``false`` quando ``isNew()`` é chamado nelas.

Salvando com associações
------------------------

Por padrão o método ``save()`` também salvará associações de um nível::

    $articlesTable = TableRegistry::get('Articles');
    $author = $articlesTable->Authors->findByUserName('mark')->first();

    $article = $articlesTable->newEntity();
    $article->title = 'An article by mark';
    $article->author = $author;

    if ($articlesTable->save($article)) {
        // A chave estrangeira foi atribuída automaticamente
        echo $article->author_id;
    }

O método ``save()`` também é capaz de criar novos registros para associações::

    $firstComment = $articlesTable->Comments->newEntity();
    $firstComment->body = 'The CakePHP features are outstanding';

    $secondComment = $articlesTable->Comments->newEntity();
    $secondComment->body = 'CakePHP performance is terrific!';

    $tag1 = $articlesTable->Tags->findByName('cakephp')->first();
    $tag2 = $articlesTable->Tags->newEntity();
    $tag2->name = 'awesome';

    $article = $articlesTable->get(12);
    $article->comments = [$firstComment, $secondComment];
    $article->tags = [$tag1, $tag2];

    $articlesTable->save($article);

Associe Muitos para Muitos (N para N) registros
------------------------------

O exemplo anterior demonstra como associar algumas tags a um artigo.
Outra maneira de realizar a mesma coisa é usando o método ``link()``
na associação::

    $tag1 = $articlesTable->Tags->findByName('cakephp')->first();
    $tag2 = $articlesTable->Tags->newEntity();
    $tag2->name = 'awesome';

    $articlesTable->Tags->link($article, [$tag1, $tag2]);
 
Salvando dados da tabela de ligação
-----------------------------
Salvar dados na tabela de ligação é realizado usando a propriedade especial
``_joinData``. Esta propriedade deve ser um instância de ``Entity`` da classe
Table de ligação::

    // Link records for the first time.
    $tag1 = $articlesTable->Tags->findByName('cakephp')->first();
    $tag1->_joinData = $articlesTable->ArticlesTags->newEntity();
    $tag1->_joinData->tagComment = 'The CakePHP ORM is so powerful!';

    $articlesTable->Tags->link($article, [$tag1]);

    // Update an existing association.
    $article = $articlesTable->get(1, ['contain' => ['Tags']]);
    $article->tags[0]->_joinData->tagComment = 'Fresh comment.'

    // Necessary because we are changing a property directly
    $article->dirty('tags', true);

    $articlesTable->save($article, ['associated' => ['Tags']]);

Você também pode criar / atualizar informações na tabela de ligação utilizando
``newEntity()`` ou ``patchEntity()``. Os seus dados de POST devem parecer::

    $data = [
        'title' => 'My great blog post',
        'body' => 'Some content that goes on for a bit.',
        'tags' => [
            [
                'id' => 10,
                '_joinData' => [
                    'tagComment' => 'Great article!',
                ]
            ],
        ]
    ];
    $articlesTable->newEntity($data, ['associated' => ['Tags']]);

Remover associação Muitos para Muitos (N para N) registros
---------------------------

A remoção de associação Muitos para Muitos registros é realizada através do método
``unlink()``::

    $tags = $articlesTable
        ->Tags
        ->find()
        ->where(['name IN' => ['cakephp', 'awesome']])
        ->toArray();

    $articlesTable->Tags->unlink($article, $tags);

Quando modificando registros, configurando ou alterando diretamente as propriedades,
nenhuma validação é realizada, que é um problema quando está aceitando dados de 
formulário. As seções seguintes demostrarão como converter eficientemente dados de 
formulário em entidades que podem ser validadas e salva.

.. _converting-request-data:

Convertendo dados de requisição em entidades
=====================================

Antes de editar e salvar os dados de volta no seu banco de dados, você precisará
converter os dados da requisição, de array mantido na requisição em entidades
que o ORM utiliza. A classe Table fornece uma maneira fácil e eficiente de converter
uma ou várias entidades dos dados de requisição. Você pode converter uma entidade
usando::

    //No controller
    $articles = TableRegistry::get('Articles');

    // Valida e converte em um objeto do tipo Entity
    $entity = $articles->newEntity($this->request->getData());

.. note::

    Se você estiver usando newEntity() e as entidades resultantes estão faltando algum
    ou todos os dados passados, verifique se as colunas que deseja definir estão
    listadas na propriedade ``$_accessible`` da sua entidade. Consulte :ref:`entities-mass-assignment`.
 
Os dados da requisição devem seguir a estrutura de suas entidades. Por exemplo, se você
tem um artigo, que pertence a um usuário, e tem muitos comentários, os seus dados de
requisição devem ser semelhante::

    $data = [
        'title' => 'CakePHP For the Win',
        'body' => 'Baking with CakePHP makes web development fun!',
        'user_id' => 1,
        'user' => [
            'username' => 'mark'
        ],
        'comments' => [
            ['body' => 'The CakePHP features are outstanding'],
            ['body' => 'CakePHP performance is terrific!'],
        ]
    ];

Por padrão, o método ``newEntity()`` valida os dados que são passados para
ele, conforme explicado na seção :ref:`validating-request-data`. Se você 
deseja pular a validação de dados, informe a opção ``'validate' => false``::

    $entity = $articles->newEntity($data, ['validate' => false]);

Ao criar formulários que salvam associações aninhadas, você precisa definir
quais associaçes devem ser convertidas:: 

    // No controller
    $articles = TableRegistry::get('Articles');

    // Nova entidade com associações aninhadas
    $entity = $articles->newEntity($this->request->getData(), [
        'associated' => [
            'Tags', 'Comments' => ['associated' => ['Users']]
        ]
    ]);
 
O exemplo acima indica que 'Tags', 'Comments' e 'Users' para os artigos devem
ser convertidos. Alternativamente, você pode usar a notação de ponto 
(dot notation) por brevidade::

    // No controller
    $articles = TableRegistry::get('Articles');

    // Nova entidade com associações aninhada usando notação de ponto
    $entity = $articles->newEntity($this->request->getData(), [
        'associated' => ['Tags', 'Comments.Users']
    ]);
Você  também pode desativar a conversão de possíveis associações aninhadas como::

    $entity = $articles->newEntity($data, ['associated' => []]);
    // ou...
    $entity = $articles->patchEntity($entity, $data, ['associated' => []]);

Os dados associados também são validados por padrão, a menos que seja informado o
contrário. Você também pode alterar o conjunto de validação a ser usada por associação::

    // No controller
    $articles = TableRegistry::get('Articles');

    // Pular validação na associação de Tags e
    // Definino 'signup' como método de validação para Comments.Users
    $entity = $articles->newEntity($this->request->getData(), [
        'associated' => [
            'Tags' => ['validate' => false],
            'Comments.Users' => ['validate' => 'signup']
        ]
    ]);

O capitulo :ref:`using-different-validators-per-association` possui mais informações
sobre como usar diferentes validadores para associações ao transformar em entidades.

O diagrama a seguir fornece uma visão geral do que acontece dentro dos métodos 
``newEntity()`` ou ``patchEntity()``:

.. figure:: /_static/img/validation-cycle.png
   :align: left
   :alt: Flow diagram showing the marshalling/validation process.

Você sempre pode contar de obter uma entidade de volta com ``newEntity()``. Se a validação
falhar, sua entidade conterá erros, e quaisquer campos inválidos não serão preenchidos 
na entidade criada.

Convertendo dados de associação BelongsToMany
-----------------------------

Se você está salvando associações belongsToMany, você pode tanto usar uma lista de entidades
ou uma lista de ids. Ao usar uma lista de dados de entidade, seus dados de requisição
devem parecer com::

    $data = [
        'title' => 'My title',
        'body' => 'The text',
        'user_id' => 1,
        'tags' => [
            ['tag' => 'CakePHP'],
            ['tag' => 'Internet'],
        ]
    ];

O exemplo acima criará 2 novas tags. Se você deseja associar um artigo com tags existentes,
você pode usar uma lista de ids. Seus dados de requisição devem parecer com::

    $data = [
        'title' => 'My title',
        'body' => 'The text',
        'user_id' => 1,
        'tags' => [
            '_ids' => [1, 2, 3, 4]
        ]
    ];

Se você precisa associar a alguns belongsToMany registros existentes, e criar novos ao
mesmo tempo, você pode usar um formato expandido::

    $data = [
        'title' => 'My title',
        'body' => 'The text',
        'user_id' => 1,
        'tags' => [
            ['name' => 'A new tag'],
            ['name' => 'Another new tag'],
            ['id' => 5],
            ['id' => 21]
        ]
    ];

Quando os dados acima são convertidos em entidades, você terá 4 tags. As duas primeiras
serão objetos novos, e as outras duas serão referências a registros existentes.

Ao converter dados belongsToMany, você pode desativar a criação de nova entidade, usando
a opção ``onlyIds``. Quando habilitado, esta opção restringe transformação de
belongsToMany para apenas usar a chave ``_ids`` e ignorar todos os outros dados.

.. versionadded:: 3.1.0
    A opção ``onlyIds`` foi adicionada na versão 3.1.0

Convertendo dados de associação HasMany
-----------------------

Se você deseja atualizar as associações hasMany existentes e atualizar suas
propriedades, primeiro você deve garantir que sua entidade seja carregada com a
associação hasMany. Você pode então usar dados de requisição semelhantes a::

    $data = [
        'title' => 'My Title',
        'body' => 'The text',
        'comments' => [
            ['id' => 1, 'comment' => 'Update the first comment'],
            ['id' => 2, 'comment' => 'Update the second comment'],
            ['comment' => 'Create a new comment'],
        ]
    ];

Se você está salvando associaçoes hasMany e deseja vincular a registros existentes,
você pode usar o formato ``_ids``::

    $data = [
        'title' => 'My new article',
        'body' => 'The text',
        'user_id' => 1,
        'comments' => [
            '_ids' => [1, 2, 3, 4]
        ]
    ];

Ao converter dados hasMany, você pode desativar a criação de nova entidade, usando
a opção ``onlyIds`. Quando ativada, esta opção restringe transformação de hasMany
para apenas usar a chave ``_ids`` e ignorar todos os outros dados.

.. versionadded:: 3.1.0
     A opção ``onlyIds`` foi adicionada na versão 3.1.0
     
Conventendo vários registros
---------------------------

Ao criar formulários que cria/atualiza vários registros ao mesmo tempo, você pode usar
o método ``newEntities()``::

    // No controller.
    $articles = TableRegistry::get('Articles');
    $entities = $articles->newEntities($this->request->getData());

Nessa situação, os dados de requisição para vários artigos devem parecer com::

    $data = [
        [
            'title' => 'First post',
            'published' => 1
        ],
        [
            'title' => 'Second post',
            'published' => 1
        ],
    ];

Uma vez que você converteu os dados de requisição em entidades, você pode
salvar com ``save()`` e remover com ``delete()`` elas::

    // No controller.
    foreach ($entities as $entity) {
        // Salva a entidade
        $articles->save($entity);

        // Remover a entidade
        $articles->delete($entity);
    }

O exemplo acima executará uma transação separada para cada entidade salva. 
Se você deseja processar todas as entidades como uma única transação, você 
pode usar ``transactional()``::

    // No controller.
    $articles->getConnection()->transactional(function () use ($articles, $entities) {
        foreach ($entities as $entity) {
            $articles->save($entity, ['atomic' => false]);
        }
    });


.. _changing-accessible-fields:

Alterando campos acessíveis
--------------------------

Também é possível permitir ``newEntity()`` escreva em campos não acessiveis.
Por exemplo, ``id`` geralmente está ausente da propriedade ``_accessible``.
Nesse caso , você pode usar a opção `accessibleFields``. Isso pode ser útil para
manter ids de entidades associadas::

    // No controller
    $articles = TableRegistry::get('Articles');
    $entity = $articles->newEntity($this->request->getData(), [
        'associated' => [
            'Tags', 'Comments' => [
                'associated' => [
                    'Users' => [
                        'accessibleFields' => ['id' => true]
                    ]
                ]
            ]
        ]
    ]);

O exemplo acima manterá a associação inalterada entre Comments e Users para a
entidade envolvida.

.. note::

    Se você estiver usando newEntity() e as entidades resultantes estão faltando algum
    ou todos os dados passados, verifique se as colunas que deseja definir estão
    listadas na propriedade ``$_accessible`` da sua entidade. Consulte :ref:`entities-mass-assignment`.
    
Mesclando dados de requisição em entidades
----------------------------------

Para atualizar as entidades, você pode escolher de aplicar dados de requisição diretamente
em uma entidade existente. Isto tem a vantagem que apenas os campos que realmente mudaram 
serão salvos, em oposição ao envio de todos os campos para o banco de dados pra ser persistido.
Você pode mesclar um array de dados bruto em uma entidade existente usando o método
``patchEntity()``::

    // No controller.
    $articles = TableRegistry::get('Articles');
    $article = $articles->get(1);
    $articles->patchEntity($article, $this->request->getData());
    $articles->save($article);
    
Validação e patchEntity
~~~~~~~~~~~~~~~~~~~~~~~~~~

Semelhante ao ``newEntity()``, o método ``patchEntity`` validará os dados
antes de ser copiado para entidade. O mecanismo é explicado na seção
:ref:`validating-request-data`. Se você deseja desativar a validação, informe a
o opção ``validate`` assim::

    // No controller.
    $articles = TableRegistry::get('Articles');
    $article = $articles->get(1);
    $articles->patchEntity($article, $data, ['validate' => false]);

Você também pode alterar a regra de validação utilizada pela entidade ou qualquer
uma das associações::

    $articles->patchEntity($article, $this->request->getData(), [
        'validate' => 'custom',
        'associated' => ['Tags', 'Comments.Users' => ['validate' => 'signup']]
    ]);

Patching HasMany and BelongsToMany
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Como explicado na seção anterior, os dados de requisição deve seguir a 
estrutura de sua entidade. O método `patchEntity()`` é igualmente capaz de
mesclar associações, por padrão, apenas o primeiro nível de associações são
mesclados, mas se você deseja controlar a lista de associações a serem mescladas
ou mesclar em níveis mais profundos, você pode usar o terceiro parâmetro do método::

    // No controller.
    $associated = ['Tags', 'Comments.Users'];
    $article = $articles->get(1, ['contain' => $associated]);
    $articles->patchEntity($article, $this->request->getData(), [
        'associated' => $associated
    ]);
    $articles->save($article);

As associações são mescladas ao combinar o campo da chave primária nas entidades de origem
com os campos correspondentes no array de dados. As associações irão construir novas
entidades se nenhuma entidade anterior for encontrada para a propriedade alvo da associação.

Por exemplo, forneça alguns dados de requisição como este::

    $data = [
        'title' => 'My title',
        'user' => [
            'username' => 'mark'
        ]
    ];

Tentando popular uma entidade sem uma entidade na propriedade user criará
uma nova entidade do tipo user::

    // In a controller.
    $entity = $articles->patchEntity(new Article, $data);
    echo $entity->user->username; // Echoes 'mark'

O mesmo pode ser dito sobre associaçes hasMany e belongsToMany, com uma
advertência importante:

.. note::
    
    Para as associações belongsToMany, garanta que a entidade relevante tenha
    uma propriedade acessível para a entidade associada.

Se um Produto pertence a várias (belongsToMany) Tag::

    // Na classe da entidade Product
    protected $_accessible = [
        // .. outras propriedades
       'tags' => true,
    ];

.. note::
    
    Para as associações hasMany e belongsToMany, se houvesse algumas entidades que
    que não pudessem ser correspondidas por chave primaria a um registro no array de dados,
    então esses registros serão descartados da entidade resultante.
    
    Lembre-se que usando ``patchEntity()`` ou ``patchEntities()`` não persiste os
    dados, isso apenas edita (ou cria) as entidades informadas. Para salvar a entidade você
    terá que chamar o método ``save()`` da model Table.

Por exemplo, considere o seguinte caso::

    $data = [
        'title' => 'My title',
        'body' => 'The text',
        'comments' => [
            ['body' => 'First comment', 'id' => 1],
            ['body' => 'Second comment', 'id' => 2],
        ]
    ];
    $entity = $articles->newEntity($data);
    $articles->save($entity);

    $newData = [
        'comments' => [
            ['body' => 'Changed comment', 'id' => 1],
            ['body' => 'A new comment'],
        ]
    ];
    $articles->patchEntity($entity, $newData);
    $articles->save($entity);

No final, se a entidade for convertida de volta para um array, você obterá o
seguinte resultado::

    [
        'title' => 'My title',
        'body' => 'The text',
        'comments' => [
            ['body' => 'Changed comment', 'id' => 1],
            ['body' => 'A new comment'],
        ]
    ];

Como você pode ver, o comentário com id 2 não está mais lá, já que ele não
pode ser correspondido a nada no array ``$newData``. Isso acontece porque CakePHP está
refletindo o novo estado descrito nos dados de requisição.

Algumas vantagens adicionais desta abordagem é que isto reduz o número de
operações a serem executadas ao persistir a entidade novamente.

Por favor, observer que isso não significa que o comentário com id 2 foi removido do
bando de dados, se você deseja remover os comentários para este artigo que não estão
presentes na entidade, você pode coletar as chaves primárias e executar uma exclusão
de lote para esses que não estão na lista::

    // Num controller.
    $comments = TableRegistry::get('Comments');
    $present = (new Collection($entity->comments))->extract('id')->filter()->toArray();
    $comments->deleteAll([
        'article_id' => $article->id,
        'id NOT IN' => $present
    ]);

Como você pode ver, isso também ajuda ao criar soluçes onde uma associação precisa de
ser implementada como um único conjunto.

Você também pode popular várias entidades ao mesmo tempo. As considerações feitas para 
popular (patch) associaçes hasMany e belongsToMany se aplicam para popular várias entidades:
As comparação são feitas pelo valor do campo da chave primária e as correspondências que
faltam no array das entidades originais serão removidas e não estarão presentes no resultado::


    // Num controller.
    $articles = TableRegistry::get('Articles');
    $list = $articles->find('popular')->toArray();
    $patched = $articles->patchEntities($list, $this->request->getData());
    foreach ($patched as $entity) {
        $articles->save($entity);
    }

Semelhante de usar ``patchEntity()``, você pode usar o terceiro argumento para
controlar as associações que serão mescladas em cada uma das entidades no array::

    // Num controller.
    $patched = $articles->patchEntities(
        $list,
        $this->request->getData(),
        ['associated' => ['Tags', 'Comments.Users']]
    );


.. _before-marshal:

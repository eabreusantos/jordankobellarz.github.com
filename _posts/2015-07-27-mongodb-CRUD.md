---
layout: post
title:  "#2 MongoDB - CRUD"
description: "..."
date:   2015-07-27 00:00:00
categories: ['mongodb']
---

Trabalhar com o MongoDB é uma tarefa muito mais simples para o desenvolvedor, já que não é necessário se preocupar em manter uma estrutura rígida nos documetos de uma coleção. Temos aqui uma flexibilidade muito grande, que só é possível graças à sua estrutura orientada a documentos.

As operações de criar, buscar, atualizar ou remover documentos ocorrem somente sobre um coleção ao mesmo tempo, pois o MongoDB não suporta JOINS.

A seguir, serão mostrados conceitos básicos de operações de CRUD (Create, Read, Update e Delete) sobre uma coleção no MongoDB.

# Criando documentos


## Comando Insert

A inserção de documentos em uma coleção é realizada pelo comando `insert`. Ele é similar ao comando `insert` de uma base de dados relacional, com a diferença de não precisarmos seguir um *schema* predefinido. Quando essa operação é realizada com sucesso, retorna um objeto com o número de documentos inseridos.

**EXEMPLO** inserção de um documento:

{% highlight javascript linenos=table %}
db.test.insert({
  nome: "MongoDB Shell"
});

// cria o documento
{
  "\_id" : ObjectId("55b771bdbb9a2f0b2bdda8f5"),
  "nome" : "MongoDB Shell"
}
{% endhighlight %}

Quando criamos um documento sem o campo `\_id`, o driver gera automaticamente um `ObjectId` e adiciona ele ao campo `\_id` do documento. O mesmo vale para a coleção: se ela ainda não existir, então uma nova coleção é criada para permitir a inserção do novo documento.

###Inserção de conjunto de documentos

Caso queiramos inserir vários documentos ao mesmo tempo, podemos passar um array de documentos ao comando insert.

**EXEMPLO** inserção de um conjunto de documentos:

{% highlight javascript linenos=table %}
db.test.insert([
  {
    nome: "MongoDB",
    tipo: "orientado a documentos"
  },
  {
    nome: "CouchDB",
    tipo: "orientado a documentos"
  },
  {
    nome: "Redis",
    tipo: "key-value"
  }
]);
{% endhighlight %}

Ao executar o comando acima, caso algum dos documentos não seja inserido, então os outros não serão mais processados.

###Inserções em massa

É possível inserir também quantidades imensas de documentos em paralelo através da API `Bulk()`. Nesse caso, se um documento falhar, os outros ainda continuarão a ser inseridos.

**EXEMPLO** inserção de documentos em massa:

{% highlight javascript linenos=table %}
var bulk = db.test.initializeUnorderedBulkOp();
bulk.insert(
  {
    nome: "CouchDB",
    tipo: "orientado a documentos"
  }
);
bulk.insert(
  {
    nome: "Redis",
    tipo: "key-value"
  }
);
bulk.execute();
{% endhighlight %}

###Velocidade de inserção

Uma inserção no MongoDB ocorre centenas de vezes mais rápido que em um banco relacional. Isso é possível, pois são feitas verificações mínimas nos documentos a serem inseridos:

* **estrutura básica:** verifica se o tamanho do documento não ultrapassa 16MB e se é uma estrutura JSON válida;
* **existência do \_id:** verifica se o driver gerou um \_id, caso contrário, o servidor cria um novo.

Os 16MB de tamanho máximo para um documento é um valor arbitrário que o MongoDB usa para evitar projetos ruins de bancos de dados. Nas próximas seções, será demonstrado como armazenar documentos maiores que 16MB através do GridFS.

###Alternativas para inserir documentos

Existem outros métodos de adicionar documentos, que funcionam quando tentamos atualizar os campos de um documento que ainda não existe no banco de dados, são eles:

{% highlight javascript linenos=table %}
db.test.update({...});
db.test.findAndModify({...});
db.test.save({...});
{% endhighlight %}

Estes métodos serão vistos em maior profundidade nas próximas seções.

# Buscando documentos
A riqueza de opções para o método `find` é tão grande, que só dele poderia sair um livro, caso explicado em conjunto com o framework de agregação. Esse comando sempre retorna um cursor para os documentos encontrados, possiblitando operações de ordenação, limites, etc.

Para retornar todos os documentos de uma coleção, usamos o método `find` sem passar nenhum parâmetro.

**EXEMPLO** retornando todos os documentos:

{% highlight javascript linenos=table %}
db.test.find();
db.test.find({}); // igual ao comando acima
db.test.findOne(); // retorna apenas um documento
{% endhighlight %}

No Mongo Shell, ao executar o método `find`, o cursor retornará apenas os 20 primeiros documentos encontrados. Para iterar sobre o cursor, nesse caso, usa-se o comando `it`.

##Comportamento do cursor
Cursores ficam ativos por 10 minutos no cliente ou até ele ser esgotado. O comportamento de *timeout* de 10 minutos, contudo, pode ser removido através do comando `addOption`, passando como parâmetro a variável global `DBQuery.Option.noTimeout`.

**EXEMPLO** removendo o comportamento de *timeout*:

{% highlight javascript linenos=table %}
db.test.find().addOption(DBQuery.Option.noTimeout);
{% endhighlight %}

###Isolamento do cursor

O mótodo `find` pode retornar o mesmo documento mais de uma vez, caso este seja modificado antes de terminar seu ciclo de vida por exaustão ou por *timeout*. Isso ocorre devido à natureza não isolada do cursor.

Para evitar que isso aconteça, podemos usar o método [snapshot](http://docs.mongodb.org/manual/faq/developers/#faq-developers-isolate-cursors). Dessa forma, apenas uma versão de cada documento será retornada no cursor.

**EXEMPLO** usando o método `snapshot`:

{% highlight javascript linenos=table %}
db.test.find().snapshot();
{% endhighlight %}

####Status dos Cursores

É possível verificar a situação dos cursores ativos e inativos no Mongo Server através do comando `serverStatus`.

**EXEMPLO** verificando a situação dos cursores no servidor:

{% highlight javascript linenos=table %}
db.serverStatus().metrics.cursor

// resultado
{
   "timedOut" : <number>,
   "open" : {
      "noTimeout" : <number>,
      "pinned" : <number>,
      "total" : <number>
   }
}
{% endhighlight %}

###Ordenando, limitando, omitindo e contando documentos através de cursores

Os cursores disponibilizam métodos para ordenar, limitar, omitir e contar documentos em uma query. Eles podem ser invocados em conjunto de forma sequencial.

**EXEMPLO** usando métodos de ordenação, limite e omissão de resultados

{% highlight javascript linenos=table %}
// ordena em ordem decrescente pelo "nome" e, depois,
// em ordem crescente pelo "tempo"
db.test.find().sort({nome:-1, tempo: 1});

// limita em 5 documentos, no máximo
db.test.find().limit(5);

// omite os 8 primeiros documentos encontrados
db.test.find().skip(8);

// conta os resultados
db.test.find().count();

// combinação dos métodos
db.test.find().sort({nome:-1, tempo: 1}).limit(5).skip(8);
{% endhighlight %}

Existe uma variedade muito grande de métodos usados sobre cursores, que serão explicados mais adiante. É possível verificar a [lista de métodos em cursores aqui](http://docs.mongodb.org/manual/reference/method/js-cursor/).

##Especificando os documentos a serem retornados

As queries começam a ficar mais interessantes ao passo que começamos a especificar os documentos que queremos que sejam retornados através de vários operadores básicos e modificadores, como será visto nos próximos tópicos:

###Procurando uma string

Para procurar por um valor nos campos, apenas o passamos como parâmetro da query.

**EXEMPLO** procurando por documentos que tenham o campo "nome" igual a "Mongo" e o campo "ano" igual a 2007:

{% highlight javascript linenos=table %}
db.test.find({"nome": "Mongo", "ano": 2007});
{% endhighlight %}

Tenha cuidado ao procurar por campos com valores nulos, pois nesse caso o servidor retornará tanto os documentos que tenham campos com valor nulo, quanto os que não tenham o campo procurado.

**EXEMPLO** procurando documentos que tenham o campo "nome" igual a null ou que não tenham o campo "nome":

{% highlight javascript linenos=table %}
db.test.find({"nome": null});

// pode retornar
{
  "nome": null
},

// e também pode retornar
{}
{% endhighlight %}

Quando queremos procurar apenas documento que contenham determinado campo, podemos usar o operador `$exists`.

**EXEMPLO** procurando documentos que tenham o campo "nome"

{% highlight javascript linenos=table %}
db.test.find({"nome": {$exists: true}});
{% endhighlight %}

###Projeção de campos
Os campos a serem retornados devem ser especificados no segundo parâmetro do método `find`.

**EXEMPLO** retornando apenas o campo "ano":

{% highlight javascript linenos=table %}
db.test.find({"nome": "Mongo"}, {"ano": 1});
{% endhighlight %}

Da mesma forma que especificamos os campos a serem retornados, podemos especificar apenas os campos a não serem retornados.

**EXEMPLO** removendo da projeção apenas o campo "status":

{% highlight javascript linenos=table %}
db.test.find({"nome": "Mongo"}, {"status": -1});
{% endhighlight %}

>> Na projeção ou usamos a remoção de campos ou a especificação dos campos a serem retornados, jamais os dois juntos, salvo no caso abaixo:


Como pode ser notado nos exemplos acima, o MongoDB sempre retorna o campo `\_id`, mesmo que não o tenhamos especificado na projeção, pois é um campo que na maioria das vezes é necessário para que possamos manipular o documento posteriormente. Podemos sobreescrever esse comportamento especificando a sua retirada da projeção.

**EXEMPLO** removendo o campo \_id da projeção:

{% highlight javascript linenos=table %}
db.test.find({"nome": "Mongo"}, {"ano": 1, "\_id": -1});
{% endhighlight %}

###Operadores
Assim como em bancos de dados relacionais, temos a possibilidade de acessar operadores lógicos (OR, AND) ou matemáticos (==, !=, <, <=, >, >=) para procurar documentos. Eles são usados com o prefixo `$` para distingui-los de campos em documentos.

####Operador $and (AND lógico)

Operações AND são realizadas usando uma vírgula entre as condições ou, alternativamente, através do modificador `$and`.

**EXEMPLO** retornando todos os documentos que tenham "qtd" igual a 100 e "preco" igual a 9.95:  

{% highlight javascript linenos=table %}
db.test.find({
  { qtd: 100 }, { preco: 9.95}
})

// alternativamente
db.test.find({
  $and: [ { qtd: 100 }, { preco: 9.95 } ]
})
{% endhighlight %}

####Operadores $or, $in e $nin  (OU lógico)
Condicionais OR são realizadas através do operador `$or`, que precisa de um array de condições a serem verificadas.

**EXEMPLO** retornando todos os documentos que tenham "qtd" igual a 100 ou "preco" igual a 9.95:  

{% highlight javascript linenos=table %}
db.test.find({
  $or: [ { qtd: 100 }, { preco: 9.95 } ]
})
{% endhighlight %}

Para o caso de queries mais simples com a cláusula OR, podemos usar o operador `$in` ou `$nin` (negação de `$in`) - eles são ligeiramente mais rápidos que o operador `$or`.

**EXEMPLO** retornando todos os documentos que tenham "qtd" igual a 20 ou "qtd" igual a 50:  

{% highlight javascript linenos=table %}
db.test.find({
  qtd: {$in: [20, 50]}
})
{% endhighlight %}

Tente sempre que possível usar o operador `$in`, pois o *query optimizer* usa ele de forma mais eficiente.

####Condicionais $gt, $gte, $lt, $lte (>, >=, <, <=)

Trabalhamos com inequações através dos operadores `$gt`, `$gte`, `$lt`, `$lte`, que tem a mesma função que >, >=, <, <=, respectivamente.

**EXEMPLO** retornando todos os documentos que tenham "qtd" maior que 100 e "preco" menor ou igual a 9.95:

{% highlight javascript linenos=table %}
db.test.find({
  { qtd: { $gt: 100 } }, { preco: { $lte: 9.95 } }
})
{% endhighlight %}

####Operador $ne (!=)

Quando precisamos que o documento tenha um campo com valor diferente do especificado, usamos o operador `$ne`, que se comporta como o operador !=.

**EXEMPLO** retornando todos os documentos que tenham "qtd" diferente de 100 e "preco" diferente de 9.95, podemos fazer a seguinte query:

{% highlight javascript linenos=table %}
db.test.find({
  $ne: [ { qtd: 100 }, { preco:  9.95 } ]
})
{% endhighlight %}

####Operador $regex

<!-- TODO: operador $type -->

<!-- TODO: operador $regex -->

###Documentos encadeados

Da mesma forma que procuramos por campos, podemos procurar dentro de subdocumentos através da notação de ponto (*dot notation*). Fique atento ao uso obrigatório de aspas em volta do nome da chave.

**EXEMPLO** procurando dentro de um campo encadeado:

{% highlight javascript linenos=table %}
db.test.find({"info.qtd": 50})

// exemplo de documento retornado
{
  "info": {
    "qtd": 50
  }
}
{% endhighlight %}

###Arrays

A sintaxe rica do MongoDB permite trabalhar com arrays através de alguns modificadores específicos, como será visto a seguir:

####Correspondência ampla

Quando queremos encontrar um documento que tenha um array com pelo menos o valor procurado ou não sabemos se o campo é realmente um array, podemos fazer uma busca de correspondência ampla.

**EXEMPLO** retornando documentos que contenham a string "banana" ou o valor "banana" em qualquer posição do array de "frutas":

{% highlight javascript linenos=table %}
db.test.find({
  "frutas": "banana"
})

// retorno
{
  "frutas": ["maçã", "mamão", "banana"]
},
{
  "frutas": ["banana"]
}
{% endhighlight %}

####Correspondência exata

Quando sabemos exatamente como é o array que o documento deve conter, devemos passá-lo como parâmetro da query.

>> A posição dos elementos no array é importante para retornar os documentos corretos. Nesse caso, uma busca por ["banana", "maçã"] e ["maçã", "banana"] retornarão documentos diferentes.

**EXEMPLO** retornando documentos que contenham exatamente o array ["banana", "maçã"]:

{% highlight javascript linenos=table %}
db.test.find({
  "frutas": ["banana", "maçã"]
})

// retorno
{
  "frutas": ["banana", "maçã"]
}
{% endhighlight %}

####Operador $all

Quando temos uma **lista de valores que o array deve conter no mínimo**, podemos passar eles com o modificador `$all`. A ordem não é importante para esse operador.

**EXEMPLO** retornando documentos que contenham pelo menos os valores `"banana" e "maçã"`:

{% highlight javascript linenos=table %}
db.test.find({
  "frutas": {$all: ["banana", "maçã"]}}
})

// retorno
{
  "frutas": ["maçã", "mamão", "banana"]
},
{
  "frutas": ["maçã", "banana"]
},
{
  "frutas": ["banana", "maçã"]
}
{% endhighlight %}

####Operador $in

É possível usarmos também o operador `$in` em arrays, a fim de encontrar documentos que possam ter **pelo menos um dos valores buscados**. A ordem não é importante para esse operador.

**EXEMPLO** retornando documentos que contenham pelo menos os valores `"banana" ou "maçã"`:

{% highlight javascript linenos=table %}
db.test.find({
  "frutas": {$in: ["banana", "maçã"]}}
})

// exemplo
{
  "frutas": ["maçã", "mamão"]
},
{
  "frutas": ["banana"]
},
{
  "frutas": ["banana", "maçã"]
}
{% endhighlight %}


#Modificando documentos

A modificação dos documentos é feita através do método `update` que recebe pelo menos 2 parâmetros: o primeiro especifica os documentos a serem modificados (exatamente igual ao método `find`) e o segundo parâmetro especifica o quê será modificado.

**EXEMPLO** modificando um documento:

{% highlight javascript linenos=table %}
// supondo que o seguinte documento existe na coleção 'test'
{
  \_id: 1,
  "nome": "Pedro",
  "idade": 45,
  "programa": ["Python", "Javascript", "PHP"]
}

// executando o update
db.test.update({
  "\_id": 0
}, {
  "nome": "João"
})
{% endhighlight %}

Note que o `update` acima fez com que o documento anterior fosse completamente trocado pelo novo documento passado como segundo parâmetro. O único campo que foi mantido, foi o \_id - esse é o comportamento padrão do método `update`.

Os próximos tópicos explicarão como modificar o valor de apenas um campo em um documento, sem que todo o documento seja substituído.

##Atomicidade do método update
Operações de modificação são sempre atômicas à nível de documento: caso mais de um cliente tente modificar o mesmo documento ao mesmo tempo, a fila de requisições é respeitada. A última requisição da fila sempre será a vencedora, dessa forma, sua ação é a que preponderará dentre as outras.

##Modificadores
Estão disponíveis uma grande variedade de modificadores para trabalhar com a atualização de campos no MongoDB. Assim como os operadores e modificadores usados com o método `find`, aqui eles também devem sempre iniciar com `$`.

###Operador $set
O método update sem nenhum modificador substitui completamente o documento antigo pelo novo (como visto anteriormente). Para evitar esse comportamento e permitir apenas a atualização de campos específicos, usamos o comando `$set`.

Caso seja executado `$set` sobre um documento que não contenha o campo que terá o valor modificado, então o campo é criado com o valor especificado.

**EXEMPLO** trocando o valor do campo "nome" de "Mongo" para "MongoDB":

{% highlight javascript linenos=table %}
db.test.update({"nome": "Mongo"}, {
  $set: {"nome": "MongoDB"}
})
{% endhighlight %}

###Operador $unset
Para remover um campo e seu conteúdo de um documento, usamos o comando `$unset`.

**EXEMPLO** removendo o campo "nome" de um documento:

{% highlight javascript linenos=table %}
db.test.update({"nome": "Mongo"}, {
  $unset: {"nome": 1}
});
{% endhighlight %}

###Operador $inc
Para incrementar um valor numérico, usamos o comando `$inc`.

**EXEMPLO** incrementando o valor do campo "likes" em 5 unidades:

{% highlight javascript linenos=table %}
db.test.update({"nome": "MongoDB"}, {
  $inc: {"likes": 5}
})
{% endhighlight %}

##Modificando arrays em documentos

###Comando $push
Para inserir um novo elemento no fim do array, usamos o comando `$push`, passando como parâmetro o elemento a ser adicionado.

**EXEMPLO** inserindo o elemento 5 no array:

{% highlight javascript linenos=table %}
db.test.update({"nome": null}, {
  $push: {"frutas": 5}
})
{% endhighlight %}

###Comando $pushAll
Para inserir vários novos elementos no fim de um array, usamos o comando `$pushAll`, passando como parâmetro a lista de elementos a serem adicionados.

**EXEMPLO** inserindo a lista ["mamão", 4, 6] no array:

{% highlight javascript linenos=table %}
db.test.update({"nome": null}, {
  $pushAll: {"frutas": ["mamão", 4, 6]}
})
{% endhighlight %}

###Comando $pop
Para remover um elemento do fim do array, usamos o comando `$pop`.

**EXEMPLO** removendo o último elemento do array:

{% highlight javascript linenos=table %}
db.test.update({"nome": null}, {
  $pop: {"frutas": 1}
})
{% endhighlight %}

###Comando $pull
Para remover um elemento em uma posição específica do array, usamos o comando `$pull`, passando como parâmetro a posição a ser removida.

**EXEMPLO** removendo o último elemento do array

{% highlight javascript linenos=table %}
db.test.update({"nome": null}, {
  $pop: {"frutas": 1}
})
{% endhighlight %}

###Comando $pullAll
Para remover vários elementos específicos de um array, usamos o comando `$pullAll`, passando como parâmetro uma lista de elementos a serem removidos.

**EXEMPLO** removendo a lista ["mamão", 4, 6] do array:

{% highlight javascript linenos=table %}
db.test.update({"nome": null}, {
  $pullAll: {"frutas": ["mamão", 4, 6]}
})
{% endhighlight %}

###Comando $addToSet
Para inserir um novo elemento no fim do array, sabendo que ele deve ser único dentro do array, usamos o comando `$addToSet`, passando como parâmetro o elemento a ser adicionado. Esse comando age como o `$push`, sempre que o elemento a ser inserido ainda não existir no array.

**EXEMPLO** adicionando um elemento que ainda não existe no array:

{% highlight javascript linenos=table %}
db.test.update({"nome": null}, {
  $addToSet: {"frutas": "abacaxi"}
})

// se eu tentar novamente, nada será inserido:
db.test.update({"nome": null}, {
  $addToSet: {"frutas": "abacaxi"}
})
{% endhighlight %}

###Comando $each
Para inserir uma lista elementos no fim do array, sabendo que cada valor da lista deve ser único dentro do array, usamos o comando `$each` combinado com o comando `$addToSet`.

**EXEMPLO** adicionando uma lista de elementos que ainda não existem no array:

{% highlight javascript linenos=table %}
db.test.update({"nome": null}, {
  $addToSet: {
    "frutas": {
      $each: ["banana", "morango", "pêra"]
    }
  }
})
{% endhighlight %}


###Comando $slice
Para permitir que o array cresça somente até um tamanho predefinido, usamos o comando `$slice`, passando como parâmetro o número máximo de posições que o array deve ter. Esse comando deve ser usado junto com a inserção de novos elementos.

Se o parâmetro que especifica o número máximo de posições que o array deve ter for positivo, então são mantidos os n primeiros documentos inseridos. Caso seja negativo, então não mantidos os n últimos elementos inseridos.

**EXEMPLO** mantendo os 4 primeiros elementos no array

{% highlight javascript linenos=table %}
// supondo o documento
{
  "\_id": 1,
  "numeros": [0, 1, 2]
}

// usando $slice com parâmetro positivo
db.test.update({"\_id": 1}, {
  $push: {
    scores: {
      $each: [3, 4, 5],
      $slice: 4
    }
  }
})

// tem como resultado o documento
{
  "\_id": 1,
  "numeros": [0, 1, 2, 4]
}
{% endhighlight %}

**EXEMPLO** mantendo os 4 últimos elementos no array

{% highlight javascript linenos=table %}
// supondo o documento
{
  "\_id": 1,
  "numeros": [0, 1, 2]
}

// usando $slice com parâmetro positivo
db.test.update({"\_id": 1}, {
  $push: {
    scores: {
      $each: [3, 4, 5],
      $slice: -4
    }
  }
})

// tem como resultado o documento
{
  "\_id": 1,
  "numeros": [2, 3, 4, 5]
}
{% endhighlight %}

###Comando $sort
Para ordenar os itens do array, usamos o comando `$sort`, passando como parâmetro se queremos em ordem crescente [1] ou decrescente [-1]. Ele é usado em conjunto com a operação de `$push` e do comando `$each`. Caso queiramos apenas ordenar os elementos do array, passamos um array vazio `[]` como parâmetro do método `$each`.

**EXEMPLO** ordenando os elementos do array:

{% highlight javascript linenos=table %}
db.test.update({"nome": null}, {
  $push: {
    frutas: {
      $each: [40, 60],
      $sort: 1
    }
  }
})
{% endhighlight %}

##Upsert
O que acontece se tentarmos mopdificar um documento que não existe na coleção? Nada. A não ser que flag **upsert** seja setada como `true`.

TODO: $SEToNiNSERT

TODO: SAVE SHELL HELPER

TODO: update em vários documentos

TODO: findAndModify

TODO: WRITE CONCERN

#Removendo documentos

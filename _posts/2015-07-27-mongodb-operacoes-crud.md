---
layout: post
title:  "#2 MongoDB - Operações CRUD"
description: "Conhecer a mala de ferramentas do MongoDB é uma tarefa muito fácil. Vamos aprender as operações básicas de inserção, leitura, modificação e remoção de documentos, com eficiência!"
date:   2015-07-27 00:00:00
categories: ['mongodb']
---

<img src="/assets/images/mongo-2.png" alt="Operações CRUD" style="width: 60%; margin:0 auto">

Trabalhar com o MongoDB é uma tarefa muito mais simples para o desenvolvedor, já que não é necessário se preocupar em manter uma estrutura rígida nos documetos de uma coleção. Temos aqui uma flexibilidade muito grande, que só é possível graças a sua estrutura orientada a documentos.

As operações de criar, buscar, modificar ou remover documentos ocorrem somente sobre uma coleção ao mesmo tempo. Essa é uma característica comum aos bancos orientados a documentos, principalmente por terem eliminado o conceito de JOINS para permitir alta escalabilidade.

A seguir, serão mostrados conceitos básicos de operações de CRUD (Create, Read, Update e Delete) sobre uma coleção no MongoDB.

# Criando documentos


## Método Insert

A inserção de documentos em uma coleção é realizada pelo método `insert`. Ele é similar ao comando `insert` de uma base de dados relacional, com a diferença de não precisarmos seguir um *schema* predefinido. Quando essa operação é realizada com sucesso, retorna um objeto com o número de documentos inseridos.

**EXEMPLO** inserção de um documento:

{% highlight javascript linenos=table %}
db.test.insert({
  nome: "MongoDB Shell"
});

// cria o documento
{
  "_id" : ObjectId("55b771bdbb9a2f0b2bdda8f5"),
  "nome" : "MongoDB Shell"
}
{% endhighlight %}

Quando criamos um documento sem o campo `_id`, o driver gera automaticamente um `ObjectId` e adiciona ele ao campo `_id` do documento. O mesmo vale para a coleção: se ela ainda não existir, então uma nova coleção é criada para permitir a inserção do novo documento.

###Inserção de um conjunto de documentos

Caso queiramos inserir vários documentos ao mesmo tempo, podemos passar um array de documentos ao método `insert`.

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
* **existência do `_id`:** verifica se o driver gerou um `_id`, caso contrário, o servidor cria um novo e insere no documento.

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
A riqueza de opções para o método `find` é tão grande, que só dele poderia sair um livro, caso explicado em conjunto com o framework de agregação. Esse método sempre retorna um cursor para os documentos encontrados, possiblitando operações de ordenação, limites, etc.

Para retornar todos os documentos de uma coleção, usamos o método `find` sem passar nenhum parâmetro.

**EXEMPLO** retornando todos os documentos:

{% highlight javascript linenos=table %}
db.test.find();
db.test.find({}); // igual ao comando acima
db.test.findOne(); // retorna apenas um documento
{% endhighlight %}

No Mongo Shell, ao executar o método `find`, o cursor retornará apenas os 20 primeiros documentos encontrados. Para iterar sobre o cursor, nesse caso, usa-se o comando `it`.

##Comportamento do cursor
Cursores ficam ativos por 10 minutos no cliente ou até ele ser esgotado. O comportamento de *timeout* de 10 minutos, contudo, pode ser removido através do método `addOption`, passando como parâmetro a variável global `DBQuery.Option.noTimeout`.

**EXEMPLO** removendo o comportamento de *timeout*:

{% highlight javascript linenos=table %}
db.test.find().addOption(DBQuery.Option.noTimeout);
{% endhighlight %}

###Isolamento do cursor

O mótodo `find` pode retornar o mesmo documento mais de uma vez, caso este seja modificado antes de terminar seu ciclo de vida por exaustão ou por *timeout*. Isso ocorre devido a natureza não isolada do cursor.

Para evitar que isso aconteça, podemos usar o método [snapshot](http://docs.mongodb.org/manual/faq/developers/#faq-developers-isolate-cursors). Dessa forma, apenas uma versão de cada documento será retornada no cursor.

**EXEMPLO** usando o método `snapshot`:

{% highlight javascript linenos=table %}
db.test.find().snapshot();
{% endhighlight %}

####Status dos Cursores

É possível verificar a situação dos cursores ativos e inativos no Mongo Server através do método `serverStatus`.

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

**EXEMPLO** usando métodos de ordenação, limite e omissão de resultados:

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
db.test.find({nome: "Mongo", ano: 2007});
{% endhighlight %}

Tenha cuidado ao procurar por campos com valores nulos, pois nesse caso o servidor retornará tanto os documentos que tenham campos com valor nulo, quanto os que não tenham o campo procurado.

**EXEMPLO** procurando documentos que tenham o campo "nome" igual a `null` ou que não tenham o campo "nome":

{% highlight javascript linenos=table %}
db.test.find({});

// pode retornar
{
  "nome": null
},

// e também pode retornar
{}
{% endhighlight %}

Quando queremos procurar apenas documentos que contenham determinado campo, podemos usar o operador `$exists`.

**EXEMPLO** procurando documentos que tenham o campo "nome":

{% highlight javascript linenos=table %}
db.test.find({
  nome: {$exists: true}
});
{% endhighlight %}

###Projeção de campos
Podemos especificar os campos a serem retornados nos documentos através do segundo parâmetro do método `find`.

**EXEMPLO** retornando apenas o campo "ano":

{% highlight javascript linenos=table %}
db.test.find({nome: "Mongo"}, {ano: 1});
{% endhighlight %}

Da mesma forma que especificamos os campos a serem retornados, podemos especificar apenas os campos a não serem retornados.

**EXEMPLO** removendo da projeção apenas o campo "status":

{% highlight javascript linenos=table %}
db.test.find({nome: "Mongo"}, {ano: -1});
{% endhighlight %}

>> Na projeção ou removemos ou adicionamos os campos a serem retornados, jamais os dois juntos, salvo no caso abaixo:


Como pode ser notado nos exemplos acima, o MongoDB sempre retorna o campo `_id`, mesmo que não o tenhamos especificado na projeção, pois é um campo que na maioria das vezes é necessário para que possamos manipular o documento posteriormente. Podemos sobreescrever esse comportamento especificando a sua retirada da projeção.

**EXEMPLO** removendo o campo `_id` da projeção:

{% highlight javascript linenos=table %}
db.test.find({nome: "Mongo"}, {ano: 1, _id: -1});
{% endhighlight %}

###Operadores de query
Assim como em bancos de dados relacionais, temos a possibilidade de acessar operadores lógicos (OR, AND) ou de comparação (==, !=, <, <=, >, >=) para procurar documentos. Eles são usados com o prefixo `$` para distingui-los de campos em documentos.

####Operador $and (AND lógico)

Operações AND são realizadas usando uma vírgula entre as condições ou, alternativamente, através do operador `$and`.

**EXEMPLO** retornando todos os documentos que tenham "tipo" igual a "orientado a documentos" **e** "ano" igual a 2007:  

{% highlight javascript linenos=table %}
db.test.find({
  { tipo: "orientado a documentos" }, { ano: 2007}
})

// alternativamente
db.test.find({
  $and: [ { tipo: "orientado a documentos" }, { ano: 2007 } ]
})
{% endhighlight %}

####Operadores $or, $in e $nin  (OU lógico)
Condicionais OR são realizadas através do operador `$or`, que precisa de um array de condições a serem verificadas.

**EXEMPLO** retornando todos os documentos que tenham "tipo" igual a "orientado a documentos" **ou** "ano" igual a 2007:

{% highlight javascript linenos=table %}
db.test.find({
  $or: [ { tipo: "orientado a documentos" }, { ano: 2007 } ]
})
{% endhighlight %}

Para o caso de queries mais simples com a cláusula OR, podemos usar o operador `$in` ou `$nin` (negação de `$in`) - eles são ligeiramente mais rápidos que o operador `$or`.

**EXEMPLO** retornando todos os documentos que tenham "tipo" igual a "orientado a documentos" **ou** "tipo" igual "colunar":

{% highlight javascript linenos=table %}
db.test.find({
  tipo: {$in: ["orientado a documentos", "colunar"]}
})
{% endhighlight %}

Tente sempre que possível usar o operador `$in`, pois o *query optimizer* usa ele de forma mais eficiente.

####Comparadores $gt, $gte, $lt, $lte (>, >=, <, <=)

Trabalhamos com inequações através dos operadores `$gt`, `$gte`, `$lt`, `$lte`, que tem a mesma função que >, >=, <, <=, respectivamente.

**EXEMPLO** retornando todos os documentos que tenham "ano" entre 2005 e 2011, inclusive:

{% highlight javascript linenos=table %}
db.test.find({
  ano: { $gt: 2005, $lte: 2011 }
})
{% endhighlight %}

####Operador $eq e $ne (== e !=)

Trabalhamos com igualdades através dos operadores `$eq` e `$ne`, que se comportam como os operadores `==` e `!=`, respectivamente. O operador `$eq` está disponível somente a partir da versão 3.0 do MongoDB.

**EXEMPLO** retornando todos os documentos que tenham "ano" diferente de 2005 e "tipo" igual a "colunar":

{% highlight javascript linenos=table %}
db.test.find({
  ano: {$ne: 2005},
  tipo: {$eq: "colunar"}
})

// a query abaixo tem o mesmo resultado
db.test.find({
  ano: {$ne: 2005},
  tipo: "colunar"
})
{% endhighlight %}

###Verificando o tipo
Para verificar se um campo é de um tipo específico, usamos o operador `$type`, que recebe como parâmetro o valor numérico correspondente ao tipo BSON, que almejamos encontrar. Veja [http://docs.mongodb.org/v2.2/reference/glossary/#term-bson](a tabela de tipos BSON e seus respectivos valores numéricos).

**EXEMPLO** procurando por campos "tipo", que contenham uma String:

{% highlight javascript linenos=table %}
db.test.find({
  tipo: {$type: 2}
})
{% endhighlight %}

###Expressões regulares
É possível procurarmos padrões em *strings* através do operador `$regex`, que recebe como parâmetro a expressão regular e, caso necessário, [algumas opções adicionais](http://docs.mongodb.org/manual/reference/operator/query/regex/#op._S_options).

**EXEMPLO** consultando campos nome que iniciem com a string "mongo" e seja *case insentive*:

{% highlight javascript linenos=table %}
db.test.find({
  nome: {$regex: /mongo*/i}
})
{% endhighlight %}

###Documentos encadeados

Da mesma forma que procuramos por campos, podemos procurar dentro de subdocumentos através da notação de ponto (*dot notation*).

>>Fique atento ao **uso obrigatório de aspas em volta do nome da chave**.

**EXEMPLO** procurando dentro de um campo encadeado:

{% highlight javascript linenos=table %}
db.test.find({"info.engine": "Wired Tiger"})

// exemplo de documento retornado
{
  "info": {
    "engine": "Wired Tiger"
  }
}
{% endhighlight %}

###Arrays

A sintaxe rica do MongoDB permite trabalhar com arrays através de alguns operadores especiais para arrays, como será visto a seguir:

####Correspondência ampla

Quando queremos encontrar um documento que tenha um array com pelo menos o valor procurado ou não sabemos se o campo é realmente um array, podemos fazer uma busca de correspondência ampla.

**EXEMPLO** retornando documentos que contenham a string "banana" ou o valor "banana" em qualquer posição do array de "frutas":

{% highlight javascript linenos=table %}
db.test.find({
  frutas: "banana"
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
  frutas: ["banana", "maçã"]
})

// retorno
{
  "frutas": ["banana", "maçã"]
}
{% endhighlight %}

####Operador $all

Quando temos uma **lista de valores que o array deve conter no mínimo**, podemos passá-los com o operador `$all`. A ordem não é importante para esse operador.

**EXEMPLO** retornando documentos que contenham pelo menos os valores "banana" e "maçã":

{% highlight javascript linenos=table %}
db.test.find({
  frutas: {$all: ["banana", "maçã"]}
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

####Operador $elemMatch

Para procurar documentos que contenham um *array* que possua pelo menos 1 elemento que condiz com o critério, usamos o operador `$elemMatch`, que recebe como parâmetro o critério a ser verificado no *array*.

**EXEMPLO** retornando documentos que contenham um *array* com pelo menos um elemento que seja maior que 50 e menor que 60:

{% highlight javascript linenos=table %}
db.test.find({
  n: {$elemMatch: {$gt:50, $lt:60}}
})

// retorno
{
  "n": [51]
},
{
  "n": [40, 700, 55]
}
{% endhighlight %}

####Operador $in

É possível usarmos também o operador `$in` em arrays, a fim de encontrar documentos que possam ter **pelo menos um dos valores buscados**. A ordem não é importante para esse operador.

**EXEMPLO** retornando documentos que contenham pelo menos o valor "banana" ou "maçã" ou os dois:

{% highlight javascript linenos=table %}
db.test.find({
  frutas: {$in: ["banana", "maçã"]}
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

##Método Update

A modificação dos documentos é feita através do método `update` que recebe pelo menos 2 parâmetros: o primeiro especifica os documentos a serem modificados (exatamente igual ao método `find`) e o segundo parâmetro especifica o quê será modificado.

**EXEMPLO** modificando um documento:

{% highlight javascript linenos=table %}
// supondo que o seguinte documento existe na coleção 'test'
{
  _id: 1,
  nome: "MongoDB",
  ano: 2007,
  concorrentes: ["CouchDB", "DocumentDB"]
}

// executando o update
db.test.update({
  _id: 1 // qual documento será modificado?
}, {
  nome: "CouchDB" // o que será modificado?
})

// resultado
{
  _id: 1,
  nome: "CouchDB"
}
{% endhighlight %}

Note que o `update` acima fez com que o documento anterior fosse completamente trocado pelo novo documento passado como segundo parâmetro. O único campo que foi mantido, foi o `_id` - esse é o comportamento padrão do método `update`.

Os próximos tópicos explicarão como modificar o valor de apenas um campo em um documento, sem que todo o documento seja substituído.

###Atomicidade do método update
Operações de modificação são sempre atômicas à nível de documento: caso mais de um cliente tente modificar o mesmo documento ao mesmo tempo, a fila de requisições é respeitada. A última requisição da fila sempre será a vencedora, dessa forma, sua ação é a que preponderará dentre as outras.

###Opções do método update
Podemos setar o comportamento padrão do método `update` através das opções que podem ser passadas dentro do terceiro parâmetro.

**EXEMPLO** opções do método `update`:

{% highlight javascript linenos=table %}
db.test.update({nome: "Mongo"}, {empresa: "10gen"}, {
  //opções
  upsert: true,
  multi: true
})
{% endhighlight %}

####Upsert - criando documentos, caso não existam
O comportamento padrão do método `update` é atualizar o documento somente se ele já existir. Caso o documento não exista, nada irá acontecer. Para sobreescrever esse comportamento setamos a opção `upsert` como `true`.

**EXEMPLO** criando um novo documento caso ele não exista:

{% highlight javascript linenos=table %}
db.test.update({nome: "Mongo"}, {empresa: "10gen"}, {
  upsert: true
})

// caso não exista o documento especificado, então cria um documento com a seguinte estrutura:
{
  "empresa": "10gen"
}
{% endhighlight %}



####Multi - modificando vários documentos ao mesmo tempo
Apenas o primeiro documento encontrado na query é afetado pelo método `update`, mesmo existindo vários documentos candidatos à modificação. Mudamos esse comportamento através da opção `multi` setada como `true`.

**EXEMPLO** modificando o campo "tipo" de todos os documentos com "ano" maior que 2005:

{% highlight javascript linenos=table %}
db.test.update({ano: {$gt: 2005}}, {tipo: "NoSql"}, {
  multi: true
})
{% endhighlight %}

###Operadores de modificação
Estão disponíveis uma grande variedade de operadores para trabalhar com a modificação de campos no MongoDB. Assim como os operadores usados com o método `find`, aqui eles também devem sempre iniciar com `$`.

####Operador $set
O método `update`, sem usar nenhum operador, substitui completamente o documento da query pelo novo (como visto anteriormente). Para evitar esse comportamento e permitir apenas a modificação de campos específicos, usamos o operador `$set`.

Caso seja executado `$set` sobre um documento que não contenha o campo que terá o valor modificado, então o campo é criado com o valor especificado.

**EXEMPLO** trocando o valor do campo "nome" de "Mongo" para "MongoDB":

{% highlight javascript linenos=table %}
db.test.update({nome: "Mongo"}, {
  $set: {nome: "MongoDB"}
})
{% endhighlight %}

####Operador $unset
Para remover um campo e seu conteúdo de um documento, usamos o operador `$unset`.

**EXEMPLO** removendo o campo "nome" de um documento:

{% highlight javascript linenos=table %}
db.test.update({nome: "Mongo"}, {
  $unset: {nome: 1}
});
{% endhighlight %}

####Operador $inc
Para incrementar um valor numérico, usamos o operador `$inc`.

**EXEMPLO** incrementando o valor do campo "likes" em 5 unidades:

{% highlight javascript linenos=table %}
db.test.update({nome: "MongoDB"}, {
  $inc: {likes: 5}
})
{% endhighlight %}

###Modificando arrays em documentos

####Operador $push
Para inserir um novo elemento no fim do array, usamos o operador `$push`, passando como parâmetro o elemento a ser adicionado.

**EXEMPLO** inserindo o elemento "abacate" no array:

{% highlight javascript linenos=table %}
db.test.update({}, {
  $push: {frutas: "abacate"}
})
{% endhighlight %}

####Operador $pushAll
Para inserir vários novos elementos no fim de um array, usamos o operador `$pushAll`, passando como parâmetro a lista de elementos a serem adicionados.

**EXEMPLO** inserindo a lista ["mamão", "abacate", "morango"] no array:

{% highlight javascript linenos=table %}
db.test.update({}, {
  $pushAll: {frutas: ["mamão", "abacate", "morango"]}
})
{% endhighlight %}

####Operador $pop
Para remover um elemento do fim do array, usamos o operador `$pop`.

**EXEMPLO** removendo o último elemento do array:

{% highlight javascript linenos=table %}
db.test.update({}, {
  $pop: {frutas: 1}
})
{% endhighlight %}

É possível também remover o primeiro elemento, passando como parâmetro o valor -1 para o operador `pop`:

**EXEMPLO** removendo o primeiro elemento do array:

{% highlight javascript linenos=table %}
db.test.update({}, {
  $pop: {frutas: -1}
})
{% endhighlight %}

####Operador $pull
Para remover um elemento em uma posição específica do array, usamos o operador `$pull`, passando como parâmetro a posição a ser removida.

**EXEMPLO** removendo o último elemento do array:

{% highlight javascript linenos=table %}
db.test.update({}, {
  $pop: {frutas: 1}
})
{% endhighlight %}

####Operador $pullAll
Para remover vários elementos específicos de um array, usamos o operador `$pullAll`, passando como parâmetro uma lista de elementos a serem removidos.

**EXEMPLO** removendo a lista ["mamão", 4, 6] do array:

{% highlight javascript linenos=table %}
db.test.update({}, {
  $pullAll: {frutas: ["mamão", 4, 6]}
})
{% endhighlight %}

####Operador $addToSet
Para inserir um novo elemento no fim do array, sabendo que ele deve ser único dentro do array, usamos o operador `$addToSet`, passando como parâmetro o elemento a ser adicionado. Esse operador age como o `$push`, sempre que o elemento a ser inserido ainda não existir no array.

**EXEMPLO** adicionando um elemento que ainda não existe no array:

{% highlight javascript linenos=table %}
db.test.update({}, {
  $addToSet: {frutas: "abacaxi"}
})

// se eu tentar novamente, nada será inserido:
db.test.update({}, {
  $addToSet: {frutas: "abacaxi"}
})
{% endhighlight %}

####Operador $each
Para inserir uma lista elementos no fim do array, sabendo que cada valor da lista deve ser único dentro do array, usamos o operador `$each` combinado com o operador `$addToSet`.

**EXEMPLO** adicionando uma lista de elementos que ainda não existem no array:

{% highlight javascript linenos=table %}
db.test.update({}, {
  $addToSet: {
    frutas: {
      $each: ["banana", "morango", "pêra"]
    }
  }
})
{% endhighlight %}


####Operador $slice
Para permitir que o array cresça somente até um tamanho predefinido, usamos o operador `$slice`, passando como parâmetro o número máximo de posições que o array deve ter. Esse operador deve ser usado junto com a inserção de novos elementos.

Se o parâmetro que especifica o número máximo de posições que o array deve ter for positivo, então são mantidos os n primeiros documentos inseridos. Caso seja negativo, então são mantidos os n últimos elementos inseridos.

**EXEMPLO** mantendo os 4 primeiros elementos no array:

{% highlight javascript linenos=table %}
// supondo o documento
{
  "_id": 1,
  "numeros": [0, 1, 2]
}

// usando $slice com parâmetro positivo
db.test.update({_id: 1}, {
  $push: {
    numeros: {
      $each: [3, 4, 5],
      $slice: 4
    }
  }
})

// tem como resultado o documento
{
  "_id": 1,
  "numeros": [0, 1, 2, 4]
}
{% endhighlight %}

**EXEMPLO** mantendo os 4 últimos elementos no array:

{% highlight javascript linenos=table %}
// supondo o documento
{
  "_id": 1,
  "numeros": [0, 1, 2]
}

// usando $slice com parâmetro positivo
db.test.update({"_id": 1}, {
  $push: {
    scores: {
      $each: [3, 4, 5],
      $slice: -4
    }
  }
})

// tem como resultado o documento
{
  "_id": 1,
  "numeros": [2, 3, 4, 5]
}
{% endhighlight %}

####Operador $sort
Para ordenar os itens do array, usamos o operador `$sort`, passando como parâmetro se queremos em ordem crescente `1` ou decrescente `-1`. Ele é usado em conjunto com a operação de `$push` e do operador `$each`. Caso queiramos apenas ordenar os elementos do array, passamos um array vazio `[]` como parâmetro do método `$each`.

**EXEMPLO** ordenando os elementos do array:

{% highlight javascript linenos=table %}
db.test.update({}, {
  $push: {
    frutas: {
      $each: ["manga", "limão"], // adiciona manga e limão
      $sort: 1 // ordena em ordem crescente
    }
  }
})
{% endhighlight %}

####Operador $setOnInsert
Quando estamos usando a opção de `upsert` (criar um novo documento, caso não exista) e queremos que um campo seja setado apenas na criação de um novo documento, então usamos o operador `$setOnInsert`. Ele é muito útil quando precisamos setar a data de criação do documento, por exemplo.

**EXEMPLO** adicionando uma data somente na inserção:

{% highlight javascript linenos=table %}
db.test.update({}, {
  $setOnInsert: {
    dataCriacao: new Date()
  }
})
{% endhighlight %}

##Método save
O método `save` funciona como um *wrapper* para o método `update` usando a opção de `upsert`. Ele recebe como parâmetro a query e o documento a ser inserido. Se especificarmos o `_id` do documento na query, então não é criado um novo documento, caso contrário, ele se comporta como o método `insert`.

**EXEMPLO** usando o método `save`:

{% highlight javascript linenos=table %}
db.test.save({}, {
  nome: "MongoDB"
})
{% endhighlight %}

##Método findAndModify
Quando queremos modificar um documento, geralmente procuramos ele com o método `find`, realizamos o `update` e retornamos o documento modificado, caso necessário. Isso pode ser perigoso, pois no tempo compreendido entre essas três operações, algum outro processo pode modificar o documento, gerando resultados inconsistentes.

Para evitar esse problema e realizar uma operação atômica, usamos o método `findAndModify`, que executa as três operações consecutivamente de forma atômica, evitando condições de disputa. Esse método permite o retorno tanto dos dados dos documentos antigos (antes da modificação), quanto dos dados dos documentos modificados.

**EXEMPLO** executando o método `findAndModify`, com o parâmetro `new` setado como `true` - isso permite retornar os documentos na forma modificada.

{% highlight javascript linenos=table %}
db.people.findAndModify({
    query: { tipo: "orientado a documentos" },
    sort: { empresa: 1 },
    update: { $inc: { likes: 1 } },
    upsert: true,
    new: true
})
{% endhighlight %}

Veja mais [sobre o findAndModify](http://docs.mongodb.org/manual/reference/method/db.collection.findAndModify/).

#Removendo documentos
A remoção dos documentos é feita através do método `remove` que recebe como parâmetro a especificação dos documentos a serem modificados (exatamente igual ao método `find`).

**EXEMPLO** removendo os documentos cujo campo "nome" é "MongoDB":

{% highlight javascript linenos=table %}
db.test.remove({
  nome: "MongoDB"
})
{% endhighlight %}

Caso seja necessário remover todos os documentos de uma coleção, podemos passar como critério um objeto vazio `{}`.

**EXEMPLO** removendo todos os documentos da coleção:

{% highlight javascript linenos=table %}
db.test.remove({})
{% endhighlight %}

Um jeito mais eficiente de realizar a remoção de todos os documentos de uma coleção é usando o método `drop`, que é executado quase que instantâneamente. Esse método também exclui qualquer índice criado na coleção.

**EXEMPLO** removendo todos os documentos e índices da coleção:

{% highlight javascript linenos=table %}
db.test.drop()
{% endhighlight %}

É possível também remover apenas um documento por vez. Para isso é só passar o valor 1 no segundo parâmetro do método `remove`.

**EXEMPLO** removendo um documento da coleção:

{% highlight javascript linenos=table %}
db.test.remove({}, 1)
{% endhighlight %}

---
layout: post
title:  "#2 MongoDB - CRUD"
description: "..."
date:   2015-07-27 00:00:00
categories: ['mongodb']
---

Qualquer operação de criar, buscar, atualizar ou remover documentos deve ocorrer sobre uma e somente uma coleção no MongoDB, devido à sua natureza não relacional.

Trabalhar com o MongoDB é uma tarefa muito mais simples para o desenvolvedor, já que não é necessário se preocupar em manter uma estrutura rígida nos documetos de uma coleção. Temos aqui uma flexibilidade muito grande, que só é possível graças à sua estrutura orientada a documentos.

A seguir, serão mostrados conceitos básicos de operações de CRUD (Create, Read, Update e Delete) sobre uma coleção no MongoDB.

# Criando documentos


## Comando Insert

A inserção de documentos em uma coleção é realizada pelo comando `insert`. Ele é similar ao comando `insert` de uma base de dados relacional, com a diferença de não precisarmos seguir um *schema* predefinido. Quando essa operação é realizada com sucesso, retorna um objeto com o número de documentos inseridos.

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

Caso queiramos inserir vários documentos ao mesmo tempo, podemos passar um array de documentos ao comando insert:

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

Para retornar todos os documentos de uma coleção, usamos o método `find` sem passar nenhum parâmetro:

{% highlight javascript linenos=table %}
db.test.find();
db.test.find({}); // igual ao comando acima
db.test.findOne(); // retorna apenas um documento
{% endhighlight %}

No Mongo Shell, ao executar o método `find`, o cursor retornará apenas os 20 primeiros documentos encontrados. Para iterar sobre o cursor, nesse caso, usa-se o comando `it`.

##Comportamento do cursor
Cursores ficam ativos por 10 minutos no cliente ou até ele ser esgotado. O comportamento de *timeout* de 10 minutos, contudo, pode ser removido através do comando `addOption`, passando como parâmetro a variável global `DBQuery.Option.noTimeout`.

{% highlight javascript linenos=table %}
db.test.find().addOption(DBQuery.Option.noTimeout);
{% endhighlight %}

###Isolamento do cursor

O mótodo `find` pode retornar o mesmo documento mais de uma vez, caso este seja modificado antes de terminar seu ciclo de vida por exaustão ou por *timeout*. Isso ocorre devido à natureza não isolada do cursor.

Para evitar que isso aconteça, podemos usar o método [snapshot](http://docs.mongodb.org/manual/faq/developers/#faq-developers-isolate-cursors). Dessa forma, apenas uma versão de cada documento será retornada no cursor:

{% highlight javascript linenos=table %}
db.test.find().snapshot();
{% endhighlight %}

####Status dos Cursores

É possível verificar a situação dos cursores ativos e inativos no Mongo Server através do comando `serverStatus`:

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

###Ordenando, limitando ou omitindo documentos através de cursores

Os cursores disponibilizam métodos para ordenar, limitar ou pular documentos em uma query. Eles podem ser invocados em conjunto sequencialmente:

{% highlight javascript linenos=table %}
// ordena em ordem decrescente pelo "nome" e, depois,
// em ordem crescente pelo "tempo"
db.test.find().sort({nome:-1, tempo: 1});

// limita em 5 documentos, no máximo
db.test.find().limit(5);

// omite os 8 primeiros documentos encontrados
db.test.find().skip(8);

// combinação dos métodos
db.test.find().sort({nome:-1, tempo: 1}).limit(5).skip(8);
{% endhighlight %}

##Especificando os documentos a serem retornados

As queries começam a ficar mais interessantes ao passo que começamos a especificar os documentos que queremos que sejam retornados através de vários operadores básicos e modificadores, como será visto nos próximos tópicos:

###Procurando uma string

Para procurar por um valor nos campos, apenas o passamos como parâmetro da query:

{% highlight javascript linenos=table %}
// o "nome" deve ser "Mongo" e o "ano" deve ser 2007
db.test.find({"nome": "Mongo", "ano": 2007});
{% endhighlight %}

Tenha cuidado ao procurar por campos com valores nulos, pois nesse caso o servidor retornará tanto os documentos que tenham campos com valor nulo, quanto os que não tenham o campo procurado, veja o exemplo:

{% highlight javascript linenos=table %}
db.test.find({"nome": null});

// pode retornar
{
  "nome": null
},

// e também pode retornar
{}
{% endhighlight %}

###Projeção de campos
Os campos a serem retornados devem ser especificados no segundo parâmetro do método `find`. Nesse exemplo, queremos retornar apenas o campo "ano":

{% highlight javascript linenos=table %}
db.test.find({"nome": "Mongo"}, {"ano": 1});
{% endhighlight %}

Da mesma forma que especificamos os campos a serem retornados, podemos especificar apenas os campos a não serem retornados. Nesse exemplo, removemos da projeção apenas o campo "status":

{% highlight javascript linenos=table %}
db.test.find({"nome": "Mongo"}, {"status": -1});
{% endhighlight %}

>> Na projeção ou usamos a remoção de campos ou a especificação dos campos a serem retornados, jamais os dois juntos, salvo no caso abaixo:


Como pode ser notado nos exemplos acima, o MongoDB sempre retorna o campo `\_id`, mesmo que não o tenhamos especificado na projeção, pois é um campo que na maioria das vezes é necessário para que possamos manipular o documento posteriormente. Podemos sobreescrever esse comportamento especificando a sua retirada da projeção:

{% highlight javascript linenos=table %}
db.test.find({"nome": "Mongo"}, {"ano": 1, "\_id": -1});
{% endhighlight %}

###Operadores
Assim como em bancos de dados relacionais, temos a possibilidade de acessar operadores lógicos (OR, AND) ou matemáticos (==, !=, <, <=, >, >=) para procurar documentos. Eles são usados com o prefixo `$` para distingui-los de campos em documentos.

####Operador $and (AND lógico)

Operações AND são realizadas usando uma vírgula entre as condições ou, alternativamente, através do modificador `$and`. Por exemplo, para retornar todos os documentos que tenham "qtd" igual a 100 e "preco" igual a 9.95:  

{% highlight javascript linenos=table %}
db.estoque.find({
  { qtd: 100 }, { preco: 9.95}
})

// alternativamente
db.estoque.find({
  $and: [ { qtd: 100 }, { preco: 9.95 } ]
})
{% endhighlight %}

####Operadores $or, $in e $nin  (OU lógico)
Condicionais OR são realizadas através do operador `$or`, que precisa de um array de condições a serem verificadas. Por exemplo, para retornar todos os documentos que tenham "qtd" igual a 100 ou "preco" igual a 9.95:  

{% highlight javascript linenos=table %}
db.estoque.find({
  $or: [ { qtd: 100 }, { preco: 9.95 } ]
})
{% endhighlight %}

Para o caso de queries mais simples com a cláusula OR, podemos usar o operador `$in` ou `$nin` (negação de `$in`) - eles são ligeiramente mais rápidos que o operador `$or`. Por exemplo, para retornar todos os documentos que tenham "qtd" igual a 20 ou "qtd" igual a 50:  

{% highlight javascript linenos=table %}
db.estoque.find({
  qtd: {$in: [20, 50]}
})
{% endhighlight %}

Tente sempre que possível usar o operador `$in`, pois o *query optimizer* usa ele de forma mais eficiente.

####Condicionais $gt, $gte, $lt, $lte (>, >=, <, <=)

Trabalhamos com inequações através dos operadores `$gt`, `$gte`, `$lt`, `$lte`, que tem a mesma função que >, >=, <, <=, respectivamente. Por exemplo, para retornar todos os documentos que tenham "qtd" maior que 100 e "preco" menor ou igual a 9.95:

{% highlight javascript linenos=table %}
db.estoque.find({
  { qtd: { $gt: 100 } }, { preco: { $lte: 9.95 } }
})
{% endhighlight %}

####Operador $ne (!=)

Quando precisamos que o documento tenha um campo com valor diferente do especificado, usamos o operador `$ne`, que se comporta como o operador !=. Por exemplo, para retornar todos os documentos que tenham "qtd" diferente de 100 e "preco" diferente de 9.95, podemos fazer a seguinte query:

{% highlight javascript linenos=table %}
db.estoque.find({
  $ne: [ { qtd: 100 }, { preco:  9.95 } ]
})
{% endhighlight %}

###Documentos encadeados

Da mesma forma que procuramos por campos, podemos procurar dentro de subdocumentos através da notação de ponto (*dot notation*). Fique atento ao uso obrigatório de aspas em volta do nome da chave.

{% highlight javascript linenos=table %}
db.estoque.find({"info.qtd": 50})

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

Quando queremos encontrar um documento que tenha um array com pelo menos o valor procurado ou não sabemos se o campo é realmente um array, podemos fazer uma busca de correspondência ampla:

{% highlight javascript linenos=table %}
db.estoque.find({
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

{% highlight javascript linenos=table %}
// o array de frutas deve conter exatamente o array ["banana", "maçã"]
db.estoque.find({
  "frutas": ["banana", "maçã"]
})

// retorno
{
  "frutas": ["banana", "maçã"]
}
{% endhighlight %}

####Operador $all

Quando temos uma **lista de valores que o array deve conter no mínimo**, podemos passar eles com o modificador `$all`. A ordem não é importante para esse operador.

Por exemplo, se quisessemos buscar documentos que contenham pelo menos os valores `"banana" e "maçã"`, fariamos:

{% highlight javascript linenos=table %}
db.estoque.find({
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

Por exemplo, se quisessemos buscar documentos que contenham pelo menos os valores `"banana" ou "maçã"`, fariamos:

{% highlight javascript linenos=table %}
db.estoque.find({
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





## Atualizando documentos

## Removendo documentos

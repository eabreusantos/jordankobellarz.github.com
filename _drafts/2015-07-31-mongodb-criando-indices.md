---
layout: post
title:  "#4 MongoDB - Criando Índices"
description: "..."
date:   2015-07-31 00:00:00
categories: ['mongodb']
---

Criar índices no MongoDB é uma maneira muito fácil de otimizar as consultas. Podemos comparar um índice no MongoDB à um índice em um livro. Já imaginou ter de procurar do começo ao fim de um livro por uma informação? Nesse caso usaríamos o seu índice, que nos indicaria a posição onde encontrar a informação de forma eficiente.

Os índices no MongoDB são exatamente iguais aos índices em um banco de dados relacional. A ideia é evitar um *table scan* toda vez que realizamos uma consulta, ou seja, não queremos que o MongoDB faça uma varredura completa dentro da coleção para encontrar os documentos procurados.

>> O conceito de índices no MongoDB é exatamente igual ao conceito de índices em um banco relacional

Para ilustrar o problema, vamos inserir 10.000.000 de registros no banco:

{% highlight javascript linenos=table %}
for(var i = 0; i < 10000000; i++) {
  db.test.insert({nome: "mongo "+i});
}
{% endhighlight %}

Se realizarmos uma consulta pelo "nome" igual a "mongo 9000000", o servidor terá de procurar em todos os documentos da coleção (10.000.000 no total) por esse nome, o que poderia levar muito tempo. Podemos visualizar esse comportamento da consulta com o método `explain`.

**EXEMPLO** executando o método `explain` para verificar a eficiência da consulta através do campo `executionStats`:

{% highlight javascript linenos=table %}
db.test.find({nome: "mongo 9000000"}).explain(true).executionStats

// resultado (parte dele foi removida para exemplo)
{
  "n" : 1,
  "nscanned" : 10000000,
  "millis" : 15862
}
{% endhighlight %}

O exemplo acima mostra que apenas um documento foi retornado (no campo "n"), contudo, antes de encontrá-lo, foi necessário percorrer todos os 10.000.000 de documentos na coleção (no campo "nscanned") e isso levou 15,8 segundos.

Para melhorar a eficiência dessa consulta, podemos limitá-la apenas ao primeiro documento encontrado, já que sabemos que existe somente um documento com "nome" igual a "mongo 9000000".

**EXEMPLO** retornando apenas o primeiro documento encontrado com o método `limit`:

{% highlight javascript linenos=table %}
db.test.find({nome: "mongo 9000000"}).limit(1).explain(true).executionStats

// resultado
{
  "n" : 1,
	"nscanned" : 9000001,
	"millis" : 2,
}
{% endhighlight %}

Veja que dessa vez a consulta foi bem mais rápida (6,6 segundos), pois teve que passar por 9.000.000 de documentos antes de encontrar o documento requisitado. Mas será que é possível melhorar isso? A resposta é: com certeza! Fazemos isso através da criação de um índice no campo "nome".

**EXEMPLO** criando um índice no campo "nome":

{% highlight javascript linenos=table %}
db.test.createIndex({nome: 1})
{% endhighlight %}

Agora, se executarmos o primeiro `find` que usamos como exemplo, teremos uma grande surpresa quanto à velocidade, pois o campo "nome" agora está dentro do índice da coleção.

**EXEMPLO** verificando a eficiência da consulta usando o índice criado

{% highlight javascript linenos=table %}
db.test.find({nome: "mongo 9000000"}).explain(true).executionStats

// resultado
{
	"n" : 1,
	"nscanned" : 1,
	"millis" : 221
}
{% endhighlight %}

Essa consulta encontrou o documento instantâneamente e levou apenas 0,2 segundos para ser concluída. Esse é o resultado quando usamos um índice bem planejado.

#Como os índices funcionam?
Quando criamos um índice no MongoDB, internamente é criada uma estrutura do tipo árvore B (*b-tree*), que é carregada em memória (caso haja espaço suficiente). Essa estrutura possibilita o uso do algoritmo de busca binária para encontrar o resultado que queremos.

<!--
TODO: colocart [esse exemplo](https://university.mongodb.com/courses/10gen/M101JS/2015_March/courseware/Week_4_Performance/52aa16abe2d4232c54a18a7a/)
-->

##Administração de índices

###Criando um índice
Usamos o método `createIndex` sobre uma coleção para criar um índice. Passamos os campos que queremos que se tornem índices no primeiro parâmetro e, no segundo parâmetro, podemos passar opções adicionais.

Ao especificar os campos que queremos que sejam indexados, passamos o nome do campo e a ordem em que ele deve ser armazenado. É usado 1 para ordem crescente e -1 para ordem decrescente.

**EXEMPLO** criação de um índice com "nome" ASC e "ano" DESC:

{% highlight javascript linenos=table %}
db.test.createIndex({nome: 1, ano: -1})
{% endhighlight %}

####Criação de índices em *background*

Uma opção muito usada ao se criar índices é fazer com que a criação aconteça em *background*, sem que a aplicação seja bloqueada. Essa opção pode fazer com que a criação do índice demore um pouco mais, porém evita que os clientes tenham que esperar essa operação terminar para poder usar o banco de dados novamente.

**EXEMPLO** criando um índice em background:

{% highlight javascript linenos=table %}
db.test.createIndex({nome: -1}, {background: true})
{% endhighlight %}

Mais opções serão exploradas nos próximos tópicos.

###Listando os índices criados
Para visualizar uma lista dos índices criados em uma coleção, usamos o método `getIndexes`.

**EXEMPLO** listando os índices já criados:

{% highlight javascript linenos=table %}
db.test.getIndexes()
{% endhighlight %}

###Removendo índices
Para remover um índice específico usamos o método `dropIndex`, passando como parâmetro o nome do índice a ser removido. Podemos saber o nome do índice no campo "key", que é retornado pelo método `getIndexes`.

Observe que o único índice que jamais poderá ser removido é o índice no campo `_id`.

**EXEMPLO** removendo um índice:

{% highlight javascript linenos=table %}
db.test.dropIndex({nome: 1})
{% endhighlight %}

#Consultas rápidas, escritas lentas
Vimos como os índices podem deixar suas consultas mais eficientes, mas isso é sempre verdade? Nesse caso estamos fazendo uma troca entre aumento de velocidade nas consultas e atraso nas escritas. Isso acontece, pois toda vez que adicionamos, modificamos ou removemos um documento da coleção, internamente o MongoDB precisa atualizar todos os índices.

Geralmente essa troca que fazemos entre ganhar velocidade nas consultas e perder nas escritas é, de fato, a melhor a ser feita, pois o ganho de velocidade pode ser evidenciado não somente quando procuramos por documentos, mas também em uma operação de modificação, por exemplo - o documento a ser modificado será encontrado de forma muito mais rápida e a modificação, nesse caso, será um pouco mais lenta, pois o MongoDB terá de atualizar todos os índices da coleção.

#Tipos de índices

Nos próximos tópicos será explicado como criar índices nos mais diversos tipos de campos e como avaliar a eficiência deles. Para os exemplos, rode o seguinte código no Mongo Shell:

{% highlight javascript linenos=table %}
var j = 1000000;
for(var i=0; i<j; i++) {
  db.test.insert({a:i, b:i, c: i})
}
{% endhighlight %}

##Compound Index - Índices em mais de um campo
Não estamos limitados a criar índices em apenas um campo. Chamamos os índices em mais de um campo de *compound indexes*. Os campos podem ser tanto valores escalares, quanto *arrays* e objetos.

**EXEMPLO** criando um índice em mais de um campo:

{% highlight javascript linenos=table %}
db.test.createIndex({a: 1, b:-1, c: 1})
{% endhighlight %}

###Consultas beneficiadas

Quando criamos um índice em mais de um campo, conseguimos cobrir uma quantidade maior de consultas diferentes. Poderíamos usar os índices criados no exemplo acima para procurar nos campos "a", "b" e "c" das seguintes formas:

{% highlight javascript linenos=table %}
db.test.find({a: "x"})
db.test.find({a: "x", b: "y"})
db.test.find({a: "x", b: "y", c: "z"})
db.test.find({a: "x", c: "z"})
{% endhighlight %}

As consultas do exemplo acima ilustram uma propriedade interessante dos índices compostos: o *query planner* pode escolher um índice composto, caso a consulta inclua seu prefixo, que é um sobconjunto desse índice, consistindo de um ou mais campos a partir do seu início. Os prefixos do índice que criamos podem ser:

* a
* a, b
* a, b, c

A última consulta do exemplo também poderia ser beneficiada pelo índice, sabendo que "a" faz parte do seu prefixo. Contudo essa estratégia não é tão eficiente, quanto se tivéssemos criado um índice para os campos "a" e "c" exclusivamente.

###Ordem das consultas

Podemos usar a ordem em que os índices foram criados para deixar a ordenação do resultado da consulta mais eficiente. Abaixo estão algumas consultas em que o índice ordenado é usado com eficiência:

{% highlight javascript linenos=table %}
db.test.find().sort({a: 1})
db.test.find().sort({a:-1})
db.test.find().sort({a: 1, b:-1})
db.test.find().sort({a:-1, b: 1})
db.test.find().sort({a: 1, b:-1, c:  1})
db.test.find().sort({a:-1, b: 1, c: -1})
{% endhighlight %}

Note que as consultas possíveis seguem exatamente a ordem que eles foram armazenados, permitindo também inverter a ordenação dos índices - podemos usar tanto `a:1, b-1, c: 1`, quanto `a:-1, b:1, c:-1`.

###Consultas não beneficiadas

Caso tentássemos qualquer uma das consultas abaixo, o *query planner* não usaria o índice criado:

{% highlight javascript linenos=table %}
db.test.find({b: "y"})
db.test.find({c: "z"})
db.test.find({b: "y", c: "z"})
{% endhighlight %}

###Interseção de índices
Além dos índices compostos, o MongoDB permite o entrelaçamento de índices criados de forma separada. Por exemplo, se tivéssemos os índices abaixo:

{% highlight javascript linenos=table %}
db.test.createIndex({a: 1})
db.test.createIndex({b: -1})
{% endhighlight %}

Se fizéssemos uma consulta envolvendo o campo "a" e o "b", o *query planner* tentaria usar os dois índices, como a consulta desse exemplo:

{% highlight javascript linenos=table %}
db.test.find({a: "x", b: "y"})
{% endhighlight %}

O mesmo vale se tivéssemos os seguintes índices:

{% highlight javascript linenos=table %}
db.test.createIndex({a: 1})
db.test.createIndex({b: -1, c: 1})
{% endhighlight %}

Nesse caso, o *query planner* usaria esses dois índices para otimizar as seguintes consultas:

{% highlight javascript linenos=table %}
db.test.find({a: "x", b: "y"})
db.test.find({a: "x", b: "y", c: "z"})
{% endhighlight %}


##Multikey Index - Índices em arrays
Quando criamos um índice em um *array*, literamente estamos tratando cada ítem do *array* como se fosse uma chave do índice separada. Supondo que temos 100 documentos na coleção e cada um tem um *array* de 50 posições, então serão criadas 5000 chaves no índice criado para essa coleção. Damos para esses tipos de índice o nome de *multikey index*.

###Compound Multikey Index - Índices em mais de um campo *array*
Seguindo a lógica de que é criado um índice para cada posição do *array*, então caso criássemos um índice em dois campos que contêm um *array* teríamos o produto cartesiano desses campos, certo? **Errado!** O MongoDB não permite criar índices compostos com dois *arrays*.

Supondo que temos um índice no campo "a" e no campo "b" e tentássemos inserir os documentos a seguir:

{% highlight javascript linenos=table %}
db.test.createIndex({a: 1, b:1});

// #1
db.test.insert({
  a: 1,
  b: [4,5,6]
})

// #2
db.test.insert({
  a: [1,2,3],
  b: 2
})

// #3
db.test.insert({
  a: [1,2,3],
  b: [4,5,6]
})
{% endhighlight %}

A primeira e a segunda operação ocorreriam sem problemas, contudo a terceira retornaria um erro informando que não é possível criar um índice em paralelo usando o *array* "a" e o array "b".

##Covered Queries
Uma consulta que usa o índice e, ao mesmo tempo, retorna um subconjunto ou todos os campos de um índice, chama-se cobertura ou *covered query*. Esse tipo de consulta tem o benefício de recuperar os dados diretamente no índice, sem a necessidade de acessar o disco para realizar essa operação.

Supondo o seguinte índice:

{% highlight javascript linenos=table %}
db.test.createIndex({a: 1, b:-1});
{% endhighlight %}

Poderíamos cobrir as seguintes consultas:

{% highlight javascript linenos=table %}
db.test.find({a:"x"}, {_id: 0, a: 1})
db.test.find({a:"x"}, {_id: 0, a: 1, b: 1})

db.test.find({a:"x", b: "y"}, {_id: 0, a: 1})
db.test.find({a:"x", b: "y"}, {_id: 0, a: 1, b: 1})
{% endhighlight %}

Observe que, obrigatóriamente, devemos remover o campo `_id` da projeção, caso ele não pertença ao índice.

>> **IMPORTANTE!** é impossível cobrir consultas, caso qualquer campo do índice seja um *array* ou um subdocumento, ou seja, *multikey indexes* e subdocumentos não podem ser cobertos.

É possível verificar se uma consulta foi coberta através do método `explain`, no qual, obrigatóriamente o campo `totalDocsExamined` deve ser igual a 0 (zero) e não pode haver um estágio de `IXCAN` dentro do estágio de `FETCH`.

Uma consulta, que aparentemente é coberta pelo índice do exemplo, seria essa:

{% highlight javascript linenos=table %}
db.test.find({a:"x"}, {_id: 0})
{% endhighlight %}

A consulta do exemplo acima não contempla uma cobertura, pois não especificamos os campos a serem retornados na projeção. Nós sabemos que existem os campos "a", "b" e "c", mas não sabemos se existe um campo "d" ou "e", por exemplo. Por isso é necessário especificar sempre os campos a serem retornados para que se possa obter uma cobertura.

##Como o *query optimizer* escolhe o melhor índice?
<iframe width="560" height="315" src="https://www.youtube.com/embed/JyQlxDc549c" frameborder="0" allowfullscreen></iframe>

#Propriedades dos índices

##Índice único
Ao criar um índice, podemos fazer com que ele seja único para toda a coleção. Para criar um índice único, ativamos a opção `unique` ao criá-lo.

**EXEMPLO** criação de um índice único

{% highlight javascript linenos=table %}
db.test.createIndex({a: 1, b:1}, {
  unique: true
});
{% endhighlight %}

Note que o valor do campo jamais poderá ser nulo e, com isso, o campo deverá estar presente em todos os documentos.

##Índice Esparso
Quando criamos um índice único, obrigatóriamente todos os documentos da coleção devem conter o campo do índice. Podemos sobreescrever esse comportamento através da opção `sparse`, que permite a criação de documetos sem o campo coberto pelo índice único. Essa opção permite uma maior flexibilidade na utilização de índices únicos.

**EXEMPLO** criação de um índice único não obrigatório

{% highlight javascript linenos=table %}
db.test.createIndex({a: 1, b:1}, {
  sparse: true
});
{% endhighlight %}

<!--
##TTL
TODO
-->

#Método explain
O MongoDB dispõe do método `explain` para verificarmos os resultados do *query planner* através dos estágios executados em uma consulta. Estes estágios são representados em forma de árvore, na qual as folhas são os primeiros estágios executados e, a raiz, é o estágio que dá o formato final do retorno da consulta.

**EXEMPLO** formas de acessar o método `explain`:

{% highlight javascript linenos=table %}
db.test.find().explain()
db.test.explain().find()
{% endhighlight %}

A primeira forma demonstrada pode ser executada somente no topo do método `find`. A segunda forma pode ser usada com qualquer método que tenha internamente uma rotina de consulta, como o método `find`, `update` e `remove`.

##Verbosidade
O método `explain` recebe como parâmetro uma `string`, que configura o nível de verbosidade do retorno do método. Por padrão, esse nível é o `queryPlanner`. Os níveis possíveis são:

* **queryPlanner:** esse é o nível de verbosidade padrão do método `explain`. Ele retorna informações sobre o índice escolhido pelo `query optimizer`;

* **executionStats:** para verificar as informações referentes à execução da consulta, incluindo os estágios de execução;

* **allPlansExecution:**  esse é o nível de verbosidade mais completo, que contempla, inclusive, resultados parciais da execução de índices rejeitados pelo `query optimizer`.


<!--

#Índices especiais
TODO

##Índices geoespaciais
TODO

##Índices Hash
TODO

##Índices Text
TODO
-->

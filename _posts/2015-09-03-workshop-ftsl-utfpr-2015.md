---
layout: post
title:  "Workshop sobre MongoDB no FTSL 2015"
description: "Workshop MongoDB ministrado no FTSL 2015"
date:   2015-09-03 00:00:00
categories: ['mongodb', 'ftsl']
---

**Bem-vindo ao tutorial do MongoDB!**

Esse material foi feito para facilitar o compreendimento do MongoDB. Leia atentamente e siga todos os passos em sequência, pois um depende do outro. Você aprenderá a modelar uma agenda de contatos.

Antes de tudo, é necessário que você importe o conjunto de dados usado nesse tutorial. Abra um terminal e execute os seguintes comandos:


{% highlight sh linenos=table %}
wget https://goo.gl/SPfTqk && mongoimport --db agenda --collection contatos --drop --file agenda.json
{% endhighlight %}

Bom estudo!

##O que é o MongoDB?
MongoDB é um banco orientado a documentos open-source que permite alta performance, alta disponibilidade e escalabilidade automática. 

###Documento

Cada registro no MongoDB é chamado de **documento**, o qual armazena uma estrutura similar ao JSON. Veja o exemplo:

{% highlight javascript linenos=table %}
{
	_id : ObjectId("55e6209332cdb66b1f8cd707"),
	nome : "Fulano Foo",
	idade : 47,
	criado : ISODate("2015-09-03T16:54:20.177Z"),
	tags : ["amigo", "universidade"],
	endereco : {
		logradouro : "Avenida 7 de Setembro",
		numero : 89989,
		complemento : "AP01",
		cidade : "Curitiba",
		uf : "PR",
		cep : "76543-210"
	},
	telefones : [
		{
			tipo : "celular",
			numero : "41 99999-9999"
		},
		{
			tipo : "fixo",
			numero : "41 33333-3333"
		}
	]
}
{% endhighlight %}

Repare que no documento acima há um campo "tags" com uma lista valores, um campo de "endereco" composto e um campo "telefones" com uma lista de objetos. Esses campos só são possíveis no MongoDB por causa da sua estrutura flexível. 

<!--
Se fossemos modelar o mesmo documento acima em um banco relacional, teríamos o seguinte esquema:

<img>representação do DER de uma lista de contatos

Nesse caso a estrutura ficou bem mais complexa. Se tivessemos que recuperar um registro completo dessa base de dados, teríamos que fazer 2 JOINS, o que se tornaria bem pesado para um conjunto grande de dados:

{% highlight sql linenos=table %}
SELECT * FROM agenda
INNER JOIN tags ON agenda.id = endereco.agenda_id,
INNER JOIN telefones ON agenda.id = telefones.agenda_id
{% endhighlight %}
-->

###Coleção
Uma coleção é responsável por agrupar documentos correlacionados. Podemos fazer uma analogia com uma tabela de um banco relacional. A grande diferença é que ela não força um *schema* para os documentos contidos nela. Por conta disso, documentos distintos podem conviver em uma mesma coleção:

{% highlight javascript linenos=table %}
{
	_id : ObjectId("55e6209332cdb66b1f8cd707"),
	nome : "Fulano Foo",
	idade : 47
},
{
	_id : ObjectId("55e6209332cdb66b1f8cd708"),
	nome : "Sicrano Bar",
	tags : ["amigo", "universidade"]
}
{% endhighlight %}

##Servidor e cliente
Hora de colocar a mão na massa!

**Iniciando o servidor**

Abra o terminal com `ctrl + shift + t` e digite o seguinte comando:

{% highlight sh linenos=table %}
mongod
{% endhighlight %}

Com o comando `mongod` você inicia o processo do servidor do MongoDB, que por padrão vem configurado para escutar conexões na porta 27017 e armazenar o conjunto de dados na pasta `/data/db`. Se o comando acima funcionar, você verá que a última mensagem é:

{% highlight sh linenos=table %}
waiting for connections on port 27017
{% endhighlight %}

**Iniciando o cliente**

Abra uma nova sessão no terminal com `ctrl + shift + t` e digite o seguinte comando:

{% highlight sh linenos=table %}
mongo
{% endhighlight %}

Usando `mongo` você conecta ao servidor e inicia a aplicação cliente chamada *Mongo shell*. Através dessa interface é possível realizar qualquer operação no MongoDB. Usaremos ela em todo o tutorial.

**Mongo shell**

O cliente Mongo shell permite acessar uma variedade de comandos de forma bem parecida como fazemos em um shell SQL. Por exemplo, para listar todos os bancos de dados no servidor, usamos o comando:

{% highlight sh linenos=table %}
show dbs
{% endhighlight %}

Uma lista de bancos de dados disponíveis irá aparecer:

{% highlight sh linenos=table %}
agenda  0.078GB
local   0.078GB
test    0.203G
{% endhighlight %}

Para escolher qual dos bancos será manipulado, use o comando `use <nome do banco>`:

{% highlight sh linenos=table %}
use agenda
{% endhighlight %}

Nesse momento o banco de dados escolhido ficará disponível na variável global `db`, que aponta para o banco selecionado. Execute o seguinte comando para ver em que banco você está conectado:

{% highlight sh linenos=table %}
db
{% endhighlight %}

A partir de agora o banco que você escolheu será o banco sobre o qual vamos executar todas as próximas operações. Para finalizar, veja quais coleções estão disponíveis nesse banco através do comando `show collections`:

{% highlight sh linenos=table %}
show collections
{% endhighlight %}

##Operações CRUD

Qualquer operação CRUD é feita no topo de uma coleção usando o *namespace* `db.<nome da coleção>`. Basicamente, vamos usar 4 operações:

{% highlight javascript linenos=table %}
db.contatos.insert({...})

db.contatos.find({...})

db.contatos.update({...}, {...})

db.contatos.delete({...})
{% endhighlight %}

**Inserindo um documento**

Para inserir um novo documento, passamos como parâmetro do método `insert` o objeto que será armazenado, por exemplo:

{% highlight javascript linenos=table %}
db.contatos.insert({
	nome: 'Fulano',
	idade: 20
})
{% endhighlight %}

Por padrão, o MongoDB retornará a seguinte mensagem: 

{% highlight javascript linenos=table %}
WriteResult({ "nInserted" : 1 })
{% endhighlight %}

Sabendo que todos os documentos devem ter um campo `_id`, quando não o adicionamos explicitamente, o Mongo shell ou o *driver* acaba adicionando-o de forma automática.

>>**Exercício 1:** insira mais um contato na sua agenda. Ele deverá conter nome, idade e um array de tags. 

**Listando documentos**

O próximo passo é verificar quais documentos estão na coleção de contatos. Use o método `find` para isso:

{% highlight javascript linenos=table %}
db.contatos.find()
{% endhighlight %}

Como retorno, você verá uma lista com os documentos que estão na coleção. Para melhorar a visualização dos documentos retornados pelo método `find`, use o método `pretty` no topo do `find`:

{% highlight javascript linenos=table %}
db.contatos.find().pretty()
{% endhighlight %}

Compare os telefones dos contatos da sua agenda: alguns tem o campo "operadora" e outros não, mas ainda assim convivem juntos na mesma coleção. É por esse motivo que classificamos o MongoDB como *schemaless*, pois não é forçado um modelo padrão para os documentos de uma mesma coleção.

**Buscando pelo nome do contato**

Supondo que quero encontrar meu amigo Fulano Foo, posso passar um parâmetro ao método `find` contendo o campo e o valor que estamos procurando:

{% highlight javascript linenos=table %}
db.contatos.find({
	nome: "Fulano Foo"
}).pretty()
{% endhighlight %}

Ao executar o comando serão retornados todos os contatos com nome igual a "Fulano Foo".

>>**Exercício 2:** procure pelos contatos que têm 18 anos.

**Buscando pela cidade do contato**

Se eu quiser saber quais são os contatos de Curitiba, posso buscar dentro do subdocumento de "endereço" da seguinte forma:

{% highlight javascript linenos=table %}
db.contatos.find({
	"endereco.cidade": "Curitiba"
}).pretty()
{% endhighlight %}

>> note que dessa vez tivemos que usar aspas em volta da chave para procurar dentro do subdocumento

>>**Exercício 3:** procure pelos contatos que estão no estado de São Paulo.

**E se eu quiser encontrar somente os amigos?**

Podemos buscar dentro do array de tags da mesma forma que buscamos por uma string qualquer. Para encontrar somente os amigos use o seguinte comando:

{% highlight javascript linenos=table %}
db.contatos.find({
	tags: "amigos"
}).pretty()
{% endhighlight %}

>>**Exercício 4:** procure pelos contatos que têm telefone fixo.

**Encontrando os amigos de Santa Catarina**

Podemos procurar também em mais de um campo:

{% highlight javascript linenos=table %}
db.contatos.find({
	tags: "amigo",
	"endereco.uf": "SC"
}).pretty()
{% endhighlight %}

Pronto! Agora temos uma lista dos amigos de Santa Catarina.

>>**Exercício 5:** procure pelos contatos que têm telefone celular e estão em Curitiva.

**Retornando apenas os campos necessários**

Nem sempre vamos utilizar todos os campos dos documentos retornados. Podemos removê-los na projeção usando o segundo parâmetro do método `find`. Por exemplo, se quisermos retornar apenas os telefones dos contatos:

{% highlight javascript linenos=table %}
db.contatos.find({}, {
	telefones: 1
}).pretty()
{% endhighlight %}

Note que mesmo não querendo, o campo `_id` também foi retornado. Esse é o comportamento padrão do MongoDB, mas podemos forçar removendo esse campo da projeção:

{% highlight javascript linenos=table %}
db.contatos.find({}, {
	telefones: 1,
	_id: 0
}).pretty()
{% endhighlight %}

Agora o campo `_id` não é mais exibido.

>>**Exercício 6:** procure pelos contatos que têm 50 anos de idade e retorne apenas o nome deles.


**Operadores de comparação**

Até agora todas as queries foram feitas usando igualdades, mas o MongoDB não se limita somente a elas, é possível comparar valores usando modificadores especiais. Por exemplo, se quisessemos encontrar todos os amigos com idade maior que 18 anos:

{% highlight javascript linenos=table %}
db.contatos.find({
	idade: {
		$gt: 18
	}
}).pretty()
{% endhighlight %}

Existem vários operadores de comparação possíveis, inclusive para buscar dentro de arrays. Veja mais em [operadores de comparação](http://docs.mongodb.org/manual/reference/operator/query-comparison/).

>>**Exercício 7:** procure pelos contatos que têm idade menor ou igual a 40 anos.

**Operadores lógicos**

Podemos combinar cláusulas usando operadores lógicos. Por exemplo, para procurar um contato cuja cidade possa ser ou São Paulo ou Curitiba:

{% highlight javascript linenos=table %}
db.contatos.find({
	$or: [
		{"endereco.cidade": "Curitiba"},
		{"endereco.cidade": "São Paulo"}
	]
}).pretty()
{% endhighlight %}

Conheça mais alguns [operadores lógicos](http://docs.mongodb.org/manual/reference/operator/query-logical/).

>>**Exercício 8:** procure pelos contatos que têm idade igual a 20 anos ou que moram na cidade de Araucária.

**Combinando operadores**

É possível também realizar combinações de operadores. Para buscarmos os contatos com nome Fulano Foo ou que têm idade maior que 50:

{% highlight javascript linenos=table %}
db.contatos.find({
	$or: [
		{idade: {$gt:50}},
		{nome: 'Fulano Foo'}
	]
}).pretty()
{% endhighlight %}

>>**Exercício 9:** procure pelos contatos que têm idade maior que 50 anos ou que moram na cidade de Araucária.

**Colocando em ordem**

Toda agenda que se preze deve ter os registros ordenados por nome. Fazer isso é simples:

{% highlight javascript linenos=table %}
db.contatos.find({}).sort({nome: 1}).pretty()
{% endhighlight %}

Agora os resultados estão em ordem crescente de nome.

>>**Exercício 10:** procure por todos os contatos e ordene em ordem decrescente de nome e ordem crescente de idade.

**Omitindo e limitando**

Da mesma forma que usamos o método de ordenação no topo do método `find`, podemos usar outros métodos, como `skip` e `limit`, para pular e limitar registros, respectivamente:

{% highlight javascript linenos=table %}
db.contatos.find({}).skip(5).limit(3).pretty()
{% endhighlight %}

**Hora de modificar um contato**

O comando `update`, diferentemente dos outros, necessita de pelo menos 2 parâmetros: o critério para encontrar o documento que será modificado e a especificação do que será modificado. Por exemplo, caso queira mudar a idade do Fulano Bar para 90 anos, realizamos o seguinte comando: 

{% highlight javascript linenos=table %}
db.contatos.update({
	nome: "Fulano Bar"
}, {
	$set: {
		idade: 90 
	}
})
{% endhighlight %}

Como resultado temos o seguinte objeto:

{% highlight javascript linenos=table %}
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
{% endhighlight %}

Quando executamos uma operação de `update`, por padrão apenas um documento é afetado. No comando acima, caso nossa agenda tivesse mais de um Fulano Bar, então somente o primeiro encontrado seria modificado. Esse comportamento pode ser modificado através de um terceiro parâmetro de configuração:

{% highlight javascript linenos=table %}
db.contatos.update({
	nome: "Fulano Bar"
}, {
	$set: {
		idade: 5 
	}
}, {
	multi: true
})
{% endhighlight %}

Dessa forma todos os Fulano Bar terão sua idade modificada para 5 anos e o retorno será parecido com esse:

{% highlight javascript linenos=table %}
WriteResult({ "nMatched" : 3, "nUpserted" : 0, "nModified" : 3 })
{% endhighlight %}

>>**Exercício 11:** atualize o nome de todos os colegas de trabalho que moram em Curitiba para "Fulano que trabalha em Curitiba".

**Operadores de modificação**

O MongoDB também disponibiliza vários operadores para alterar documentos. Por exemplo, se quisermos adicionar mais um ítem no array de tags do Fulano Bar:

{% highlight javascript linenos=table %}
db.contatos.update({
	nome: "Fulano Bar"
}, {
  $push: {tags: "Super Amigo"} 
})
{% endhighlight %}

Veja todos os [operadores de modificação](http://docs.mongodb.org/manual/reference/operator/update/) do MongoDB.

>>**Exercício 12:** incremente em 10 unidades a idade de todos os Sicrano Foo. Dica: use o operador $inc.

**Removendo contatos**

Por fim, para remover aquele conhecido que você acha que nunca mais vai ver na vida, use o comando `remove`. Ele recebe como parâmetro o critério para encontrar os documentos a serem removidos, exatamente como no comando `find`:

{% highlight javascript linenos=table %}
db.contatos.remove({nome: 'Fulano Daz'})
{% endhighlight %}

Ao executar esse comando, todos os Fulano Daz serão removidos e o shell retornará o seguinte objeto:

{% highlight javascript linenos=table %}
WriteResult({ "nRemoved" : 2 })
{% endhighlight %}

>>**Exercício 13:** remova todos os contatos com nome "Fulano que trabalha em Curitiba" e tem idade menor que 20 anos.

<!--
**help me!**
{% highlight javascript linenos=table %}
help
db.help()
db.agenda.help()
db.agenda.find().help()
{% endhighlight %}
-->
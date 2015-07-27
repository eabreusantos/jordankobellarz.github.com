---
layout: post
title:  "MongoDB - Essencial"
description: "..."
date:   2015-07-23 14:58:00
categories: ['mongodb']
---

>> MongoDB é um banco orientado a documentos open-source que permite alta performance, alta disponibilidade e escalabilidade automática. - 10gen

MongoDB é um banco de dados **orientado a documentos** multiuso e versátil. Sua abordagem prática veio para suplantar um vão imenso que não era coberto pelos bancos relacionais, principalmente em aplicações modernas, que requerem alguns dos seguintes itens:

**quanto à estrutura**

* escalabilidade horizontal automática
* flexibilidade na modelagem dos dados
* armazenamento de estruturas não tabulares
* inserções em massa (Internet das Coisas)

**quanto ao projeto**

* desenvolvimento ágil
* iterações curtas e recorrentes
* mudanças constantes na especificação de requisitos
* prototipação rápida

**quanto à equipe**

* poucos membros
* falta de DBA

**quanto ao negócio**

* *time to market* curto
* redução de custos
* redução de riscos

O MongoDB surge sob um cenário de grande demanda por aplicações escaláveis e distribuídas para suportar o vertiginoso fluxo de dados provenientes da Internet, por essa razão sua estrutura foi projetada primeiramente para permitir escalabilidade horizontal. É nesse ponto em que o MongoDB opera com maestria.

De 2005 em diante, principalmente após a ascenção das redes sociais em âmbito mundial, ficaram evidentes os problemas de escalabilidade já previstos no começo da World Wide Web. É nesse contexto que aflora o conceito de Big Data e, quase junto, aparecem os bancos de dados não-relacionais.

Se antes usávamos técnicas de *scaling up*, ou seja, comprávamos um servidor mais potente para prover o poder computacional, hoje vemos essa técnica inviabilizada pelas aplicações sedentas por dados e, portanto, precisamos recorrer ao *scaling out*, permitindo com que esses dados sejam espalhados entre várias máquinas e, ao mesmo tempo, possam ser acessados como se fosse uma única máquina.

Essa técnica se tornou proibitiva para os bancos relacionais, por conterem estruturas naturalmente acopladas que permitem o uso da álgebra relacional e aspectos de segurança e consistência. Muitas adaptações (gambiarras funcionais) foram criadas, mas nenhuma delas foi eficaz em suprir essa demanda.

Quanto aos negócios, verificamos que o *time to market*, hoje, representa um fator crucial para o seu sucesso. A tecnologia empregada geralmente é um gargalo e, portanto, cada vez mais aumenta a necessidade de iterações rápidas no ciclo de vida de um software para prover melhorias contínuas - nada de *waterfall* aqui.

Para esse caso, os bancos não-relacionais vieram trazer agilidade às equipes de desenvolvimento. Quem define a estrutura de um banco orientado a documentos, por exemplo, são os desenvolvedores a partir da própria aplicação no momento em que estão implementando-a - não existe mais a ideia de *Schema* - o banco é definido *"on the fly"*, agora ele é *Schemaless*.

# 1 - CONCEITOS BÁSICOS DO MONGODB

O MongoDB pode ser comparado de frente à um banco relacional devido à sua organização. O que muda realmente é a flexibilidade que ele nos dá para armazenar estruturas, que jamais seria possível com um banco relacional.

Dentro de um banco encontram-se **coleções** (tabelas), com vários **documentos** (linhas) e dentro de cada documento existem vários **campos** (colunas) do tipo chave-valor.

TODO: mostrar o mapeamento

## Documentos

No coração do MongoDB há uma unidade atômica chamada **documento**. Essa unidade tem a mesma função que uma linha de uma tabela em um banco de dados relacional: armazenar dados de uma entidade.

Se a função de um documento é a mesma de uma linha de um banco de dados relacional, então por quê usaríamos um banco de dados orientado a documentos, como o MongoDB? Simples: **flexibilidade** e **agilidade**.

A natureza chave-valor dos documentos, permite que sua estrutura seja modificada ao decorrer de seu ciclo de vida. Damos para essa chave-valor o nome **campo**. É possível adicionar, alterar ou remover um campo de um documento sem a necessidade de alterar a estrutura de todos os documentos que estão na mesma coleção e sem *downtime*.

#### JSON

Em um documento no MongoDB armazenamos estruturas JSON, que possibilitam maior poder expressivo ao permitir o uso de objetos encadeados e arrays - duas coisas que são quase um pecado no mundo dos bancos relacionais.

Essas estruturas são muito semelhantes às que usamos nas linguagens de programação modernas e isso possibilita o casamento de impedâncias, que jamais seria possível se usássemos um banco relacional - nada de camadas adicionais de software, como ORM ou DAO para casar as impedâncias.

O formato JSON teve seu uso alavancado com as aplicações web modernas, pois é independente da linguagem e é muito mais enxuto que outros formatos, como o XML, por exemplo. Seu entendimento é tão simples, que em apenas 1 diagrama podemos compreender a sua sintaxe - veja a [documentação do JSON](http://json.org/json-pt.html).

A especificação JSON possibilita o uso de 6 tipos de dados:

{% highlight javascript linenos=table %}
{
  nome: "luvas de inverno", // strings
  descricao: null,          // null
  disponivel: true,         // boolean
  estoque: 549,             // numeric
  tag: ["lã","quente"],     // array
  info: {                   // object
    fabricante: "Companhia XYZ",
    ano: 2015
  }
}
{% endhighlight %}

Como os 6 tipos nativos do JSON não eram suficientes para as operações do MongoDB, adicionaram suporte para mais alguns tipos:

{% highlight javascript linenos=table %}
{
  dataCriacao: ISODate("2015-07-24T03:00:00Z"), // date (UNIX timestamp)
  RegEx: /[a-z]\w+/i,                           // expressão regular
  numeroGrande: NumberLong(99999999999),        // números de 8 e 4 bytes
  \\_id: ObjectId("507f191e810c19729de860ea"),
      // identificador único de 12 bytes para documentos
  binario: 01000110100101,                      // dados binários
  codigo: function () {                         // código Javascript
    return;
  }
}
{% endhighlight %}

####Tipos extendidos

#####Date e Timestamp
Os campos do tipo **date** são independentes do fuso horário, dependendo únicamente deste ser adicionado em um outro campo ou no próprio código, caso seja uma aplicação local.

#####RegEx
TODO: falar dos campos regex

#####Number
TODO: falar dos campos number (int de 4 bits e long de 8 bits)

#####Dados binários
TODO: falar sobre os campos de dados binários, BSON

#####Code
TODO: falar dos campos code

####Campo \_id
Cada documento do banco possui uma chave identificadora única chamada **_id**. Essa chave pode ser gerada por um algoritmo do próprio MongoDB ou criada manualmente (isso pode ser perigoso!).

O \_id poderia ser um inteiro auto incrementado como em um banco relacional, mas aí teriamos um grande problema quanto à possibilidade de dividir os documentos em vários servidores, por causa da dificuldade de sincronização que eles teriam.
Sabendo que o objetivo principal do MongoDB é permitir escalabilidade horizontal de forma fácil, foi implementado um algoritmo que gera um hash de 12 bytes de comprimento com 24 casas hexadecimais.

Esse algoritmo permite uma granularidade tão alta, que é quase impossível a ocorrência de uma colisão de hashes. Ele é composto pelas seguintes cadeias hexadecimais:

* **|0|1|2|3|**: UNIX timestamp (em segundos)
* **|4|5|6|**: identificador da máquina (hash do nome da máquina)
* **|7|8|**: id do processo do algoritmo de geração de \_id
* **|9|10|11|**: incremento (permite 256³ combinações diferentes por segundo)

**Exemplo:**  &nbsp; &nbsp;  50 7f 19 1e &nbsp; &nbsp; 81 0c 19 &nbsp; &nbsp; 72 9d &nbsp; &nbsp; e8 60 ea

O hash criado automaticamente possui um propriedade muito interessante: é ordenado pelo tempo de criação através dos quatro primeiros dígitos que formam o timestamp.

#### BSON
O usuário trabalha em alto nível usando o formato amigável JSON, porém por baixo do capô o MongoDB usa um formato de serialização chamado BSON, que é simplesmente o formato JSON convertido para dados binários e que contêm mais tipos de dados. Veja [bsonspec.org](http://bsonspec.org/).

TODO: estratégia de alocação de espaço para documentos

## Coleções
Um conjunto de documentos correlacionados é organizado dentro de coleções, que são como as tabelas de um banco de dados relacional. As coleções, contudo, são muito mais flexíveis, pois não obrigam uma estrutura rígida para seus documentos.

As coleções também possuem algumas propriedades especiais, tais como TTL (tempo de vida para os documentos), Capped Colections (a coleção tem um tamanho máximo e, caso ele seja ultrapassado, um algoritmo FIFO elimina documentos para entrarem novos), entre outras propriedades que serão mostradas nos próximos capítulos.

## Bases de dados
Cada base de dado é representada por um arquivo no disco. Ela pode armazenar várias coleções, até a capacidade máxima do disco. O conceito de bases de dados no MongoDB é o mesmo que para uma base de dados em um banco relacional.

Uma mesma instância do MongoDB pode executar várias bases de dados ao mesmo tempo, contudo devemos seguir uma regra simples para uma boa aplicação: uma aplicação deve usar somente uma base de dados. Se estiver usando mais de uma, há alguma coisa errada em sua modelagem e isso pode significar o fracasso de seu projeto.

**atenção:** um projeto poderá acessar mais de uma base de dados se ele for um gerenciador de base de dados, claro. Isso parece óbvio, mas é importante salientar, para que a afirmação do parágrafo acima não se torne um empecilho nesses casos.

## MongoServer
O servidor do MongoDB é acionado pelo comando [mongod] através da porta padrão 27017, assumindo que o diretório de trabalho encontra-se em /data/db. Ao mesmo tempo em que ele escuta conexões, também disponibiliza uma interface RESTful que é executada em uma porta 1000 vezes acima da porta em que está executando - que nesse caso seria a porta 28017.

Esse servidor roda sobre a plataforma Node.js escrita em C++, que através da engine V8 executa código arbitrário *Javascript*.

## MongoShell
O cliente do MongoDB tem um shell especial que executa funções *Javascript* arbitrárias através de seu interpretador embutido. Uma seção no MongoShell é iniciada através do comando [mongo], que tenta se comunicar pela porta padrão, caso não seja especificada.

Quando a seção no shell é iniciada, é criada uma variável global apontando para o banco de dados padrão *test*. Essa variável global é responsável por disponibilizar o acesso à todas as funcionalidades que o cliente tem permissão para o banco que ela está apontando.

O MongoDB também disponibiliza alguns comandos que não são permitidos no *Javascript*, mas que são familiares aos usuários de bancos relacionais, tais como:

* use nome_do_banco
* show dbs
* show collections
* ...

No MongoShell é possível realizar qualquer tipo de operação CRUD, assim como especificar a estrutura de coleções, seu comportamento, criar indexes, usar o framework de agregação, ver log de erros, aferir o desempenho, e muito mais.

A filosofia do MongoDB é fazer o mínimo possível no servidor e deixar todas as tarefas onerosas para o cliente, como a criação de ObjectId, processamento de queries e validação de documentos.

#### Executando scripts

Caso seja necessário executar *scripts* em *Javascript* complexos, podemos invocá-los no shell através do comando [mongo script1.js script2.js ...]. Isso pode ser muito útil para automatizar tarefas repetitivas, como remover um grupo de documentos específicos que tenha uma campo "destruir" setado como true.

O único *script* que é executado automaticamente em qualquer seção do shell é o .mongorc.js, que deve estar dentro do diretório home. Com esse arquivo, podmeos sobreescrever funcionalidades nativas do shell, que devem ficar inacessíveis para o usuário atual, por exemplo.

É possível desabilitar o .mongorc.js facilmente com o comando --norc.

# 2 - Modelagem

A maioria dos autores coloca essa seção depois das operações de CRUD, mas particularmente acho ela muito mais importante, pois quem está acostumado com o modelo relacional não tem uma concepção orientada a documentos consistente. É como pilotar um Boeing 737 tendo apenas a carteira de habilitação para motos.

TODO

# 3 - CRUD

Qualquer operação de criar, buscar, atualizar ou remover documentos deve ocorrer sobre uma e somente uma coleção no MongoDB - isso acontece devido à natureza não relacional das operações.

Trabalhar com o MongoDB é uma tarefa muito mais simples para o desenvolvedor, já que não é necessário se preocupar em manter uma estrtura rígida nos documetos de uma coleção. Temos aqui uma flexibilidade muito grande, que só é possível graças à sua estrutura orientada a documentos.

A seguir, serão mostrados conceitos básicos de operações de CRUD (Create, Read, Update e Delete) sobre uma coleção no MongoDB.

## Criando documentos


### Comando Insert

A inserção de documentos é realizada pelo comando [insert] no topo de uma coleção. Ele é similar ao comando insert de uma base de dados relacional. Em caso de sucesso, retorna um objeto com o número de documentos inseridos.

{% highlight javascript linenos=table %}
db.test.insert({
    nome: "MongoDB Shell"
});
{% endhighlight %}

Quando criamos um documento sem o campo _id, o driver gera automaticamente um ObjectId e adiciona ele ao identificador do documento. O mesmo vale para a coleção: se ela ainda não existir, então uma nova é criada para inserir o documento.

Caso queiramos inserir vários documentos ao mesmo tempo, podemos passar um array de documentos no comando insert da seguinte forma:

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
    tipo: "colunar"
  }
]);
{% endhighlight %}

Ao executar o comando acima, caso algum dos documentos não seja inserido, então os outros não serão mais processados.

É possível inserir também quantidades imensas de documentos em paralelo através da API Bulk(). Nesse caso, se um documento falhar, os outros ainda continuarão a ser inseridos.

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
    tipo: "colunar"
  }
);
bulk.execute();
{% endhighlight %}

Existem outros métodos de adicionar documentos, que funcionam quando tentamos atualizar um documento que não existe, são eles: (serão vistos em maior profundidade na próxima seção)

{% highlight javascript linenos=table %}
db.test.update({...});
db.test.findAndModify({...});
db.test.save({...});
{% endhighlight %}

## Buscando documentos
A riqueza de opções para o comando [find] é tão grande, que só dele poderia sair um livro, caso explicado em conjunto com o framework de agregação. Esse comando sempre retorna um cursor para os documentos encontrados, possiblitando operações de ordenação, limites, etc.

Supondo a necessidade de retornar todos os documentos de uma coleção, usamos o comando sem nenhum parâmetro:

{% highlight javascript linenos=table %}
db.test.find();
db.test.findOne(); // retorna apenas um documento
{% endhighlight %}

No mongo Shell, ao executar o método [find] o cursor retornará apenas os 20 primeiros documentos encontrados. Para iterar sobre o cursor, nese caso, usa-se o comando [it].

###Comportamento do cursor
Cursores ficam ativos por 10 minutos no cliente ou até ele ter sido esgotado. Para remover o comportamento de *timeout* deve ser passado como parâmetro do método [addOption] o valor DBQuery.Option.noTimeout.

{% highlight javascript linenos=table %}
db.test.find().addOption(DBQuery.Option.noTimeout);
{% endhighlight %}

####Isolamento do cursor

O mótodo find pode retornar um documento mais de uma vez, caso este seja modificado antes de terminar seu ciclo de vida por exaustão ou por timeout. Isso acontece devido à natureza não isolada do cursor.

Para evitar que isso aconteça, podemos usar o método [snapshot](http://docs.mongodb.org/manual/faq/developers/#faq-developers-isolate-cursors). Dessa forma, apenas uma versão de cada documento será retornada no cursor:

{% highlight javascript linenos=table %}
db.test.find().snapshot();
{% endhighlight %}

####Status dos Cursores
É possível verificar dados sobre os cursores ativos e inativos através do comando serverStatus:

{% highlight javascript linenos=table %}
db.serverStatus().metrics.cursor

# resultado
{
   "timedOut" : <number>,
   "open" : {
      "noTimeout" : <number>,
      "pinned" : <number>,
      "total" : <number>
   }
}
{% endhighlight %}

####Ordenando, limitando ou omitindo documentos através de cursores

{% highlight javascript linenos=table %}
/* -- ordena em ordem decrescente pelo "nome" e, depois,
   em ordem crescente pelo "tempo"
   -- limita  */
db.test.find().sort({nome:-1, tempo: 1});

/* limita */
db.test.find().sort({nome:-1, tempo: 1});
{% endhighlight %}

###Especificando os documentos a serem retornados

As queries começam a ficar mais interessantes ao passo que começamos a especificar os documentos que queremos que sejam retornados através de igualdades ou modificadores, como será visto nos próximo tópicos:

####Procurando uma string

{% highlight javascript linenos=table %}
// o "nome" deve ser "Mongo" e o ano deve ser 2007
db.test.find({"nome": "Mongo", ano: 2007});
{% endhighlight %}

####Projeção de campos
Osa campos a serem projetado devem ser especificados no segundo parâmetro do método [find]:

{% highlight javascript linenos=table %}
// o "nome" deve ser "Mongo" e deve ser retornado apenas o campo "ano"
db.test.find({"nome": "Mongo"}, {"ano": 1});
{% endhighlight %}

Deve-se observar que o documento retornado sempre retornará o campo \_id. Para remover esse campo, devemos tirá-lo da projeção:

{% highlight javascript linenos=table %}
db.test.find({"nome": "Mongo"}, {"ano": 1, "_id": -1});
{% endhighlight %}

####Operadores
Assim como em bancos de dados relacionais, temos a possibilidade de acessar operadores lógicos ou matemáticos para procurar documentos. Eles são usados com o prefixo $ para distingui-los de campos em documentos.

#####Operador $and (AND lógico)
{% highlight javascript linenos=table %}
db.estoque.find({
  { qtd: { $gt: 100 } }, { preco: { $lt: 9.95 }}
})

// alternativamente
db.estoque.find({
  $and: [ { qtd: { $gt: 100 } }, { preco: { $lt: 9.95 } } ]
})
{% endhighlight %}

#####Operador $or (OU lógico)
{% highlight javascript linenos=table %}
db.estoque.find({
  $or: [ { qtd: { $gt: 100 } }, { preco: { $lt: 9.95 } } ]
})
{% endhighlight %}

#####Operador $gt, $gte, $lt, $lte (>, >=, <, <=, respectivamente)
{% highlight javascript linenos=table %}
db.estoque.find({
  $or: [ { qtd: { $gt: 100 } }, { preco: { $lte: 9.95 } } ]
})
{% endhighlight %}

####Documentos encadeados

TODO: colocar queries sem o dot notatios e falar sobre o problema de ordem de campos

Da mesma forma que procuramos por campos, podemos procurar dentro de subdocumentos através da notação de ponto (dot notation).

{% highlight javascript linenos=table %}
db.estoque.find({"info.quantidade": {$gt:50}})
{% endhighlight %}

É necessário ficar atento ao uso obrigatório de aspas em volta do nome da chave.

####Arrays

A sintaxe rica do MongoDB permite trabalhar facilmente com arrays através de alguns mofificadores especiais.

#####Correspondência exata

{% highlight javascript linenos=table %}
// o array de frutas deve conter exatamente "banana" e "maçã"
db.estoque.find({
  "frutas": ["banana", "maçã"]
})
{% endhighlight %}

#####Correspondência ampla

{% highlight javascript linenos=table %}
// o array de frutas deve conter exatamente "banana" e qualquer outra coisa
db.estoque.find({
  "frutas": "banana"
})

// retorna documentos que contenham
{
  "frutas": ["maçã", "mamão", "banana"]
},
{
  "frutas": ["banana"]
}
{% endhighlight %}

#####Operador $all

{% highlight javascript linenos=table %}
/* o array de frutas deve conter exatamente "banana" e "maçã"
   e qualquer outra coisa */
db.estoque.find({
  "frutas": {$all: ["banana", "maçã"]}}
})

// retorna documentos que contenham
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

#####Operador $in

{% highlight javascript linenos=table %}
/* o array de frutas deve conter "banana" ou "maçã"
   e qualquer outra coisa */
db.estoque.find({
  "frutas": {$in: ["banana", "maçã"]}}
})

// retorna documentos que contenham
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

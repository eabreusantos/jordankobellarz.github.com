---
layout: post
title:  "#1 MongoDB - Conceitos Básicos"
description: "..."
date:   2015-07-24 00:00:00
categories: ['mongodb']
---

O MongoDB pode ser comparado de frente à um banco relacional devido à sua organização. O que muda, na prática, é a flexibilidade que ele nos proporciona para armazenar estruturas, que jamais seriam possíveis em um banco relacional.

Dentro de um banco encontram-se **coleções** com vários **documentos** e, dentro de cada documento, existem vários **campos** do tipo chave-valor.

**Mapeamento SQL para o MongoDB:**

<table>
  <tr><th>RDBMS</th>          <th>MongoDB</th></tr>
  <tr><td>Banco de dados</td> <td>Banco de dados</td></tr>
  <tr><td>Tabela</td>         <td>Coleção</td></tr>
  <tr><td>Linha</td>          <td>Documento</td></tr>
  <tr><td>Coluna</td>         <td>Campo</td></tr>
  <tr><td>Índice</td>         <td>Índice</td></tr>
  <tr><td>JOIN</td>           <td>Documento encadeado ou referência</td></tr>
</table>

**Executáveis:**

<table>
  <tr>
    <th></th>
    <th>MongoDB</th>
    <th>MySQL</th>
    <th>Oracle</th>
  </tr>
  <tr>
    <td>servidor</td>
    <td>mongod</td>
    <td>mysqld</td>
    <td>oracle</td>
  </tr>
  <tr>
    <td>cliente</td>
    <td>mongo</td>
    <td>mysql</td>
    <td>sqlplus</td>
  </tr>
</table>

Veja [mais comparações entre a sintaxe SQL e a do MongoDB](http://docs.mongodb.org/manual/reference/sql-comparison/).

## Documentos

No coração do MongoDB há uma unidade atômica chamada **documento**, que tem a mesma função que uma linha de uma tabela em um banco de dados relacional: armazenar dados de uma entidade.

Se a função de um documento é a mesma de uma linha de um banco de dados relacional, então por quê usaríamos um banco de dados orientado a documentos, como o MongoDB? Simples: **flexibilidade** e **agilidade**.

A natureza chave-valor dos documentos permite que sua estrutura seja modificada rapidamente ao decorrer de seu ciclo de vida. Damos para essa chave-valor o nome **campo**. É possível adicionar, alterar ou remover um campo de um documento sem a necessidade de alterar a estrutura de todos os documentos que estão na mesma coleção - lembre-se: o MongoDB é *Schemaless*.

#### JSON

Em um documento no MongoDB armazenamos estruturas JSON, que possibilitam maior poder expressivo ao permitir o uso de objetos encadeados e arrays - duas coisas que são quase um pecado no mundo dos bancos relacionais - tente imaginar uma tabela dentro de outra... não dá certo!

Essas estruturas são muito semelhantes às que usamos nas linguagens de programação modernas e isso possibilita o casamento de impedâncias, que jamais seria possível se usássemos um banco relacional - nada de camadas adicionais de *software*, como ORM ou DAO para casar as impedâncias.

O formato JSON teve seu uso alavancado com as aplicações web modernas, pois é independente da linguagem e é muito mais enxuto que outros formatos, como o XML, por exemplo. Seu entendimento é tão simples, que em apenas um diagrama podemos compreender a sua sintaxe - veja a [documentação do JSON](http://json.org/json-pt.html).

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

####Tipos extendidos

Os 6 tipos nativos do JSON não são suficientes para as operações de um banco de dados, como trabalhar com datas, por exemplo. É por esse motivo, que o MongoDB introduziu mais alguns tipos especiais para suprir essa necessidade. São eles:

{% highlight javascript linenos=table %}
{
  dataCriacao: ISODate("2015-07-24T03:00:00Z"), // date (UNIX timestamp)
  RegEx: /[a-z]\w+/i,                           // expressão regular
  numeroGrande: NumberLong(99999999999),        // números de 8 e 4 bytes
  _id: ObjectId("507f191e810c19729de860ea"),
      // identificador único de 12 bytes para documentos
  binario: 01000110100101,                      // dados binários
  codigo: function () {                         // código Javascript
    return;
  }
}
{% endhighlight %}

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

####Campo _id
Cada documento do banco possui uma chave identificadora única chamada `_id`. Essa chave pode ser gerada por um algoritmo do próprio MongoDB ou criada manualmente (isso pode ser perigoso!).

>> TODOS os documentos no MongoDB, sem exceção, possuem um campo `_id`.

O `_id` poderia ser um inteiro auto incrementado como em um banco relacional, mas aí teriamos um grande problema quanto à possibilidade de dividir os documentos entre vários servidores, causado pela dificuldade de sincronização necessária para gerar novos `_id`.

Sabendo que o objetivo principal do MongoDB é permitir escalabilidade horizontal automática, foi implementado um algoritmo que gera um *hash* de 12 *bytes* de comprimento com 24 casas hexadecimais.

Esse algoritmo permite uma granularidade tão alta, que é quase impossível a ocorrência de uma colisão de *hashes*. Ele é composto pelas seguintes cadeias hexadecimais:

{% highlight javascript linenos=table %}
{
  _id: ObjectId(507f191e810c19729de860ea),
  hashEmPartes: {
    "timestamp":  "507f191e",
    "idMaquina":  "810c19",
    "idProcesso": "729d",
    "incremento": "e860ea"
  }
}
{% endhighlight %}

* **|0|1|2|3|**: *UNIX timestamp* (em segundos)
* **|4|5|6|**: identificador da máquina (*hash* do nome da máquina)
* **|7|8|**: id do processo do algoritmo de geração de `_id`
* **|9|10|11|**: incremento (permite 256³ combinações diferentes por segundo)

O `hash` criado automaticamente possui um propriedade muito interessante: é ordenado pelo tempo de criação através dos quatro primeiros dígitos que formam o `timestamp`.

#### BSON
O usuário trabalha em alto nível usando o formato amigável JSON, porém, por baixo do capô, o MongoDB usa um formato de serialização chamado BSON, que é simplesmente o formato JSON convertido para dados binários, o qual contêm mais tipos de dados. Veja [bsonspec.org](http://bsonspec.org/).

TODO: estratégia de alocação de espaço para documentos

## Coleções
Um **conjunto de documentos correlacionados** é organizado dentro de coleções, que são como as tabelas de um banco de dados relacional. As coleções, contudo, são muito mais flexíveis, pois não obrigam uma estrutura rígida para seus documentos.

As coleções podem possuir algumas propriedades especiais, tais como TTL (tempo de vida para os documentos), Capped Colections (a coleção tem um tamanho máximo e, caso ele seja ultrapassado, um algoritmo FIFO elimina documentos para entrarem novos), entre outras propriedades que serão mostradas nos próximos capítulos.

## Bases de dados
Cada base de dado é representada por um arquivo no disco. Ela pode armazenar várias coleções, até a capacidade máxima do disco. O conceito de bases de dados no MongoDB é o mesmo que para uma base de dados em um banco relacional.

Uma mesma instância do MongoDB pode executar várias bases de dados ao mesmo tempo, contudo devemos seguir uma regra simples para uma boa aplicação:

>> uma boa aplicação deve usar somente uma base de dados. Se estiver usando mais de uma, há alguma coisa errada em sua modelagem e isso pode significar o fracasso de seu projeto.

**Atenção:** uma aplicação poderá acessar mais de uma base de dados se ela for um gerenciador de base de dados - possivelmente esse seja o único tipo de projeto que necessite de acesso múltiplo a várias bases de dados.

## Mongo Server
O servidor do MongoDB é acionado pelo comando `mongod` através da porta padrão 27017, assumindo que o diretório de trabalho encontra-se em `/data/db`. Ao mesmo tempo em que ele escuta conexões, também disponibiliza uma interface RESTful que é executada em uma porta 1000 vezes acima da porta em que está executando - que nesse caso seria a porta 28017.

Esse servidor roda sobre a plataforma Node.js escrita em C++, que através da engine V8 executa código arbitrário *Javascript*.

## Mongo Shell
O cliente do MongoDB tem um shell especial que executa funções *Javascript* arbitrárias através de seu interpretador embutido. Uma seção no MongoShell é iniciada através do comando `mongo`, que tenta se comunicar pela porta padrão, caso não seja especificada.

Quando a sessão no shell é iniciada, é também criada uma variável global apontando para o banco de dados *test*. Essa variável global é responsável por disponibilizar o acesso à todas as funcionalidades que o cliente tem permissão para o banco que ela está apontando.

O MongoDB também disponibiliza alguns comandos que não são permitidos no *Javascript*, mas que são familiares aos usuários de bancos relacionais, tais como:

* `use nome_do_banco`
* `show dbs`
* `show collections`
* ...

No Mongo Shell é possível realizar qualquer tipo de operação CRUD, assim como especificar a estrutura de coleções, seu comportamento, criar indexes, usar o framework de agregação, ver log de erros, aferir o desempenho, e muito mais.

A filosofia do MongoDB é fazer o mínimo possível no servidor e deixar todas as tarefas onerosas para o cliente, como a criação de ObjectId, processamento de queries e validação de documentos.

#### Executando scripts

Caso seja necessário executar *scripts* em *Javascript* complexos, podemos invocá-los no shell através do comando `mongo script1.js script2.js ...`. Ele pode ser muito útil para automatizar tarefas repetitivas, como remover um grupo de documentos específicos que tenha uma campo "destruir" setado como `true`, por exemplo.

O único *script* que é executado automaticamente em qualquer sessão do shell é o `.mongorc.js`, que deve estar dentro do diretório `/home`. Com esse arquivo, podemos sobreescrever funcionalidades nativas do shell, que devem ficar inacessíveis para o usuário atual, por exemplo.

É possível desabilitar o `.mongorc.js` facilmente com o comando `--norc`.

###CRUD no MongoDB

No próximo post, colocaremos a mão na massa: vamos [inserir, buscar, modificar e remover documentos em um banco de dados no MongoDB](http://jordankobellarz.github.io/mongodb/2015/07/27/mongodb-CRUD.html).

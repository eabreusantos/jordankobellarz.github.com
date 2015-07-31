---
layout: post
title:  "Guia MongoDB para RDBMS'ers"
description: "Todos os adeptos ao MySql, Oracle, SqlServer ou outros bancos de dados relacionais ouviram pelo menos uma vez sobre o MongoDB ou outro banco de dados NoSQL orientado a documentos. Nesse post irei comparar os conceitos básicos das duas tecnologias."
date:   2015-01-17 08:30:00
categories: ['mongodb']
---

Todos os adeptos ao MySql, Oracle, SqlServer ou a outros bancos de dados relacionais ouviram pelo menos
uma vez sobre bancos NoSQL orientados a documentos. Nesse post irei fazer uma introdução ao
MongoDB e, também, comparar as duas tecnologias.

> *Antes de começar quero deixar explícito que o MongoDB não substitui nenhum outro banco de dados
relacional ou outro banco de dados NoSQL que não seja orientado a documentos. Vários autores afirmam
que em torno de 90% dos projetos requerem um banco relacional e os 10% restantes um não-relacional.

## O que é o MongoDB?

> **pelo MongoDB:**
> is an open-source, document-oriented database designed for ease of development and scaling.

## Traduzindo SQL para NoSQL

Aos usuários de bancos de dados relacionais, segue abaixo uma comparação entre conceitos das duas
tecnologias e suas nomenclaturas. Mais detalhes em [SQL to MongoDB Mapping Chart](http://docs.mongodb.org/manual/reference/sql-comparison/#sql-to-mongodb-mapping-chart).

###SQL para NoSQL:

* baco de dados = banco de dados
* tabela = coleção
* linha = documento
* coluna = campo
* chave = chave
* join = documentos encadeados ou links
* chave primária = chave primária

Supondo a tabela **usuarios** contendo as propriedades **id**, **usuario_id**, **idade** e **status**.
Para criar a tabela e inserir um novo registro fariamos:

{% highlight sql linenos=table %}
CREATE TABLE usuarios (
    id MEDIUMINT NOT NULL AUTO_INCREMENT,
    usuario_id Varchar(30),
    idade int(3),
    status char(1),
    PRIMARY KEY (id)
);

INSERT INTO usuarios(usuario_id, idade, status)
VALUES ("abcd0123", 45, "A");
{% endhighlight %}

No MongoDB faríamos da seguinte maneira:

{% highlight javascript linenos=table %}
db.usuarios.insert({
    usuario_id: "abcd0123",
    idade: 45,
    status: "A"
});
{% endhighlight %}

Note que em 5 linhas conseguimos criar a coleção de usuarios e também inserir o primeiro registro.

## O básico do MongoDB

O MongoDB armazena os dados no formato de **documentos** que ficam dentro de **coleções**. Os documentos podem ser interligados através de referência ou, para a maioria dos casos, podem conter **documentos encadeados** (um documento dentro do outro).

####Documentos encadeados
O armazenamento através de documentos encadeados permite com que não sejam necessárias operações pesadas de JOIN, porém o custo é a **desnormalização**.

####Link entre documentos
Cada documento possui um campo **_id**, que é um hash **único** e **global** para todo o banco de dados. O _id de um documento permite que outro documento aponte pra ele, através de referência direta.

####Armazenamento binário
Sob o capô, o MongoDB armazena o JSON como dados binários chamados **BSON**. Isso
possibilita rapidez na busca de grandes quantidades documentos, que não seria possível se fosse usado
JSON. Essa tática, contudo, é responsável pela grande quantidade de memória gasta para guardar
uma coleção.

Essa forma de guardar os dados, permite que o MongoDB armazene qualquer tipo de arquivo binário,
como imagem, áudio, video, etc. através do **GridFS**, que quebra os arquivos em pedaços e guarda
cada um em um documento separado, os quais são indexados por um único arquivo, que facilita sua
recuperação.

Para aumentar a segurança o MongoDB usa o conceito de **réplica sets**, que promovem redundância
dos dados em múltiplos nós, possibilitando a recuperação de dados em caso de falha de hardware.

Acima de todas essas caracerísticas, a mais proeminete é a introdução do conceito de **sharding**, que
nada mais é do que o balanceamento de carga para múltiplos nós através do escalonamento horizontal.
Essa função é muito útil quando um servidor não tiver espaço suficiente para uma grande quantidade de
dados ou quando o uso de processamento e memória for tão intenso que um único nó não seria capaz de
suportar a carga.

## Quando escolher o MongoDB?
De acordo com o site [java.dzone.com](/http://java.dzone.com/articles/when-use-mongodb-rather-mysql) e
um artigo de [Andrew Brust](http://www.zdnet.com/article/rdbms-vs-nosql-how-do-you-pick/):

1. se tiver que fazer grandes quantidades de inserções
* se precisar de grande disponibilidade (recuperação instantânea por falhas de hardware).
* se for balancear grandes quantidades de dados em vários bancos
* se o banco de dados for alocado em vários servidores diferentes para maior disponibilidade
* se os dados não possuem estrutura tabular ex.: geolocalização, logs, registro de eventos, ...
* se os dados precisam ser armazenados em multiníveis
* se os modelos mudam constantemente (típico de startups)
* se integridade e normalização não forem prioridades
* se sua equipe não tiver um DBA
* se sua equipe não se importar com mudanças bruscas

###Resumo
* MongoDB é um banco de dados NoSQL orientado a documentos
* armazena JSON em forma de dados binários BSON
* possui coleções de documentos
* documentos podem conter documentos encadeados
* documentos podem referenciar outros documentos através de links
* armazena arquivos de imagem, video, audio, etc.
* facilidade em replicar o banco de dados
* facilidade em distribuir grandes massas de dados em bancos separados

Bom, é isso. E lembrem-se: os bancos de dados não-relacionais suprem problemas diferentes dos
relacionais, cada um tem sua aplicação específica, sendo que na maioria dos casos os bancos relacionais
ainda sejam mais indicados.

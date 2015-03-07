---
layout: post
title:  "Modelos de bancos NoSQL"
description: "O modelo não-relacional entrou com força no mercado e hoje é uma tendência, pois
suportam de forma eficaz a demanda por escalabilidade de aplicações como redes sociais, 
redes de compartilhamento de imagens e videos, messengers, APIs e conceitos como IOT,
BigData, aplicações web soft real time, ... "
date:   2015-01-27 08:30:00
categories: ['nosql']
---

O modelo não-relacional entrou com força no mercado e hoje é uma tendência, pois
suportam de forma eficaz a demanda por escalabilidade de aplicações como redes sociais, 
redes de compartilhamento de imagens e videos, messengers, APIs e conceitos como IOT,
BigData, aplicações web soft real time, ... 

Nesse artigo será mostrado como funcionam os diferentes modelos de bancos de dados não relacionais 
que temos atualmente e suas respectivas aplicabilidades em projetos.

## Banco orientado a documentos
* modelo mais próximo do relacional, enquanto um BD relacional guarda os dados em linhas e colunas, o BD orientado a documentos guarda em documentos
* os documentos geralmente são armazenados como JSON
* são amigáveis à programação orientada a objetos, visto que cada documento é um objeto que contem vários atributos (campos) com seus respectivos tipos
* em vez de armazenar dados correlacionados em locais diferentes, esse modelo permite que tudo fique em um só documento, eliminando a necessidade de joins
* os campos podem mudar de um documento para outro, permitindo flexibilidade na estrutura do banco, mesmo após a implementação
* as querys são relativamente menores do que em um banco relacional
* é o mais usado dentre os bancos não relacionais

#### aplicação:
* é um banco de propósito geral

#### exemplos:
* MongoDB
* CouchDB



## Banco orientado a grafos
* usa grafos para armazenar os dados
* baseado na teoria dos grafos (rede de elementos interconectados)
* cada nó representa uma entidade (como uma pessoa, empresa, conta, etc.), que contém suas respectivas propriedades
* não requer operações de join
* requer uma grande curva de aprendizado
* flexibilidade na estrutura dos nós

#### aplicação:
* redes sociais
* árvores genealógicas
* controle de acesso
* georreferenciamento

#### exemplos:
* Neo4j
* HyperGraphDB



## Banco Chave-Valor ou Tupla (Key-Value)
* comparando com um banco de dados relacional, é uma única tabela com duas colunas: uma chave primária
e um valor
* é o banco NoSQL mais simples de todos
* cada item no banco é apenas o nome de um atributo (chave) e seu respectivo valor
* podemos compará-lo a um hashmap
* os dados são acessados somente pela chave (os valores são transparentes ao sistema)
* flexibilidade na estrura dos dados
* extremamente rápido
* suporta quantidades extraordinariamente grande de dados

#### aplicação
* propósito geral
* aplicações mobile (metadados do aplicativo)


#### exemplos
* Riak
* Redis
* DynamoDB



## Banco Wide Column
* é um banco chave-valor multidimensional, ou seja, cada tupla (chave, valor) pode conter várias tuplas encadeadas
* as colunas podem ser agrupadas por famílias de colunas
* o acesso é feito através da chave da coluna

#### aplicação
* propósito geral

#### exemplos
* HBase
* Cassandra (criado pelo Facebook)



## Outros bancos NoSQL (pouco usuais)
* Orientado a objetos
* Orientado a serviços
* Multivalor 
* Multidimensional

<hr/>
### Características em comum:
* todos esses modelos permitem flexibilidade no projeto do banco de dados
* são amigáveis à programação ágil
* podem ser consistentes ou ter eventuais consistências

### Leituras adicionais:
* [lista de bancos NoSQL](http://nosql-database.org/)
* [whitepaper comparativo do MongoDB](http://www.mongodb.com/lp/white-paper/nosql-considerations)
* [resumo dos modelos não relacionais](http://imasters.com.br/artigo/17043/banco-de-dados/nosql-voce-realmente-sabe-do-que-estamos-falando/)
* [casos de uso de cada tipo de banco (leitura rápida)](http://highscalability.com/blog/2011/6/20/35-use-cases-for-choosing-your-next-nosql-database.html)

>> em breve coloco algumas imagens para abstrair a ideia de cada modelo. 
---
layout: post
title:  "MongoDB - Essencial"
description: "..."
date:   2015-07-23 14:58:00
categories: ['mongodb']
---

MongoDB é um banco de dados **orientado a documentos** multiuso e versátil. Sua abordagem prática veio para suplantar um vão imenso que não era coberto pelos bancos relacionais, principalemente em aplicações modernas, que requerem alguns dos seguintes itens:

**quanto à estrutura**

* necessidade de escalabilidade horizontal (Big Data)
* flexibilidade na modelagem dos dados
* necessidade de armazenar estruturas não tabulares
* distribuição transparente de dados em um cluster

**quanto à velocidade**

* grande quantidade de inserções consecutivas (Internet das Coisas)
* distribuição transparente de queries entre os nós de um cluster

**quanto ao desenvolvimento**

* agilidade no desenvolvimento da aplicação
* equipe pequena ou sem DBA
* mudanças recorrentes e/ou bruscas na especificação de requisitos
* prototipação rápida

**quanto ao negócio**

* *time to market* curto
* redução de custos
* redução de riscos à curto prazo

De todos esses itens, o mais proeminente é o da necessidade de escalabilidade horizontal. É nesse ponto em que o MongoDB opera com maestria, justamenta pela sua organização orientada a documentos, como será visto a seguir.

Por conseguinte, estão os fatores relativos à agilidade da equipe, pois quem define a estrutura do banco são os desenvolvedores a partir do próprio programa que estão escrevendo - não existe mais a ideia de *Schema* - o banco é definido *"on the fly"*, ou seja, ele é *Schemaless*.

# 1 - CONCEITOS BÁSICOS DO MONGODB

## Documentos

No coração do MongoDB há uma unidade atômica chamada **documento**. Essa unidade tem a mesma função que uma linha de uma tabela em um banco de dados relacional: armazenar dados de uma entidade.

Se a função de um documento é a mesma de uma linha de um banco de dados relacional, então por quê usaríamos um banco de dados orientado a documentos, como o MongoDB? Simples: **flexibilidade**.

#### JSON

Em um documento no MongoDB armazenamos estruturas JSON, que possibilitam maior poder expressivo ao permitir o uso de objetos encadeados e arrays - duas coisas que são quase um pecado no mundo dos bancos relacionais.

Essas estruturas são muito semelhantes às que usamos nas linguagens de programação modernas e isso possibilita o casamento de impedâncias, que jamais seria possível se usássemos um banco relacional - nada de camadas adicionais de software, como ORM ou DAO para casar as impedâncias.

O JSON é um formato de dados muito usado em aplicações modernas, pois é independente da linguagem e é muito mais enxuto que outros formatos, como o XML, por exemplo. Seu entendimento é tão simples, que em apenas 1 diagrama podemos compreender a sua sintaxe - veja a [documentação do JSON](http://json.org/json-pt.html).

#### \_id
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
TODO:

## Coleções
Um conjunto de documentos correlacionados é organizado dentro de coleções, que são como as tabelas de um banco de dados relacional. As coleções, contudo, são muito mais flexíveis, pois não obrigam uma estrutura rígida para seus documentos.

As coleções também possuem algumas propriedades especiais, tais como TTL (tempo de vida para os documentos), Capped Colections (a coleção tem um tamanho máximo e, caso ele seja ultrapassado, um algoritmo FIFO elimina documentos para entrarem novos), entre outras propriedades que serão mostradas nos próximos capítulos.

## Bases de dados
Cada base de dado é representada por um arquivo no disco. Ela pode armazenar várias coleções, até a capacidade máxima do disco. O conceito de bases de dados no MongoDB é o mesmo que para uma base de dados em um banco relacional.

Uma mesma instância do MongoDB pode executar várias bases de dados ao mesmo tempo, contudo devemos seguir uma regra simples para uma boa aplicação: uma aplicação deve usar somente uma base de dados. Se estiver usando mais de uma, há alguma coisa errada em sua modelagem e isso pode significar o fracasso de seu projeto.

**atenção:** um projeto poderá acessar mais de uma base de dados se ele for um gerenciador de base de dados, claro. Isso parece óbvio, mas é importante salientar, para que a afirmação do parágrafo acima não se torne um empecilho nesses casos.

# 2 - MONGOSHELL

...

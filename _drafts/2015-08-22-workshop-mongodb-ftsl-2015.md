---
layout: post
title:  "Workshop sobre MongoDB no FTSL 2015"
description: "Ministrarei um workshop sobre o MongoDB com o professor Leandro Batista no FTSL 2015."
date:   2015-08-22 00:00:00
categories: ['mongodb', 'php', 'java', 'python']
---


não se esquecer de pular linhas no Shell para novos comandos


* introdução aos bancos NoSQL
* introdução ao MongoDB
** mostrar o video do mongo world
** documentos e coleções
*** atomicidade das operaçõs em documentos
*** schemaless
*** demonstrar a criação explicita do banco e da coleção
*** mostrar documentos diferentes que podem conviver na mesma coleção
** comparação MySQL e MongoDB
** estrutura de um documento
*** JSON - tipos
*** BSON - tipos
**** ObjectId (demonstrar os segundos e o contador com esse código: for(i=0; i<100000; i++) print(new ObjectId());
** mostrar pasta de executaveis do Mongo
** Mongo Server
*** porta padrão
*** diretório de dados padrão
** Mongo Shell
*** engine V8
*** variável global db
*** comandos use, show dbs, show collections, help
*** executar um programa javascript
*CRUD no Mongo Shell
** método insert
*** anatomia do método insert
*** mostrar como o _id é criado automaticamente
*** velocidade das inserçoes por causa da validação
** método findOne
*** anatomia do método find one
** método find
*** anatomia do método find
*** cursor, funções do cursor (sort, skip, limit, count)
*** modificadores
**** operadores and e or
**** operadores > < <= >= !=
**** expressão regular
**** opoeradores para arrays
** método update
*** anatomia do método update
*** operador $set e $unset
*** operador $inc
*** operadores de array ($push, $pop, $pushAll, $pull, $pullAll, $addToSet)
*** opções do método update (upsert e multi)
**método save
*** anatomia do método save
*** problema do método save e dos upserts (quando alguém exclui um comentario que voce iria alterar posteriomente
* criando indices
** indice em um campo
** indice composto
** indice em array
** indice composto em array
** full text index
* write concern
** w
** j
* replica sets
** modelo primario e secundario
** distribuição de escritas
** heartbeat
** eleição
* sharding
** conceitro
** shard key
** balanceamento


*mongo com php
** instalação do driver com pear
** php.ini


inserir um cara standalone
inserir um cara standalone diferente

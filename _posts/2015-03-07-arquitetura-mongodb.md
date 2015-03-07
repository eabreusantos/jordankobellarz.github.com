---
layout: post
title:  "Arquitetando com MongoDB"
description: "Arquitetar um banco com o MongoDB é fácil, no entanto a flexibilidade que ele nos dá sobre a estrutura pode ser um grande problema, pois um projeto pode priorizar a consistência sobre a performance ou vice-versa. Nesse artigo demonstrarei os conceitos necessários para arquitetar um banco usando o MongoDB."
date:   2015-03-07 07:02:00
categories: ['mongodb']
---

Arquitetar um banco com o MongoDB é fácil, no entanto a flexibilidade que ele nos dá sobre a estrutura pode ser um grande problema, pois um projeto pode priorizar a consistência sobre a performance ou vice-versa. Nesse artigo demonstrarei os conceitos necessários para arquitetar um banco usando o MongoDB.

Conteúdo completo aqui: [Introdução à modelagem de dados com o MongoDB](http://docs.mongodb.org/manual/core/data-modeling-introduction/).

Ao arquitetar um banco de dados com o MongoDB devemos equilibrar as necessidades da aplicação, performance e padrões de acesso aos dados. Devemos pensar em termos de entidades (documentos) e o relacionamento entre elas, através de referência direta (links) ou documentos encadeados. Além disso, é necessário entender alguns conceitos específicos da engine do MongoDB.

###Modelo de documentos linkados (normalizado)
Essa é uma arquitetura bem próxima ao modelo relacional e, portanto, se usada em todo o banco, podemos obter sua **normalização**. Essa abordagem, contudo, possui efeitos negativos na performance das queries, visto que os relacionamentos se dão entre vários documentos.
<center><img style="max-width:600px" alt="Data model using references to link documents. Both the ``contact`` document and the ``access`` document contain a reference to the ``user`` document." src="http://docs.mongodb.org/manual/_images/data-model-normalized.png"></center>
#####casos de uso:
* sempre que for necessário representar relações N:N (vários para vários)
* quando tiver um relacionamento 1:N (um para vários), no qual N é um número muito grande de documentos
* quando a permormance (tempo e consumo de memória) não for priorizada
* sempre que for necessária a normalização dos dados

###Modelo de documentos encadeados (desnormalizado)
Encadear documentos desnormaliza a estrutura do banco, porém facilita a inserção e recuperação de dados correlacionados, demandando menos operações CRUD, pois são guardados em um mesmo documento. Essa abordagem, no entanto, pode ser perigosa se o tamanho ou a quantidade de documentos filhos crescer a ponto de consumir quantidades superiores a 1MB de memória para uma simples operação find(), por exemplo.
<center><img style="max-width:600px" alt="Data model with embedded fields that contain all related information." src="http://docs.mongodb.org/manual/_images/data-model-denormalized.png"></center>
#####casos de uso:
* quando existir um relacionamento 1:1 (um para um)
* quando existir um relacionamento 1:N (um para vários), no qual N é um número pequeno de documentos

###GridFS
Documentos maiores que 16MB (limite de um documento BSON), devem ser armazenados usando o GridFS, que separa um documento em várias partes, a fim de carregá-lo parcialmente em memória.
#####casos de uso:
* sempre que for preciso armazenar arquivos maiores que 16MB, como textos longos, imagens, videos...
* quando quiser acessar um arquivo sem carregá-lo por completo na memória

###Sharding (escalonamento horizontal)
Se o banco tiver que lidar com quantidades massivas de dados, é possível que ele seja distribuído em várias instâncias, a fim de otimizar sua performance. Isso pode ser conseguido por técnicas de sharding, escalando o banco de dados horizontalmente. Ao contrário dos bancos relacionais, o Mongo é capaz de realizar essa tarefa de forma bastante conveniente, facilitando a vida do DBA.

###Uso de chaves
As queries podem ter sua performance maximizada usando a chave (_id) para realizar as operações de CRUD.

###Coleções muito grandes
Se uma coleção crescer muito, é preferível que ela seja repartida em outras  coleções. Por exemplo uma coleção de log, que pode ser repartida em log\_usuarios e log\_sistema.

###Ciclo de vida dos dados
O MongoDB pode dar uma data de expiração para um documento ([Time to Live](http://docs.mongodb.org/manual/tutorial/expire-data/)) ou usar o algoritmo FIFO para eliminar documentos antigos ([capped collections](http://docs.mongodb.org/manual/core/capped-collections/)). Isso pode maximizar a performance do banco, caso seja possível eliminar documentos automaticamente.

<hr/>
Com esses conceitos em mente, é possível arquitetar um banco com eventual consistência (priorizando performance) ou sempre consistente (priorizando a normalização). Em próximos artigos mostrarei como arquitetar um banco para suprir diferentes tipos de requisitos.

Veja vários [exemplos de arquiteturas usando MongoDB](http://docs.mongodb.org/manual/applications/data-models/). 
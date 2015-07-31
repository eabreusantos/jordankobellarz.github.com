---
layout: post
title:  "#0 MongoDB - Introdução ao Armazenamento do Futuro"
description: "..."
date:   2015-07-23 00:00:00
categories: ['mongodb']
---

<img src="/assets/images/mongo-0.png" alt="Armazenamento do futuro" style="width: 60%; margin:0 auto">

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

O MongoDB surge sob um cenário de **grande demanda por aplicações escaláveis e distribuídas** para suportar o vertiginoso fluxo de dados provenientes da Internet, por essa razão sua estrutura foi projetada primeiramente para **permitir escalabilidade horizontal**. É nesse ponto em que o MongoDB opera com maestria.

###Projetos famintos por dados

De 2005 em diante, principalmente após a ascenção das redes sociais em âmbito mundial, ficaram evidentes os problemas de escalabilidade já previstos no começo da World Wide Web. É nesse contexto que aflora o conceito de Big Data e, quase junto, aparecem os bancos de dados não-relacionais.

Se antes usávamos técnicas de *scaling up*, ou seja, comprávamos um servidor mais potente para prover o poder computacional, hoje vemos essa técnica inviabilizada pelas aplicações "famintas" por dados e, portanto, precisamos recorrer ao *scaling out*, permitindo com que esses dados sejam espalhados entre várias máquinas e, ao mesmo tempo, possam ser acessados como se estivessem em uma única máquina.

Essa técnica se tornou proibitiva para os bancos relacionais, por conterem estruturas naturalmente acopladas para permitir o uso da álgebra relacional e prover consistência de ponta a ponta. Muitas adaptações (gambiarras funcionais) foram criadas, mas nenhuma delas foi eficaz em suprir a demanda por escalabilidade automática.

###Urgência nos negócios

No âmbito corporativo (incluindo *startups*, com certeza!), verificamos que o *time to market*, hoje, representa um fator crucial para o seu sucesso. A tecnologia empregada geralmente é um gargalo para o negócio e, portanto, cada vez mais aumenta a necessidade de iterações rápidas no ciclo de vida de um *software* para prover melhorias contínuas - nada de *waterfall* aqui.

Nesse caso, os bancos não-relacionais vieram trazer agilidade às equipes de desenvolvimento. Quem define a estrutura de um banco orientado a documentos, por exemplo, são os desenvolvedores a partir da própria aplicação no momento em que estão implementando-a - não existe mais a ideia de *Schema* - o banco é definido *"on the fly"*, agora ele é *Schemaless*.

Para se ter uma ideia de quanta complexidade removemos usando um banco de dados como o MongoDB, usaremos como exemplo o MEAN stack: com ele temos *Javascript* de ponta a ponta e isso implica o não uso de rotinas de serialização do formato JSON, ou seja, ele atravessa todas as camadas da aplicação, sem a necessidade de ser transformado.

Além disso temos aqui o casamento de impedâncias: não precisamos mais daquela camada de ORM pesadíssima, pois o formato JSON é naturalmente compatível com as estruturas de dados orientadas a objeto presentes nas aplicações modernas. Parece um sonho, não?

Nos próximos posts, serão cobertos, de forma breve, os tópicos básicos referentes à operacionalização de uma base de dados MongoDB, permitindo uma visão holística sobre essa nova tecnologia, que tem grandes chances de ser o próximo *buzzword* no mercado de banco de dados.

Desejo um bom estudo! Siga agora pra o próximo post: [MongoDB - Pensando em Documentos](http://jordankobellarz.github.io/mongodb/2015/07/24/mongodb-pensando-em-documentos.html).

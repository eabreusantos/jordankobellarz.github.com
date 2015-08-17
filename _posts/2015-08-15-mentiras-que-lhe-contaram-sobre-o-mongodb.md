---
layout: post
title:  "Mentiras que lhe contaram sobre MongoDB"
description: "..."
date:   2015-08-15 00:00:00
categories: ['mongodb']
---

O MongoDB surgiu em 2007 e só teve seu código aberto em 2009. É claro que o tempo para ele amadurecer foi bem menor do que os seus concorrentes indiretos do mundo relacional vindos dos anos 70. Nesse meio tempo foram feitas diversas melhorias, principalmente com a versão 3.0 lançada nesse ano.

Se você escutou alguém falando mal de MongoDB há 2 anos ou mais, é melhor revisar a afirmação, pois ela tem grandes chances de ter sido suplantada. Começando com as mais tênues:

##MongoDB usa *unacknowledged writes* para ganhar em *benchmarks*
Todos sabem que o MongoDB é um expoente nas propagandas e grandes eventos que realiza para promover o "banco de dados que escala o universo":

<iframe width="560" height="315" src="https://www.youtube.com/embed/3MNIrKlQp2E" frameborder="0" allowfullscreen></iframe>

É claro que eles usam os seus *benchmarks* comedores de MySql para aumentar sua base de usuários, ou melhor, clientes. Não há nenhum problema em demonstrar seus pontos fortes. Contudo, exatamente pelo motivo de serem tão "marketeiros" muitos afirmam: *"o MongoDB usa sua configuração padrão para vencer em benchmarks"*.

Qual é a configuração padrão que faz com que o MongoDB seja mais rápido? São os chamados *unacknowledged writes*, ou seja, escritas do tipo "inserir e esquecer". O servidor em uma escrita *unacknowledged* não vai retornar exatamente nada! Não se sabe se o documento realmente foi persistido, se a estrutura do documento era um JSON válido e nem se ocorreram erros de qualquer tipo.

Essa configuração realmente deixa o MongoDB muito mais rápido, mas a afirmação de que usam isso exclusivamente como propaganda é pouco fundamentada principalmente por esses quatro motivos:

* nem a equipe do MongoDB, nem a comunidade confirmaram essa afirmação
* as novas versões dos *drivers* e do Mongo shell estão configurados por padrão com *acknowledged writes*
* um bom *benchmark* deve prever as configurações necessárias para prover um ambiente aproximadamente igual para testar os dois bancos - é possível usar *unacknowledged writes* também no MySql
* bancos NoSQL são muito diferentes entre si, e mais diferentes ainda quando comparados com bancos relacionais, por isso a maioria dos *benchmarks* são tendenciosos e sem fundamento

##MongoDB é mais rápido que o MySQL
Essa afirmação vai de encontro ao tópico acima: comparar NoSQL com bancos relacionais gera grandes discrepâncias de resultados. Simular um ambiente com configurações parecidas, nesse caso, é uma tarefa complexa e ainda assim há margem para desvios.

Quanto ao MongoDB ser mais rápido nas operações de escrita, além dos *unacknowledged writes*, é também por que não realiza uma validação completa do documento durante uma operação de escrita, a qual verifica apenas a estrutura JSON válida do documento e se existe um `ObjectId()`, caso contrário gera um novo. Nada de *constraints* para verificar o conteúdo dos campos ou de integridade relacional.

>> para afirmar que o MongoDB é mais rápido que o MySql é necessário saber em
**"quais condições o MongoDB é mais rápido que o MySql"** e isso depende de projeto para projeto, não de um apanhado geral de problemas recorrentes, pois aliás, essas duas bases de dados tem aplicações bem diferentes.

##MongoDB é um comedor de memória RAM
Essa é uma meia verdade. Não necessariamente precisamos de muita memória para usar o MongoDB. A responsabilidade de alocação de memória nem fica por conta do MongoDB, mas do sistema operacional, que tenta usar o máximo de RAM para armazenar índices, realizar operações de agregação, ordenação e também, caso tenha espaço, guardar o conjunto de documentos usado pela aplicação mais recentemente.

Tudo isso funciona de modo dinâmico a partir da memória virtual, sobre a qual o sistema operacional possui autonomia para subi-la à memória RAM e para ceder espaço, caso outro processo precise usá-la. O MongoDB só tem o controle sobre como os documentos são armazenados em disco, sendo responsável pela sua estratégia de alocação, que pode diferir de acordo com a *engine* utilizada.

A melhor estratégia é: ter uma quantidade de memória RAM suficiente para seu *working set*.

##MongoDB pode substituir o MySql
Essa é uma coisa básica que qualquer um deve saber antes mesmo de começar a estudar o MongoDB: ele não veio para desbancar as bases de dados relacionais, mas para cobrir um vão que elas não cobriam com eficiência. A possibilidade de escalar automaticamente e permitir conjuntos de dados não estruturados são os principais trunfos não só do MongoDB, mas da maioria dos bancos não relacionais.

A discussão de MongoDB vs MySql está mais para SQL vs NoSQL, dois universos  distintos e que podem, inclusive, se completar. Nesse último caso, podemos ter aplicações com persistência poliglota, unindo o melhor do SQL com o melhor do NoSQL.

##MongoDB possui poucos clientes
Há três anos isso era verdade, contudo essa ideia não se perpetuou, justamente pelo motivo de grandes players terem entrado no mercado usando o MongoDB e, por necessidade, acabaram sendo desenvolvidas ferramentas de monitoramento e manutenção, plataformas em nuvem (DBaaS), ferramentas de suporte visual, ...

##MongoDB não permite JOIN
Essa é outra meia verdade. O MongoDB permite que sejam armazenadas referências de um documento para o outro, contudo é uma técnica pouco sustentável e vai de maneira contrária à ideia de encadear documentos no MongoDB.

Esse "link" feito entre os documento é realizado através dos DBRefs, que apontam para o `_id` de um documento em uma determinada coleção, parecido com um JOIN de tabelas em um banco relacional, porém sem ter nenhuma validação ou *constraint* sobre o que estamos fazendo: é possível criar um DBRef para um documento que não existe, por exemplo.

Outro problema dos DBRefs é que para os pouco familiarizados com a modelagem através de documentos encadeados no MongoDB, pode ser um empecílio para uma boa arquitetura. Quem vem do modelo relacional vai sair criando várias coleções e linkando-as, o que não é nada conveniente para um banco que permite encadear os documentos.

Arquitetar com o MongoDB é a tarefa mais difícil de todas e só se aprende estudando casos reais ou por tentativa e erro. Para modelar um banco não existe uma fórmula exata, nem mesmo regras de normalização. A ideia principal é casar o seu modelo com os padrões de acesso da sua aplicação, a fim de otimizar a performance e permitir a mínima integridade esperada para o seu conjunto de dados.

<hr/>

Por hora essas são algumas meia-verdades que lembrei. Quando lembrar mais algumas melhoro esse post. Se conhecer alguma outra que não mencionei, comente que eu vou aumentando o repertório.

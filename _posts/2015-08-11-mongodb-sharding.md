---
layout: post
title:  "#7 MongoDB - Sharding"
description: "..."
date:   2015-08-11 00:00:00
categories: ['mongodb']
---

Quando a quantidade de processamento e/ou memória de um único nó é excedida a ponto de não ser mais possível escalar seus recursos verticalmente, temos a opção de particionar os dados de uma ou mais coleções através de uma técnica chamada *sharding* ou escalonamento horizontal.

O escalonamento horizontal automático é um grande trunfo das bases de dados não relacionais, pois as entidades são desacopladas a ponto de permitir com que os dados sejam distribuídos automaticamente sem gerar problemas de integridade relacional. É exatamente por isso que é difícil de fazer *sharding* em um banco relacional, pois as regras de particionamento são criadas a nível de aplicação, o que pode gerar inconsistências e dificuldade em sua manutenção.

O MongoDB suporta escalonamento horizontal automático, ou seja, os dados poderiam ser particionados em 10, 100 ou até 1000 nós e ainda assim a aplicação enxergaria como se tudo estivesse sendo enviado e recebido de um único nó. Esse roteamento de requisições no MongoDB é feito de maneira totalmente transparente à aplicação através de um processo chamado `mongos`.

Como cada nó fica somente com um pedaço dos dados, a carga imposta sobre ele fica menor à medida que mais nós rodam em paralelo com ele. Nesse post será mostrado quais são os componentes de um *sharding*, como os dados são particionados e como configurar um *sharding* local.

<img alt="Diagram of a large collection with data distributed across 4 shards." src="http://docs.mongodb.org/manual/_images/sharded-collection.png">

##Quando particionar os dados
Mesmo que o MongoDB permita com que a tarefa de particionar os dados entre vários servidores seja feita de maneira fácil, é necessário que se tenham várias precauções antes de usá-la, pois uma vez implementado o *sharding* em um sistema é muito difícil de voltar atrás.

A maioria dos sistemas não precisam de *sharding* e os que realmente precisam, devem preferir implementá-lo somente quando não for mais possível escalar verticalmente. Essa não é uma decisão que é feita no começo do projeto, mas no decorrer do projeto, quando um único nó começar a se mostrar ineficiente para a quantidade de dados ou não tenha recursos suficientes para suportar a carga.

Em geral o *sharding* deve ser usado quando é necessário:

* mais RAM disponível
* mais espaço em disco
* reduzir a carga em um único servidor
* ler e escrever com mais *througput* que um único servidor pode suportar

##Arquitetura de um ambiente particionado

###Mongos - roteador de requisições
Em aplicações cujos dados ficam em apenas em um servidor, acessamos apenas uma instância `mongod` ou, no caso de um ambiente replicado, acessamos a instância `mongod` primária, que recebe todas as operações de escrita e as redistribui aos outros membros do *replica set*.

No caso de um ambiente com *sharding*, ao invés de acessarmos uma instância `mongod`, acessamos um roteador de requisições chamado `mongos`, o qual permite com que os dados sejam espalhados em várias instâncias `mongod` de forma transparente.

<img alt="Diagram of a sample sharded cluster for production purposes.  Contains exactly 3 config servers, 2 or more `mongos` query routers, and at least 2 shards. The shards are replica sets." src="http://docs.mongodb.org/manual/_images/sharded-cluster-production-architecture.png">

Uma instância `mongos` não guarda dados ou realiza qualquer outra tarefa, senão rotear as requisições. Tudo o que é necessário para ela é que consiga acessar as configurações de particionamento através de um `config server` e consiga acessar as instâncias `mongod` (*shards*) nas quais os dados serão persistidos.

É possível ter várias instância `mongos` para promover redundância em caso de falhas.  

###Shards - partições
Cada *shard* no sistema possui um ou vários servidores físicos (para o caso de um *replica set*), que contêm "pedaços" de uma coleção que foi particionada. Esses "pedaços" nada mais são do que um conjunto de documentos que possuem um *shard key* dentro de um determinado intervalo.

<img alt="Diagram of the shard key value space segmented into smaller ranges or chunks." src="http://docs.mongodb.org/manual/_images/sharding-range-based.png">

###Config Servers - servidores de configurações
Os servidores de configuração podem ser considerados o "cérebro" de um ambiente particionado, pois são eles os responsáveis por saber exatamente onde está cada pedaço de dado dentro do *cluster*. Eles carregam metadados sobre quais coleções foram particionadas, onde estão as partições e quaisquer outras configurações do *cluster*.

Em um ambiente de *sharding* devem existir exatamente 3 servidores de configuração, os quais devem ter o mesmo conjunto de dados para promover redundância. Apesar de se parecerem com um *replica set*, eles são independentes um do outro - todos podem receber leitura e escrita e, para manterem o mesmo conjunto de dados, eles são populados usando-se *two-phase-commits*, uma espécie de transação, que permite a sua integridade, mesmo não estando dentro de um *replica set*.

Cada servidor de configuração deve estar em uma máquina separada e, de preferência, localizados geograficamente de forma estratégica para diminuir a latência.

##Shard key
O MongoDB sabe como particionar os dados usando um índice que chamamos de *shard key*. Esse índice é exatamente igual a um índice qualquer, seja em apenas um campo ou um índice composto de uma coleção. Note que índices *multikey* jamais poderão ser usados como *shard keys*.

A grande diferença de um índice comum para um *shard key* é que o segundo deve estar presente em todos os documentos e também ser único.

Existem duas formas do MongoDB usar o *shard key* para distribuir os pedaços de dados entre os nós: por intervalo e por *hash*:

###Range based sharding
Quando temos um índice que cresce de forma homogênea, como um *timestamp* ou um contador, por exemplo, conseguimos dividi-lo em faixas, com valores mínimos e máximos possíveis para cada faixa.

Por exemplo, para um intervalo de valores entre 0 e 10000 (inclusive) poderíamos ter 5 faixas distribuídas da seguinte maneira:

* 1 - [$minKey, 2000], sendo `$minKey` o infinito negativo
* 2 - [2001, 4000]
* 3 - [4001, 6000]
* 4 - [6001, 8000]
* 5 - [8001, $maxKey], sendo `$minKey` o infinito positivo

Essas 5 faixas poderiam ser divididas entre 2 *shards* chamados `serv1` e `serv2`, sendo que o `serv1` ficaria com as faixas 1, 2 e 5 e o `serv2` com as faixas 3 e 4, por exemplo.

Sabendo que um dado encontra-se em uma determinada faixa, o `mongos` consegue direcionar as operações para apenas um pequeno número de *shards*, fazendo com que elas se tornem muito mais eficientes. Nesse caso geralmente não é necessário que todos os *shards* saibam da operação, mas somente aqueles que contêm a faixa solicitada.

Essa estratégia, contudo, pode fazer com que no decorrer do tempo um *shard key* crescente comece a se concentrar dentro do intervalo 5 (no `serv1`), fazendo com que o *cluster* fique desbalanceado.

>> usando um range based sharding, valores próximos têm grande probabilidade de estarem no mesmo shard

###Hash based shard Key
Para sanar o problema gerado pelo desbalanceamento que pode ocorrer com um *shard key* baseado em faixas de valores, podemos distribuir os dados de forma aleatória através do *hash* do índice que vamos usar como *shard key*.

Essa estratégia permite que a distribuição dos documentos entre os *shards* seja mais homogênea, contudo torna proibitivo o direcionamento das operações a um conjunto pequeno de nós, pois não se sabe em qual nó o valor do *shard key* pode estar. Nesse caso as operações seriam distribuidas entre todos os nós.

>> usando um hash based sharding, valores próximos possivelmente estarão em shards distintos

##Balanceamento
A forma com que os documentos são distribuídos entre os *shards* pode levar ao desbalanceamento do *cluster* ao decorrer do tempo. Quando o *balancer* nota que um *shard* está sendo mais requisitado que os outros, então esse processo joga as faixas de documentos do *shard* mais carregado para o menos carregado.

O *balancer* é um processo que roda dentro de uma instância `mongos` e em background, ou seja, o `mongos` não fica bloqueado durante uma rotina de balanceamento.

##Sharding e replicação
Em um ambiente de produção, é obrigatório que o *sharding* seja usado em conjunto com a replicação. A configuração é feita da seguinte maneira: cada *shard* possui um *replica set* próprio, ou seja, supondo que temos um ambiente de *sharding* com 3 nós e mais 3 nós replicados para cada shard, então teríamos no total 12 nós, dos quais 9 são *replica sets* e 3 são servidores de configuração.

##Criando um ambiente particionado
A ideia para configurar um ambiente particionado é similar à criação de um ambiente replicado. É claro que, nesse caso, temos muito mais componentes a serem configurados e regras que devem ser seguidas para manter um bom *sharding*.

Os passos apresentados aqui são exatamente os mesmos desse video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/dn45G2yw20A" frameborder="0" allowfullscreen></iframe>

Primeiro é necessário matar todos os processos do MongoDB rodando na máquina e também remover qualquer arquivo referente a um *sharding* feito anteriormente:

{% highlight javascript linenos=table %}
// matando os processos
killall mongod
killall mongos

// removendo os diretórios recursivamente
rm -rf /data/config
rm -rf /data/shard*
{% endhighlight %}

Para cada *shard*, criamos os diretórios necessários e iniciamos um *replica set* com 3 nós. Damos também um nome para cada um, que nesse caso será S0, S1 e S2:

{% highlight javascript linenos=table %}
// inicia os membros do replica set
mkdir -p /data/shard0/rs0 /data/shard0/rs1 /data/shard0/rs2
mongod --replSet s0 --logpath "s0-r0.log" --dbpath /data/shard0/rs0 --port 37017 --fork --shardsvr --smallfiles
mongod --replSet s0 --logpath "s0-r1.log" --dbpath /data/shard0/rs1 --port 37018 --fork --shardsvr --smallfiles
mongod --replSet s0 --logpath "s0-r2.log" --dbpath /data/shard0/rs2 --port 37019 --fork --shardsvr --smallfiles

// configura o replica set para o shard S0
mongo --port 37017 << 'EOF'
config = { _id: "s0", members:[
          { _id : 0, host : "localhost:37017" },
          { _id : 1, host : "localhost:37018" },
          { _id : 2, host : "localhost:37019" }]};
rs.initiate(config)
EOF
{% endhighlight %}

{% highlight javascript linenos=table %}
// inicia os membros do replica set
mkdir -p /data/shard1/rs0 /data/shard1/rs1 /data/shard1/rs2
mongod --replSet s1 --logpath "s1-r0.log" --dbpath /data/shard1/rs0 --port 47017 --fork --shardsvr --smallfiles
mongod --replSet s1 --logpath "s1-r1.log" --dbpath /data/shard1/rs1 --port 47018 --fork --shardsvr --smallfiles
mongod --replSet s1 --logpath "s1-r2.log" --dbpath /data/shard1/rs2 --port 47019 --fork --shardsvr --smallfiles

// configura o replica set para o shard S1
mongo --port 47017 << 'EOF'
config = { _id: "s1", members:[
          { _id : 0, host : "localhost:47017" },
          { _id : 1, host : "localhost:47018" },
          { _id : 2, host : "localhost:47019" }]};
rs.initiate(config)
EOF
{% endhighlight %}

{% highlight javascript linenos=table %}
// inicia os membros do replica set
mkdir -p /data/shard2/rs0 /data/shard2/rs1 /data/shard2/rs2
mongod --replSet s2 --logpath "s2-r0.log" --dbpath /data/shard2/rs0 --port 57017 --fork --shardsvr --smallfiles
mongod --replSet s2 --logpath "s2-r1.log" --dbpath /data/shard2/rs1 --port 57018 --fork --shardsvr --smallfiles
mongod --replSet s2 --logpath "s2-r2.log" --dbpath /data/shard2/rs2 --port 57019 --fork --shardsvr --smallfiles

// configura o replica set para o shard S2
mongo --port 57017 << 'EOF'
config = { _id: "s2", members:[
          { _id : 0, host : "localhost:57017" },
          { _id : 1, host : "localhost:57018" },
          { _id : 2, host : "localhost:57019" }]};
rs.initiate(config)
EOF
{% endhighlight %}

Perceba que usamos o comando `<< 'EOF'` para desprender o processo do terminal, caso contrário teríamos que abrir várias sessões do Mongo shell.

Após criar os três *replica sets* e estando devidamente configurados, criamos os diretórios e iniciamos os servidores de configuração:

{% highlight javascript linenos=table %}
mkdir -p /data/config/config-a /data/config/config-b /data/config/config-c
mongod --logpath "cfg-a.log" --dbpath /data/config/config-a --port 57040 --fork --configsvr --smallfiles
mongod --logpath "cfg-b.log" --dbpath /data/config/config-b --port 57041 --fork --configsvr --smallfiles
mongod --logpath "cfg-c.log" --dbpath /data/config/config-c --port 57042 --fork --configsvr --smallfiles
{% endhighlight %}

Por último, iniciamos o processo `mongos` na porta padrão, passando como parâmetro as portas de acesso aos 3 servidores de configuração que foram iniciados na última etapa:

{% highlight javascript linenos=table %}
mongos --logpath "mongos-1.log" --configdb localhost:57040,localhost:57041,localhost:57042 --fork
{% endhighlight %}

Aguarde 60 segundos para que todos os *replica sets* estejam funcionando e execute esses últimos comandos para configurar o roteador `mongos`:

{% highlight javascript linenos=table %}
mongo <<'EOF'
db.adminCommand( { addshard : "s0/"+"localhost:37017" } );
db.adminCommand( { addshard : "s1/"+"localhost:47017" } );
db.adminCommand( { addshard : "s2/"+"localhost:57017" } );

// criando o shard key no banco de dados test e na coleção estudantes
db.adminCommand({enableSharding: "test"})
db.adminCommand({shardCollection: "test.estudantes", key: {idAluno:1}});
EOF
{% endhighlight %}

Seu *sharding* está pronto para ser usado. Faça testes inserindo vários documentos na coleção de alunos e execute o método `explain` em uma consulta para visualizar como os documentos estão sendo distribuídos entre os nós do *cluster*.

---
layout: post
title:  "#6 MongoDB - Replicação"
description: "A maioria dos sistemas necessitam rodar em ambientes que ofereçam uma infraestrutura confiável para evitar desastres. Devemos ter o mesmo cuidado quando usamos o MongoDB através da replicação de uma base de dados."
date:   2015-08-09 00:00:00
categories: ['mongodb']
---

Até agora trabalhamos apenas com uma instância do MongoDB, mas será que essa é uma forma segura de manter uma aplicação em produção? Se a base de dados estiver sendo operada por uma equipe Kamikaze, tudo bem, mas caso não, aí devemos pensar em replicar os dados.

Replicar significa ter o mesmo conjunto de dados em diferentes nós, a fim de que, no caso de um deles cair, tenha outro à disposição - tudo isso de forma automática. Um conjunto de nós replicados chama-se *replica set*. Eles podem servir nos casos em que precisamos:

* **redundância**: manter cópias dos dados
* **alta disponibilidade**: se um nó cai, os outros entram em ação de forma rápida e transparente
* **distribui carga de leitura**: operações de leitura podem ser distribuídas entre os vários nós para diminuir a carga
* **distribuição geográfica**: diminuir a latência de acesso aos servidores, distribuindo os dados de forma geolocalizada

##Como a replicação funciona no MongoDB
Na configuração de um *replica set* encontramos basicamente dois tipos de nós: um nó primário e vários secundários. Todo *replica set* possui apenas um nó primário e vários nós secundários, somando no máximo 50 nós possíveis (antes o limite era apenas 12).

O nó primário é o único que recebe operações de escrita e redistribui para os outros nós através de um sistema de *heartbeats*, que fica responsável por sincronizar os nós em uma determinada frequência.

###Nó Primário
Todo *replica set* deve obrigatóriamente possuir um e somente um nó primário. Esse nó fica responsável por receber todas as operações de escrita e, ao mesmo tempo também permite receber leituras.

Dentro de um *replica set* quando uma operação de escrita é realizada, esse nó primeiro armazena o dado e, posteriormente escreve a operação realizada em seu *oplog*, que é uma espécie de fila de operações de escrita em formato de *log*.

Quando o nó primário cai, é realizada uma eleição através dos membros votantes, os quais escolhem o melhor nó para ser o próximo primário.

###Nós secundários
Os nós secundários de um *replica set* são responsáveis por manter cópias do nó primário, a fim de promover a redundância esperada. Esses nós ficam sincronizados entre si (incluindo o nó primário), sendo obrigatório que todos consigam acessar todos mutuamente.

A sincronização entre os nós é feita pelo sistema de *heartbeats*: em uma determianda frequência os nós secundários verificam se o *oplog* do nó primário mudou de estado e, em caso positivo, copiam-no de forma assíncrona para o seu próprio *oplog* e persistem as modificações em sua base de dados.

###Oplog
Todo nó tem seu próprio *oplog*, que é simplesmente uma fila de operações de escrita armazenada com o comportamento de uma *capped collection*. O *oplog* de qualquer nó fica disponível para que todos os nós possam copiá-lo ao seu próprio *oplog*, a fim de armazenarem o mesmo conjunto de dados em sua própria base de dados.

##Como acontece uma eleição?
Quando o nó primário cai, o sistema de *heartbeat* avisa todos os outros nós que este nó ficou inacessível e dá início à uma eleição, na qual os nós com poder de voto escolhem o melhor nó para ser o próximo nó primário.

Sabendo que em uma eleição não podem ocorrer empates (que levam à resultados inconsistentes), é necessário que um *replica set* tenha sempre um número ímpar de nós. É exatamente por esse motivo que o menor *replica set* funcional possível e, o mais comum, é formado por três membros, com as seguintes configurações:

* 1 nó primário e 2 secundários ou
* 1 nó primário, 1 secundário e 1 árbitro

###Árbitro
Como mostrado na segunda configuração do último tópico há um nó diferente dos outros chamado árbitro. Esse é um nó especial usado quando um *replica set* possui um número par de nós e necessita-se de mais um nó para que, no caso de uma eleição, jamais ocorra empate.

Mas por quê adicionaríamos um nó arbitrário se um nó secundário comum poderia ter o mesmo poder de voto? Esse nó, no caso, é escolhido por ser bem mais enxuto que um nó secundário padrão. Ele só serve para votar, não guarda nenhum dado, nem possui um *oplog*.

Ele também fica sincronizado com os outros nós através do sistema de *heartbeats*, a fim de que, no caso de uma eleição, ele possa exercer a sua função de nó votante.

##Tipos de nós secundários

###Nó de prioridade 0
Nós de prioridade 0 (zero) jamais podem ser eleitos para serem primários, mas ainda assim possuem direito de votação. Fora isso, esse nó é exatamente igual a um nó secundário comum.

<!-- TODO: quando ele é útil? -->

###Nós escondidos (*hidden members*)
Os nós do tipo *hidden* ficam escondidos para as aplicações-cliente, sendo somente acessíveis através do sistema de *heartbeat*. Eles, contudo, ainda possuem direito de voto, pois comportam-se exatamente como um nó de prioridade 0 (zero).

Esse tipo de nó é útil para atividades dedicadas, que não são executadas pelos clientes, como armazenar um *backup*, o qual só pode ser acessado manualmente em caso de perdas irreversíveis em um *replica set*.

###Nós atrasados (*delayed members*)
Nós com *delay* guardam uma cópia atrasada do nó primário (*snapshot*), a fim de promover recuperação em caso de desastres, como uma operação de *update* malsucedida, por exemplo. Ao ser configurado, passamos como parâmetro o tempo de atraso em segundos em relação ao nó primário, o qual deve ser sempre maior que a janela de manuitenção, para permitir recuperá-lo manualmente.

Todo nó atrasado é ao mesmo tempo um nó *hidden* e, por conseguinte, também um nó de prioridade 0 (zero), ou seja, ele fica escondido para os clientes e não pode se tornar um nó primário. Eles também têm poder de voto.

##Tolerância a falhas
Ao criar um *replica set* almejamos aumentar a resiliência quanto às possíveis falhas que podem ocorrer em nosso sistema, a fim de que se possa manter um mínimo de segurança para manter os dados da aplicação nas condições especificadas.

A tolerância a falhas é uma medida da capacidade de resiliência do sistema, Quanto maior, melhor. Ela é aferida em relação à quantidade de nós no sistema com capacidade de voto que sobram no caso de uma falha em qualquer um dos nós.

Podemos calcular a tolerância à falhas usando a lógica da seguinte tabela:

<table>
  <tr><th>membros</th><th>maioria votante</th><th>tolerância a falhas</th></tr>
  <tr><td>3</td><td>2</td><td>1</td></tr>
  <tr><td>4</td><td>3</td><td>1</td></tr>
  <tr><td>5</td><td>3</td><td>2</td></tr>
  <tr><td>6</td><td>4</td><td>2</td></tr>
</table>

##Criando um *replica set* local para teste
Para demonstrar como um *replica set* é configurado, vamos criar um projeto de replicação com 3 membros, sendo um primário e dois secundários. Esse é o tipo de projeto mais comum, que oferece um bom nível de segurança, por isso foi escolhido para esse tutorial.

Primeiro, criamos um arquivo para cada instância `mongod`:

{% highlight javascript linenos=table %}
mkdir -p /data/rs0-0 /data/rs0-1 /data/rs0-2
{% endhighlight %}

Para cada uma das três instâncias `mongod`, abrimos uma seção no Mongo shell:

{% highlight javascript linenos=table %}
mongod --port 27017 --dbpath /data/rs0-0 --replSet rs0 --smallfiles --oplogSize 128
{% endhighlight %}

{% highlight javascript linenos=table %}
mongod --port 27018 --dbpath /data/rs0-1 --replSet rs0 --smallfiles --oplogSize 128
{% endhighlight %}

{% highlight javascript linenos=table %}
mongod --port 27019 --dbpath /data/rs0-2 --replSet rs0 --smallfiles --oplogSize 128
{% endhighlight %}

Repare que cada instância `mongod` rodará em uma porta diferente usando um arquivo de dados também diferente, mas todas participarão do mesmo *replica set* chamado **rs0**, que foi configurado através do parâmetro `replSet`. Os parâmetros `smallfiles` e `oplogSize`, nesse caso, são usados para que os processos não deixem a máquina lenta durante os testes, já que ela estará executando três instâncias ao mesmo tempo.

Com as três instâncias rodando, conectamos um cliente a qualquer uma delas. Para efeito de teste, conecte na que está rodando na porta 27017:

{% highlight javascript linenos=table %}
mongo --port 27017
{% endhighlight %}

Através do Mongo shell, usamos o *namespace* `rs` para acessar as funcionalidades de replicação. Como queremos iniciar um *replica set*, usamos o método `initiate`, que recebe um documento de configuração:

{% highlight javascript linenos=table %}
config = {
  _id: "rs0",
  members: [
    {
      _id: 0,
      host: "localhost:27017"
    }
  ]
}

rs.initiate(config)
{% endhighlight %}

Para verificar a configuração atual do *replica set*, use o método `conf`:

{% highlight javascript linenos=table %}
rs.conf()
{% endhighlight %}

Agora podemos adicionar os nós restantes:

{% highlight javascript linenos=table %}
rs.add("localhost:27018")

rs.add("localhost:27019")
{% endhighlight %}

Verifique novamente como ficaram as configurações:

{% highlight javascript linenos=table %}
rs.conf()
{% endhighlight %}

Se tudo ocorrer corretamente, uma eleição será executada. Pronto! Seu conjunto de nós replicados está configurado.

##Testando o *replica set* criado
Na mesma sessão do Mongo shell conectada ao nó escutando a porta 27017 do *replica set* criado no tópico anterior, insira os seguintes documentos:

{% highlight javascript linenos=table %}
for(i=0; i<10000; i++) db.test.insert({"nome": "MongoDB"})
{% endhighlight %}

Nesse momento, os outros nós tentarão sincronizar com o nó primário e replicar os documentos inseridos. Para verificar se os dados realmente foram replicados, desconecte a sessão atual do Mongo shell usando `ctrl + c` e conecte-a na porta 27018:

{% highlight javascript linenos=table %}
mongo --port 27018
{% endhighlight %}

Veja que agora você está conectado à um nó secundário. Tente agora realizar uma operação de leitura:

{% highlight javascript linenos=table %}
db.test.find()
{% endhighlight %}

Uma mensagem de erro será apresentada mostrando que as operações de leitura não são permitidas em nós secundários para esse *replica set*. Essa é a configuração padrão, porém podemos modificá-la através do método `slaveOk`:

{% highlight javascript linenos=table %}
rs.slaveOk(true)
{% endhighlight %}

A partir de agora as operações de leitura foram desbloqueadas para qualquer nó no *replica set*. Execute o método `find` novamente para verificar se esse comportamento foi configurado corretamente.

Agora vamos tentar realizar uma operação de escrita nesse nó secundário:

{% highlight javascript linenos=table %}
db.test.remove({"nome": "MongoDB"})
{% endhighlight %}

Nesse caso aparecerá uma mensagem de erro dizendo que o nó não é primário e não pode receber operações de escrita diretamente.

O último teste será quanto à resiliência do *replica set*. Para isso, derrube o processo `mongod` que é o nó primário. Para saber a porta em que o nó primário está, execute o método `isMaster` e verifique o campo `primary`:

{% highlight javascript linenos=table %}
rs.isMaster()
{% endhighlight %}

Quando o nó primário é derrubado, uma eleição ocorre e um novo nó é eleito para ser o próximo primário. Execute novamente o método `isMaster` para verificar qual é o novo nó primário.

Com esses testes já é possível se ter uma noção de como a replicação funciona no MongoDB.

<hr/>

No próximo post, será apresentada a funcionalidade mais proeminete do MongoDB, o *sharding*, que permite a distribuição automática dos dados entre os nós de um *cluster*. Aprenda [como particionar seus dados com sharding](http://jordankobellarz.github.io/mongodb/2015/08/11/mongodb-sharding.html).

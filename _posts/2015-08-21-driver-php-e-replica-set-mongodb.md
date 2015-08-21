---
layout: post
title:  "Conectando o PHP ao Replica Set"
description: ""
date:   2015-08-21 00:00:00
categories: ['mongodb', 'php']
---

Nesse post será explicado como usar o driver do MongoDB com PHP conectado a um *replica set*. Caso não saiba como funciona um *replica set*, sugiro que leia primeiro o artigo sobre [replicação com MongoDB](http://jordankobellarz.github.io/mongodb/2015/08/09/mongodb-replicacao.html).

<hr/>

##Configurando o *replica set* local:

Antes de começar, crie os diretórios para os arquivos de banco de dados de cada nó no *replica set* e inicie-os:

{% highlight javascript linenos=table %}
mkdir -p /data/rs1-0 /data/rs1-1 /data/rs1-2
mongod --port 27017 --replSet rs1 --dbpath /data/rs1-0 --smallfiles --oplogSize 128 &
mongod --port 27018 --replSet rs1 --dbpath /data/rs1-1 --smallfiles --oplogSize 128 &
mongod --port 27019 --replSet rs1 --dbpath /data/rs1-2 --smallfiles --oplogSize 128 &
{% endhighlight %}

Agora precisamos iniciar e configurar o *replica set*. Abra uma sessão no Mongo shell conectada a um dos membros que acabamos de iniciar e execute o método `rs.initiate()`:

{% highlight javascript linenos=table %}
rs.initiate()
{% endhighlight %}

Será criado um arquivo de configuração padrão para o novo *replica set*. Verifique que essa configuração tem apenas o primeiro membro. Use o método `rs.status()` para isso:

{% highlight javascript linenos=table %}
rs.status()
{% endhighlight %}

Para adicionar os outros membros use o método `rs.add()`:

{% highlight javascript linenos=table %}
rs.add('localhost:27018')
rs.add('localhost:27019')
{% endhighlight %}

##Conectando o *replica set* com a aplicação PHP

Para conectar ao *replica set* é necessário que a URL de conexão contenha pelo menos o nó primário, caso contrário um `MongoConnectionException` é lançado. Crie o seguinte arquivo PHP e execute:

{% highlight php startinline=true linenos=table %}
$connStr = sprintf('mongodb://%s:%d,%s:%d,%s:%d/?replicaSet=%s',
  'localhost', 27017,
  'localhost', 27018,
  'localhost', 27019,
  'rs1');

$conn = null;

try {
  $conn = new MongoClient($connStr);
} catch (MongoConnectionException $e) {
  echo $e->getError();
}
{% endhighlight %}

Se tudo ocorreu corretamente, a partir de agora sua aplicação estará conectada ao *replica set*. Agora vamos fazer alguns testes de fogo para entender como o *driver* se comporta com um *replica set*.

###*inserindo* um documento no *replica set*:

Adicione as seguintes linhas no arquivo PHP e execute-o novamente:

{% highlight php  startinline=true linenos=table %}
$res = $conn->test->test->insert([
  'nome' => 'MongoDB'
]);
{% endhighlight %}

Você acabou de inserir um documento no nó primário, que será replicado para os outros nós a partir do momento em que o *oplog* do nó primário for copiado pelos outros nós.

###*Driver* PHP e *Write concern*

A configuração padrão para o *write concern* para o *driver* é `w = 1` e `j = 0`, ou seja, a operação de escrita é do tipo *acknownledged* e não é necessário aguardar o próximo *commit* do sistema de *journaling* ocorrer.

Para aumentar o nível de confiabilidade do nosso *replica set* é necessário configurar o parâmetro `w = 'majority'`, pois caso o nó primário caia, é necessário que a última entrada do seu *oplog* tenha sido copiada para pelo menos a maioria dos nós. Podemos configurar nossa *query* da seguinte forma:

{% highlight php startinline=true linenos=table %}
$res = $conn->test->test->insert([
  'nome' => 'MongoDB'
], [
  'w' => 'majority'
]);
{% endhighlight %}

Isso aumenta a resiliância do sistema (*fault tolerance*), contudo essa configuração ainda não é segura para aplicações que requerem que os dados sejam obrigatóriamente persistidos antes de continuar o fluxo de trabalho. Para configurar a **durabilidade**, setamos o parâmetro `j = 1`:

{% highlight php startinline=true linenos=table %}
$res = $conn->test->test->insert([
  'nome' => 'MongoDB'
], [
  'w' => 'majority',
  'j' => 1
]);
{% endhighlight %}

Essa configuração assegura que a informação tenha sido persistida no nó primário e replicada para a maioria dos outros nós. Usando `w = 'majority'` e `j = 1` em um *replica set*, podemos atingir alta disponibilidade e durabilidade para o conjunto de dados usado pela aplicação.

Mesmo assim, fique atento para a arquitetura do *replica set*: se os nós estiverem dentro do mesmo *datacenter*, pode ter certeza que a sua confiabilidade será abalada por ser susceptível a *network partitioning*, ou seja, no caso de uma falha de rede no seu *datacenter* nenhum nó ficará acessível. Isso é péssimo.

###Atenção:

>> ATENÇÃO! jamais configurar `w = n`, sendo n > 1, pois caso o número de nós restantes em uma falha seja menor que n, então o *driver* não permitirá realizar mais escritas. Tente sempre usar `w = 1` ou `w = 'majority'`, dependendo do mínimo de disponibilidade necessária à sua aplicação.

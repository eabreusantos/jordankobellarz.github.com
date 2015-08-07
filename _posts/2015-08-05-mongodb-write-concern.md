---
layout: post
title:  "#5 MongoDB - Write concern"
description: "Quem disse que o MongoDB não se parece com um banco relacional? Aprenda a fazer com que as operações de escrita sejam persistidas em disco, mesmo que ocorra um desastre sem precedentes em seu servidor."
date:   2015-08-07 00:00:00
categories: ['mongodb']
---

Seus dados são duráveis? A garantia de que uma operação de escrita ocorra com sucesso e os dados realmente sejam persistidos no disco, é verificada através do *write concern*. Essa configuração pode obrigar que uma operação de inserção, modificação ou remoção de um documento retorne ao cliente se a operação realmente foi persistida ou não.

O *write concern* possibilita que a aplicação, por exemplo, fique travada até que ela tenha certeza que a escrita efetivamente ocorreu. Nesse post será demonstrado como usar o *write concern* para casos específicos e como configurá-lo.

##Como funciona?
O MongoDB permite que o cliente saiba ou não se uma operação de escrita foi enviada ao servidor. Para isso há dois níveis chamados de *acknowledged* e *unacknowledged*. O primeiro é o padrão tanto dos *drivers* quanto do Mongo shell, pois permite um nível aceitável de segurança para as operações de escrita.

Quando a persistência dos dados no disco não é tão importante, como, por exemplo, os comentários em um *blog* sobre o seu cachorro, podemos setar as operações de escrita como *unacknowledged*. Nesse caso, quando fosse adicionado um novo comentário, então a aplicação continuaria normalmente, mesmo que tivesse ocorrido algum problema durante a inserção.

Note que para operações *unacknowledged*, nenhum tipo de erro será reportado, seja ele um erro de documento com `_id` repetido, tamanho do documento maior que 16MB ou até se o documento não estiver no formato JSON correto. Outro problema que os *drivers* conseguem tratar e, que nesse caso não seria tratado, seriam as falhas na conexão, que podem ocorrer tanto em uma solicitação ao servidor, quanto em uma resposta.

##Journaling
As escritas *acknowledged* sempre garantem que a operação foi persistida em disco? A resposta é **não**, pois através dessa opção saberíamos apenas que a operação de escrita foi recebida pelo servidor. Se foi ou não gravada em disco, nós não sabemos.

Para garantir que as operações de escrita tenham sido persistidas em disco, o MongoDB disponibiliza de uma outra configuração referente ao sistema de *Journaling*, que é uma fila de operações de escrita armazenada em um arquivo de log chamado de *journaling*.

O *journaling* é uma espécie de arquivo de banco de dados intermediário, que guarda as operações que ainda não foram persistidas em disco. Esse arquivo permite que, em caso do servidor cair de forma inesperada, o sistema possa recuperar as operações de escrita que ainda não modificaram o estado do banco de dados efetivamente.

Por padrão, a cada 100*ms* o servidor comita as operações do sistema de *journaling* no arquivo do banco de dados. É exatamente nesse momento que os dados realmente são persistidos em disco.

##Configurando o *write concern*
Para configurar o *write concern* são usados dois parâmetros principais **w** e **j**. O primeiro refere-se à necessidade ou não do servidor retornar uma resposta ao cliente (*acknowledged* e *unacknowledged*) e, o segundo, refere-se à obrigatoriedade ou não da operação ter sido comitada pelo sistema de *journaling*, ou seja, a operação espera o próximo commit ser terminado.

São possíveis 3 configurações distintas para o *write concern*, as quais permitem diferenciados níveis de segurança nas operações de escrita. São eles:

* `{w: 1, j: 0}`: essa é a configuração padrão, que permite um nível de segurança confiável (janela de vulnerabilidade pequena) e velocidade de escrita relativamente rápida
* `{w: 1, j: 1}`: esse é o maior nível de segurança possível para a escrita, contudo tem um impacto negativo na velocidade, visto que a operação obrigatóriamente deve ser comitada pelo sistema de *journaling*
* `{w: 0, j: 0}`: esse nível é o mais perigoso e só deve ser usado quando o conjunto de dados sobre os quais se está trabalhando não sejam importantes

##*Write concern* em *replica sets*
Para usar o *write concern* com um *replica set*, deve-se saber qual é a quantidade aceitável de nós que receberam a requisição do cliente. Dessa forma podemos configurar o parâmetro **w** com a quantidade de nós que deverão receber a requisição antes de retornar qualquer informação ao cliente.

Por exemplo, se **w** for igual a 1, então o cliente receberá uma resposta logo após que a solicitação seja enviada ao nó primário, sem se importar que os outros nós ainda não tenham recebido a requisição. Para o caso em que **w** é **n** e **n** é um número maior que 1, então a requisição obrigatóriamente deverá atingir **n** servidores antes do cliente receber uma resposta.

Há uma outra forma de configurar o parâmetro **w** a fim de que a requisição seja replicada  maioria dos servidores, evitando que, em caso de uma eleição para um novo membro, tenhamos o problema de eleger um membro incapaz de recuperar um `oplog`, por exemplo.

Mas o que significa "maioria"? Supondo que temos apenas um nó, a maioria seria 1. Para o caso de 2 nós, a maioria seria 2. Já com 3 nós a maioria seria também 2. A lógica é: em um grupo de **n** nós, a maioria é sempre maior que a metade de **n**.

Veja abaixo exemplos de configuração do *write concern* para *replica sets*:

* `{w: 1, j: 1}`: apenas o nó primário precisa ter recebido a requisição
* `{w: 2, j: 1}`: o nó primário e, pelo menos, um nó secundário devem ter recebido a requisição
* `{w: 3, j: 1}`: o nó primário e, pelo menos, dois nós secundários devem ter recebido a requisição
* `{w: "majority", j: 1}`: mais da metade dos nós deverão ter recebido a requisição

>>**ATENÇÃO** se **w** for maior que o número de nós que contêm dados, então ele ficará aguardando até que os nós existentes respondam e, portanto, o cliente ficará bloqueado indefinidamente.

###*Journaling* em *replica sets*
Quando obrigamos o sistema de *journaling* a retornar se a escrita foi comitada em um *replica set*, é necessário somente que o nó primário finalize o *commit* para retornar o sucesso na operação, não importando a quantidade de nós especificados em **w**.

###Opção de *timeout* de escrita
O *driver* ou o Mongo shell podem ficar bloqueados por não terem recebido a resposta de um servidor, quando a opção **w** for maior que 0 (zero). Para evitar que isso aconteça, podemos setar um *timeout* a fim de que seja retornado um erro, caso o tempo de resposta do servidor extrapole o tempo máximo.

##Velocidade de inserção
As configurações do *write concern* podem ter um impacto muito grande na velocidade da aplicação. Rode os exemplos abaixo para ver a diferença.

Em uma seção do `mongo` rode o seguinte comando:

{% highlight javascript linenos=table %}
for(i=0; i<1000000; i++) {
  db.test.insert({nome: 'Mongo'}, {
    writeConcern: {w: 1, j: 1}
  })
}
{% endhighlight %}

Em uma outra seção do mongo, (enquanto ainda estiver inserindo os documentos do exemplo acima), rode o comando `mongostat` e verifique a coluna **insert**. Em um Core i5, a média mostrada nessa coluna foi de 28 inserções por segundo.

Agora feche a seção do terminal que está inserindo os documentos com `ctrl + c` e execute esse outro comando:

{% highlight javascript linenos=table %}
for(i=0; i<1000000; i++) {
  db.test.insert({nome: 'Mongo'}, {
    writeConcern: {w: 0, j: 0}
  })
}
{% endhighlight %}

No mesmo computador, o número de inserções usando escritas *unacknowledged* passou para em torno de 3200 inserções por segundo, ou seja, as inserções rodaram 114 vezes mais rápido!

Agora... 3200 inserções é bem pouco, não? Podemos melhorar isso usando o método Bulk() para inserções em lote:

{% highlight javascript linenos=table %}
var bulk = db.test.initializeUnorderedBulkOp();

for(i=0; i<1000000; i++) {
  bulk.insert({nome: 'Mongo'})
}

bulk.execute({w: 0, j: 0});
{% endhighlight %}

Usando esse método, o número de inserções passou para, em média, 46000 inserções por segundo.

<hr/>

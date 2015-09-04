---
layout: post
title:  "Workshop sobre MongoDB no grupo Coders"
description: "Workshop sobre o básico de MongoDB feito para o grupo Coders da UTFPR"
date:   2015-08-31 00:00:00
categories: ['mongodb']
---


**iniciando o servidor**
{% highlight javascript linenos=table %}
mongod
{% endhighlight %}


**iniciando o cliente**
{% highlight javascript linenos=table %}
mongo
{% endhighlight %}

**mostrando todos os bancos**
{% highlight javascript linenos=table %}
show dbs
{% endhighlight %}

**usando um banco que ainda não existe**
{% highlight javascript linenos=table %}
use agenda
{% endhighlight %}

**variável global referenciando o banco**
{% highlight javascript linenos=table %}
db
{% endhighlight %}






**inserindo o primeiro registro na agenda**
{% highlight javascript linenos=table %}
db.agenda.insert({
	nome: 'Fulano',
	idade: 20
})
{% endhighlight %}

**mostrando o registro salvo**
{% highlight javascript linenos=table %}
db.agenda.findOne()
{% endhighlight %}

**inserindo mais um registro na agenda**
{% highlight javascript linenos=table %}
db.agenda.insert({
	nome: 'Sicrano',
	idade: 25,
	empresa: 'UTFPR'
})
{% endhighlight %}







**mostrando todos os registros da agenda**
{% highlight javascript linenos=table %}
db.agenda.find()
{% endhighlight %}

**melhorando a visualização dos redultados**
{% highlight javascript linenos=table %}
db.agenda.find().pretty()
{% endhighlight %}

**buscando por um nome específico**
{% highlight javascript linenos=table %}
db.agenda.find({nome: 'Fulano'})
{% endhighlight %}

**buscando por um nome e idade específica**
{% highlight javascript linenos=table %}
db.agenda.find({
	nome: 'Sicrano',
	idade: 25
})
{% endhighlight %}






**help me!**
{% highlight javascript linenos=table %}
help
db.help()
db.agenda.help()
db.agenda.find().help()
{% endhighlight %}






**limitando os campos retornados (projeção)**
{% highlight javascript linenos=table %}
db.agenda.find({}, {nome: 1})
{% endhighlight %}

**removendo o _id da projeção**
{% highlight javascript linenos=table %}
db.agenda.find({}, {nome: 1, _id: 0})
{% endhighlight %}








**inserindo um registro com documento encadeado**
{% highlight javascript linenos=table %}
db.agenda.insert({
	nome: 'Beltrano',
	empresa: {
		nome: 'UTFPR',
		departamento: 'DAINF'
	}
})
{% endhighlight %}

**buscando dentro de um documento encadeado**
{% highlight javascript linenos=table %}
db.agenda.find({'empresa.departamento': 'DAINF'})
{% endhighlight %}

**inserindo um registro com array**
{% highlight javascript linenos=table %}
db.agenda.insert({
	nome: 'Varano',
	idade: 15,
	grupo: ['amigos', 'trabalho']
})
{% endhighlight %}

**buscando dentro de um array**
{% highlight javascript linenos=table %}
db.agenda.find({grupo: 'amigos'})
{% endhighlight %}

**inserindo um registro com array de subdocumentos**
{% highlight javascript linenos=table %}
db.agenda.insert({
	nome: 'Varano',
	idade: 15,
	empresas: [
		{nome: 'UTFPR', departamento: 'DAINF'},
		{nome: 'UTFPR', departamento: 'DADIN'},
	]
})
{% endhighlight %}

**buscando dentro de um array de subdocumentos**
{% highlight javascript linenos=table %}
db.agenda.find({'empresas.nome': 'UTFPR'})
{% endhighlight %}








**usando operadores de comparação**
{% highlight javascript linenos=table %}
db.agenda.find({idade: {$gt: 17}})
{% endhighlight %}

**usando operadores lógicos**
{% highlight javascript linenos=table %}
db.agenda.find({
	$or: [
		{idade: 30},
		{nome: 'Sicrano'}
	]
})
{% endhighlight %}

**combinando operadores**
{% highlight javascript linenos=table %}
db.agenda.find({
	$or: [
		{idade: {$gt:20}},
		{nome: 'Fulano'}
	]
})
{% endhighlight %}





**ordenando os resultados**
{% highlight javascript linenos=table %}
db.agenda.find({}).sort({idade: 1})
{% endhighlight %}






**modificando um documento**
{% highlight javascript linenos=table %}
db.agenda.update(
	{nome: 'Fulano'}, 
	{nome: 'Fulano Modificado', idade: 30}
)
{% endhighlight %}

**modificando apenas um campo de um documento**
{% highlight javascript linenos=table %}
db.agenda.update(
	{nome: 'Fulano Modificado'}, 
	{$set: {nome: 'Fulano'}}
)
{% endhighlight %}

**adicionando um documento caso o critério não ache nenhum documento**
{% highlight javascript linenos=table %}
db.agenda.update(
	{nome: 'Tadeu'}, 
	{nome: 'Fulaninho'}, 
	{upsert: true}
)
{% endhighlight %}

**alternativa para adicionar um documento caso não exista**
{% highlight javascript linenos=table %}
db.agenda.save({_id: 10, nome: 'Tadeu'})
{% endhighlight %}

>>*OBS: repita o comando acima e veja o resultado




**removendo um documentos**
{% highlight javascript linenos=table %}
db.agenda.remove({nome: 'Fulano'})
{% endhighlight %}






**executando javascript arbitrário**
{% highlight javascript linenos=table %}
for (i=0; i<100; i++) {
	db.agenda.insert({nome: 'Fulano ' + i});
}
{% endhighlight %}

**procurando os registros inseridos com RegEx**
{% highlight javascript linenos=table %}
db.agenda.find({nome: /^Fulano/})
{% endhighlight %}

**iterando o cursor**
{% highlight javascript linenos=table %}
it
{% endhighlight %}
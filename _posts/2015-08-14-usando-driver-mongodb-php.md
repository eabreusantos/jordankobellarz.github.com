---
layout: post
title:  "Usando o driver do MongoDB com PHP"
description: "..."
date:   2015-08-14 00:00:00
categories: ['php', 'mongodb']
---


Conhecendo-se como usar o MongoDB pelo Mongo shell, podemos partir para uma aventura mais interessante através dos *drivers* oficiais para várias linguagens de programação. Nesse *post* será mostrado como realizar as principais operações através do *driver* feito para PHP.

##Instalando o *driver* do MongoDB para PHP
Para instalar o *driver* é necessário ter o PEAR instalado para que possamos baixá-lo usando o PECL. Para instalá-lo é fácil:

{% highlight sh linenos=table %}
sudo apt-get install php5-dev php5-cli php-pear
{% endhighlight %}

Depois de instalar o PEAR, você poderá baixar o driver oficial do MongoDB através do repositório do PECL:

{% highlight sh linenos=table %}
sudo pecl install mongo
{% endhighlight %}

Só falta mais um passo: ativar o *driver* globalmente na sua instalação do PHP através do `php.ini`. Use o comando `find / -name 'php.ini'` em seu terminal, caso não saiba onde o arquivo `php.ini` está. Depois de encontrá-lo, abra-o em um editor e adicione a seguinte linha (de preferência na seção de *extensions*):

{% highlight sh linenos=table %}
extension=mongo.so
{% endhighlight %}

Reinicie o servidor:

{% highlight sh linenos=table %}
# para nginx
sudo service nginx restart

#para Apache
sudo service apache2 restart
{% endhighlight %}

Crie um arquivo `index.php` na raiz da pasta pública do seu servidor com o seguinte conteúdo:

{% highlight php  startinline=true linenos=table %}
<?php
phpinfo();
{% endhighlight %}

Abra o browser em `http://localhost/` e verifique na página que abriu se há uma seção mostrando detalhes sobre o *driver* do MongoDB. Se tudo ocorreu corretamente, corra para o abraço!

##Conectando o MongoDB à sua aplicação PHP
Primeiro inicie seu servidor MongoDB através do terminal com o comando `mongod`. Ele ficará aguardando conexões na porta 27017 (padrão).

Crie um *script* PHP com o seguinte conteúdo (vou omitir a tag de abertura do PHP, assumindo que você já sabe que ela existe):

{% highlight php startinline=true linenos=table %}
$conn = new MongoClient();
{% endhighlight %}

Agora a variável `$conn` possui uma referência para a conexão com o servidor do MongoDB rodando em sua máquina na porta padrão. Caso queira especificar outro caminho para o servidor, passe ele como parâmetro no método `MongoClient`:

{% highlight php startinline=true linenos=table %}
$conn = new MongoClient('192.168.25.2:27018');
{% endhighlight %}

A partir da variável `$conn` podemos acessar o MongoDB de forma similar ao acesso de uma base de dados relacional usando um ORM. Para escolher uma banco de dados, simplesmente apontamos para ela como se fosse um atributo da classe `MongoClient` que acabamos de instanciar:

{% highlight php startinline=true linenos=table %}
$conn = new MongoClient();

// acessando a base de dados "blog"
$conn->blog;

// de forma alternativa
$conn->selectDB("blog");
{% endhighlight %}

A primeira forma `$conn->blog` é a mais usual para acessar um banco, contudo ela é inviável para o caso de um banco chamado `erp.blog`, por exemplo, daí o método `$conn->selectDB()` é útil.

De forma parecida com que acessamos um banco podemos acessar também uma coleção:

{% highlight php startinline=true linenos=table %}
$conn = new MongoClient();

// acessando a coleção "posts"
$conn->blog->posts;

// de forma alternativa
$conn->selectDB('blog')->selectCollection('posts')

// ou também
$conn->blog->selectCollection('posts')
{% endhighlight %}

Se lembra da ideia de que o MongoDB é *schemaless*? Sabendo disso, quando acessamos o nome de um banco ou de uma coleção inexistente, estamos de fato criando-as de forma implícita. Essa é uma das maiores dores de cabeça de quem está começando a usar os *drivers* do MongoDB sem conhecê-lo.

Para exemplificar esse problema veja o código abaixo:

{% highlight php startinline=true linenos=table %}
$conn->blogui->posts->insert(array());
{% endhighlight %}

Viu o problema? Nesse código o *driver* solicitou ao servidor que fosse criado o banco chamado "blogui" para inserir um novo documento ao invés de usar o banco "blog" que já tinhamos criado. Por isso esteja sempre atento ao seu código!

##Representação dos documentos no PHP
Um documento no MongoDB é um objeto binário BSON, que pode ser visto em alto nível como um objeto JSON. Os objetos JSON no PHP são mapeados para arrays. Por exemplo, o objeto JSON abaixo:

{% highlight javascript linenos=table %}
{
  "autor": "Jordan Kobellarz",
  "titulo": "Aventuras com o MongoDB",
  "conteudo": "bla bla bla",
  "tags": ["mongodb", "nosql", "documentos"],
  "comentarios": [
    {
      "autor":"Tobias",
      "comentario": "Eu adorei o artigo. Parabéns!"
    },
    {
      "autor": "Emanuel",
      "comentario": "Estou usando o MongoDB em produção, muito bom!"
    }
  ]
}
{% endhighlight %}

A mesma estrutura do objeto do exemplo acima ficaria assim no PHP:

{% highlight php startinline=true linenos=table %}
$doc = [
  "autor" => "Jordan Kobellarz",
  "titulo" => "Aventuras com o MongoDB",
  "conteudo" => "bla bla bla",
  "tags" => ["mongodb", "nosql", "documentos"],
  "comentarios" => [
    [
      "autor" => "Tobias",
      "comentario" => "Eu adorei o artigo. Parabéns!"
    ],
    [
      "autor" => "Emanuel",
      "comentario" => "Estou usando o MongoDB em produção, muito bom!"
    ]
  ]
];
{% endhighlight %}

Lembre-se que a criação de *arrays* no PHP é feita de maneira implícita usando apenas colchetes somente para as versões do PHP acima da 5.4. Caso a versão do PHP que você esteja usando seja menor que 5.4, use a notação `array()`.

##Operações CRUD
Os mesmos métodos que são acessados no topo de uma coleção através do terminal também estão presentes para acesso através do *driver* PHP:

{% highlight php startinline=true linenos=table %}
$conn = new MongoClient();
$db = $conn->blog;

$doc = [
  "nome" => "Jordan",
  "titulo" => "Hello World!"
];

// inserindo um documento
$db->posts->insert($doc);

// procurando um documento
$db->posts->findOne(['nome' => 'Jordan']);

// removendo um documento
$db->posts->remove(['titulo' => 'Hello World!']);
{% endhighlight %}

Os métodos usados no topo de uma coleção pelo *driver* têm a mesma quantidade de parâmetros que os métodos acessados pelo terminal. Por exemplo, para projetar os campos que queremos retornar no metodo `find`:

{% highlight php startinline=true linenos=table %}
$query = ['nome' => 'MongoDB'];
$projecao = ['_id' => 0];

$db->posts->findOne($query, $projecao);
{% endhighlight %}

É possível também encadear métodos no topo do método `find`, quando queremos fazer uma ordenação, limitar ou contar os resultados, por exemplo:

{% highlight php startinline=true linenos=table %}
$db->posts->find([])->sort(['nome' => 1])->count();
{% endhighlight %}

Os modificadores de consulta iniciados com `$` também podem ser passados da mesma forma que o usamos no Mongo shell, mas aqui devemos ter o cuidado de passá-los com aspas simples, caso contrário o PHP interpretará como uma variável:

{% highlight php startinline=true linenos=table %}
$db->posts->find([
  "nome" => [
    '$ne' => 'Mongo'
  ]
]);
{% endhighlight %}

##Mapeando SQL para os comandos do MongoDB

No manual do *driver* do MongoDB para PHP encontramos um mapeamento de comandos SQL para comandos do MongoDB. Veja abaixo alguns exemplos:

{% highlight php startinline=true linenos=table %}
// INSERT INTO USERS VALUES(1,1)
$db->users->insert(["a" => 1, "b" => 1]);

// SELECT a,b FROM users
$db->users->find([], ["a" => 1, "b" => 1]);

//SELECT * FROM users WHERE age=33
$db->users->find(["age" => 33]);

// SELECT a,b FROM users WHERE age=33
$db->users->find(["age" => 33], ["a" => 1, "b" => 1]);

// SELECT a,b FROM users WHERE age=33 ORDER BY name
$db->users->find(["age" => 33], ["a" => 1, "b" => 1])->sort(["name" => 1]);

// SELECT * FROM users WHERE age>33
$db->users->find(["age" => ['$gt' => 33]]);

// SELECT * FROM users WHERE age<33
$db->users->find(["age" => ['$lt' => 33]]);

// SELECT * FROM users WHERE name LIKE "%Joe%"
$db->users->find(["name" => new MongoRegex("/Joe/")]);

// SELECT * FROM users WHERE name LIKE "Joe%"
$db->users->find(["name" => new MongoRegex("/^Joe/")]);

// SELECT * FROM users WHERE age>33 AND age<=40
$db->users->find(["age" => ['$gt' => 33, '$lte' => 40]]);

// SELECT * FROM users ORDER BY name DESC
$db->users->find()->sort(["name" => -1]);

// CREATE INDEX myindexname ON users(name)
$db->users->ensureIndex(["name" => 1]);

// CREATE INDEX myindexname ON users(name,ts DESC)
$db->users->ensureIndex(["name" => 1, "ts" => -1]);

// SELECT * FROM users WHERE a=1 and b='q'
$db->users->find(["a" => 1, "b" => "q"]);

// SELECT * FROM users LIMIT 20, 10
$db->users->find()->limit(10)->skip(20);

// SELECT * FROM users WHERE a=1 or b=2
$db->users->find(['$or' => [["a" => 1], ["b" => 2]]]);

// SELECT * FROM users LIMIT 1
$db->users->find()->limit(1);

// EXPLAIN SELECT * FROM users WHERE z=3
$db->users->find(["z" => 3])->explain()

// SELECT DISTINCT last_name FROM users
$db->command(["distinct" => "users", "key" => "last_name"]);

// SELECT COUNT(*) FROM users
$db->users->count();

// SELECT COUNT(age) from users
$db->users->find(["age" => ['$exists' => true]])->count();

// UPDATE users SET a=1 WHERE b='q'
$db->users->update(["b" => "q"], ['$set' => ["a" => 1]]);

// UPDATE users SET a=a+2 WHERE b='q'
$db->users->update(["b" => "q"], ['$inc' => ["a" => 2]]);

// DELETE FROM users WHERE z="abc"
$db->users->remove(["z" => "abc"]);
{% endhighlight %}

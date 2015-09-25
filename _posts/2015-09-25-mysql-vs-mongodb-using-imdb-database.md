---
layout: post
title:  "MySQL vs MongoDB: um exemplo usando a base de dados do IMDB"
description: ""
date:   2015-09-07 00:00:00
categories: ['mongodb', 'mysql']
---

O IMDB é um repositório de filmes gigantesco. Surgiu em 1990 e hoje é a maior e mais popular fonte de dados sobre filmes, TV e dados sobre celebridades. A parte mais interessante disso é que disponibilizam alguns arquivos de sua base de dados gratuitamente para uso pessoal. Essa é uma ótima oportunidade para usá-los para explorar tecnologias em bancos de dados.

Nesse artigo mostrarei como preparar o ambiente, instalar a base do IMDB no MySQL, tranferir o conjunto de dados para o MongoDB e depois comparar o MySQL e o MongoDB frente a frente para um conjunto de consultas mais usado para essa base de dados, a fim de aferir a eficiência das duas tecnologias de armazenamento. 

Você aprenderá:

* ter paciência
* transferir uma base relacional para o MongoDB
* modelar um banco de dados eficiente com o MongoDB
* comparar os pros e contras de um banco relacional para o MongoDB

##Instalação da base de dados do IMDB

Antes de começar, vamos para a parte chata: instalar a base de dados do IMDB no MySQL. Usaremos o projeto JMDB, que converte os arquivos em texto plano do IMDB para um formato aplicável para o MySQL ou PostgreSQL. 

**#0: Dependências**

Para rodar todo os exemplos a seguir, você deverá ter todos os requisitos:

* Linux
* Java JRE
* MySQL ou PostgreSQL
* MongoDB >= 2.6
* ~ 5GB sobrando no HD
* 5h sobrando para baixar e instalar toda a base de dados

**#1: Baixando os arquivos em texto plano do IMDB:**

O IMDB fornece gratuitamente para uso não-comercial alguns arquivos em texto plano que contém boa parte da sua base de dados. Esses arquivos podem ser baixados com o seguinte comando:

{% highlight sh linenos=table %}
wget -r -np -l 1 -A gz ftp://ftp.fu-berlin.de/pub/misc/movies/database/
{% endhighlight %}

>> eu adoraria distribuir um arquivo .sql via torrent, contudo as políticas do IMDB não permitem que seus arquivos sejam distribuídos em um formato que não seja igual ao .txt deles

Esse código baixa os arquivos link por link do [mirror ftp do IMDB](ftp://ftp.fu-berlin.de/pub/misc/movies/database/). Vai demorar bastante, pois as quotas de uso do FTP deles são bem limitadas. Ao todo vai dar aproximadamente 1.5GB de arquivos.

**#2: Baixando o projeto JMDB:**

O JMDB é um projeto em Java criado para mapear os arquivos .txt do IMDB para uma base de dados MySQL ou PostgreSQL. Ele também possui uma interface amigável para realizar as buscas, permitindo inclusive full-text search.

Baixe o [projeto JMDB por aqui](http://www.juergen-ulbts.de/content/download/project/jmdb/Java_Movie_Database_V1-40pre2p_2014-02-12_gen.zip). Depois descompacte em uma pasta pública de seu computador. 

**#3: Preparando o JMDB:**

Entre na pasta do projeto do JMDB e procure por uma pasta chamada **MovieCollection**. Remova tudo o que tiver dentro dela e depois copie os arquivos de texto plano que você baixou no **passo 1** para esta pasta.

Na pasta do JMDB você encontrará também um arquivo com o nome **startlinux.sh**. Tive que editar ele, pois estava dando problemas em minha distribuição Debian. Então abra ele com um editor de texto e substitua tudo por:

<pre>
#!/bin/bash
java -Xmx850M -classpath .:jmdb.jar:postgresql-8.4-701.jdbc3.jar:mysql-connector-java-5.1.11-bin.jar:./plugins/export/itext-1.4.3.jar jmdb.base.JMDBMain debugmode=10
</pre>

**#4: Executando o JMDB:**

Agora que seu ambiente está preparado, entre na pasta do JMDB pelo terminal e execute o arquivo **startlinux.sh**:

{% highlight sh linenos=table %}
sh startlinux.sh
{% endhighlight %}

A tela de configuração será aberta. Acesse a aba **DB Setup** e forneça os dados para conexão com o banco de dados que você vai usar. Lembre-se que é possível usar tanto o MySQL quanto o PostgreSQL. Ao fim, aperte em **Test Connection** e verifique se os parâmetros que você passou estão corretos. Pode ser que dê um erro, pois a base de dados **jmdb** ainda não foi criada -> não tem problema, ela será criada automaticamente depois. 

Agora vá para a aba **IMDB-Import** e em **Import Path** selecione a pasta **MovieCollection**, caso ela não venha configurada por padrão. Se der certo, a coluna **filecheck** será marcada com um **ok** para todos os arquivos. 

Na mesma aba, desmarque a coluna **import** apenas dos arquivos **german-aka-titles** e **italian-aka-titles**. Caso não os desmarque, poderá ter problemas no decorrer da instação da base de dados.

Basicamente é isso. Para finalizar clique em **save** no canto inferior esquerdo. A partir de agora você poderá fazer a importação dos arquivos .txt para a sua base de dados MySQL ou PostgreSQL. Nessa tela clique em **Create Database** e aguarde até tudo ser concluído. Leva aproximadamente 4h para a base ser instalada por completo. 

##Explorando o banco do IMDB no MySQL

continua...
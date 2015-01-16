---
layout: post
title:  "Como criar um blog grátis com Jekyll e GitHub Pages em 10 minutos"
description: "O Jekyl é um criador de sites estáticos, capaz de rodar nos servidores do GitHub Pages, que permite criar páginas web gratuitamente."
date:   2015-01-14 15:00:00
categories: ['jekyll','GitHub']
---

Para hospedar esse site usei o [GitHub Pages](https://pages.github.com/) e para gerar a estrutura 
em forma de blog usei o [Jekyll](http://jekyllrb.com/), um gerador de sites estáticos, com as mesmas funcionalidades de um CMS.

**O que é o Jekyll?**

Jekyll é uma ferramenta simples que transforma páginas escritas em HTML, Textile ou Markdown em um
blog estruturado sem a necessidade de um banco de dados ou código dinâmico no *backend*.

**O que é o GitHub Pages?**

GitHub Pages é uma hospedagem de sites estáticos disponibilizada gratuitamente pelo GitHub.

##O que é preciso?
* PC com Linux ou MacOS
* conta no [GitHub](https://github.com/)
* 10 minutos sobrando
* editor de texto para HTML como [Brackets](http://brackets.io/), [Atom](https://atom.io/), [Sublime](http://www.sublimetext.com/), ...

##1. Crie um repositório no GitHub
* entre em sua conta do [GitHub](https://github.com/)
* crie um [novo repositório](https://github.com/new) exatamente com o nome **username.github.io**,
no qual **username** é o seu nome de usuário (não funciona se você não colocar exatamente seu nome de usuário).
* abra o terminal `ctrl + alt + t` e clone o repositório
{% highlight bash linenos=table %}
$ git clone https://github.com/username/username.github.io
{% endhighlight %}

##2. Escolha um tema para seu website <small>[opcional]</small>
* entre no [respositório de temas do Jekyll](http://jekyllthemes.org/) e escolha o seu favorito.
Recomendo o tema [Kasper](https://github.com/rosario/kasper), que é o que eu uso nesse site.
* clone o tema no repositório que acabou de criar (não se esqueça que **username** é o seu nome de
usuário no GitHub).
{% highlight bash linenos=table %}
$ git clone https://github.com/rosario/kasper username.github.io
$ cd username.github.io
{% endhighlight %}

##3. Suba os arquivos para seu respositório no GitHub
ele vai pedir seu nome de usuário e sua senha
{% highlight bash linenos=table %}
$ git add --all
$ git commit -m "novo post: Hello World!"
$ git pull origin master
{% endhighlight %}

##4. Acesse seu novo site
A URL é exatamente o nome do repositório que você criou. Então, acesse:
**username.github.io**. Caso ainda não esteja disponível, aguarde alguns minutos.

##5. E o Jekyll?
Agora que você tem um site funcionando, é preciso aprender a criar novos posts. É aí que entra o Jekyll.

Para criar um novo post é necessário criar um arquivo dentro da pasta `_posts` com
nome obedecendo o seguinte formato: `YYYY-MM-DD-titulo-do-post.html`, por exemplo, 
`2015-01-14-hello-world.html`. Se não for obedecido, o Jekyll não saberá como
organizar seus posts.

###Mãos à obra:

* instale o Jekyll 
{% highlight bash linenos=table %}
$ gem install jekyll
{% endhighlight %}

* crie um arquivo chamado `2015-01-14-hello-world.html`
{% highlight bash linenos=table %}
$ touch \_posts\2015-01-14-hello-world.html
{% endhighlight %}

* adicione as seguintes linhas no arquivo criado: 
{% highlight html linenos=table %}
---
layout: post
title:  Hello World!
date:   2015-01-14 15:00:00
---
<h2>Jekyllti</h2>
<p>Y Love You World s3</p>
{% endhighlight %}
* para visualizar como sua página ficará online, inicie um servidor Jekyll 
{% highlight bash linenos=table %}
$ jekyll serve
{% endhighlight %}

* acesse [localhost:4000](http://localhost:4000)

##6. Conquiste o mundo!
repita o passo #3 e veja seu novo post online. Caso queira aprender mais sobre o Jekyll, acesse a sua 
[documentação oficial](http://jekyllrb.com/docs/home/). Ela é bem completa e fácil de
compreender. Vale a pena dar uma olhada.

Comente abaixo sua experiência com o Jekyll e com o tutorial. 
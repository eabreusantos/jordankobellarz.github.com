---
layout: post
title:  "Hello World!"
date:   2015-01-12 12:25:00
categories: ['jordan', 'desenvolvedor']
---

Como todo site pessoal, é de costume darmos as boas vindas ao público e aproveitar
para vender as ideias que irei propor aqui.

{% highlight javascript linenos=table %}
console.log('Hello World!');
{% endhighlight %}

{% highlight php linenos=table %}
<?php echo 'Hello, world!'; ?>
{% endhighlight %}

Gosto de *PHP* e *javascript*, mas não me limito a essas linguagens, pois, aliás, 
a dinâmica das tecnologias web emergentes faz com que sejamos agnósticos.

Comecei a escrever pela necessidade de espalhar conhecimento em língua portuguesa para
a área de desenvolvimento e, também, para melhorar minhas habilidades pessoais, pois consigo 
absorver conteúdos complexos com maior facilidade quando escrevo.

> reter conhecimento em meio a tantas tecnologias requer um esforço cada vez maior. Melhor
começar agora do que tarde!

## Apenas sobre desenvolvimento?

Não. Sobre tudo! Até sobre minha vida, mas sempre voltado aos desenvolvedores.



1. Item 1
  1. A corollary to the above item.
  2. Yet another point to consider.
2. Item 2
  * A corollary that does not need to be ordered.
    * This is indented four spaces, because it's two spaces further than the item above.
    * You might want to consider making a new list.
3. Item 3

{% highlight php startinline linenos=table %}
// comment out the following two lines when deployed to production
defined('YII_DEBUG') or define('YII_DEBUG', true);
defined('YII_ENV') or define('YII_ENV', 'dev');

require(__DIR__ . '/../vendor/autoload.php');
require(__DIR__ . '/../vendor/yiisoft/yii2/Yii.php');

$config = require(__DIR__ . '/../config/web.php');

(new yii\web\Application($config))->run();

{% endhighlight %}

Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll's GitHub repo][jekyll-gh].

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com

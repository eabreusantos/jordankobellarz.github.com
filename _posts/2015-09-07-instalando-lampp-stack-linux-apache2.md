---
layout: post
title:  "Intalando o LAMP Stack: Linux, Apache, MongoDB e PHP5"
description: ""
date:   2015-09-07 00:00:00
categories: ['linux', 'apache', 'mongodb', 'php5']
---

Hora de mudar um pouco seu LAMP Stack: trocando o M para MongoDB. Esse totorial mostrará como instalar o servidor Apache 2 para rodar PHP5 junto com MongoDB no Debian ou Ubuntu.

**1 -Atualize seus pacotes:**

{% highlight sh linenos=table %}
sudo apt-get update && apt-get upgrade
{% endhighlight %}

**2 - Instale o servidor Apache:**

{% highlight sh linenos=table %}
sudo apt-get install apache2 libapache2-mod-php5
{% endhighlight %}

**3 - Instale o MongoDB:**

Cada distribuição e versão do Debian e do Ubuntu tem um modo de instalação diferente. Veja as [intruções de instalação do MongoDB](http://docs.mongodb.org/master/administration/install-on-linux/).

**4 - Instale o PHP5 e as extensões necessárias:**

Além do PHP5 é necessário ter instalado o PEAR, php5-dev e o php5-cli para poder adicionar a extensão do MongoDB através do repositório PECL.

>>Não instale o driver através do pacote `php5-mongo`, sob risco de obter uma versão desatualizada do driver.

{% highlight sh linenos=table %}
sudo apt-get install php5 php5-dev php5-cli php5-pear
{% endhighlight %}

**5 - Instale a extensão do MongoDB através do PEAR:**

{% highlight sh linenos=table %}
sudo pecl install mongo
{% endhighlight %}

**6 - Adicione o MongoDB ao php.ini do Apache:**

{% highlight sh linenos=table %}
echo 'extension=mongo.so' | sudo tee --append /etc/php5/apache2/php.ini
{% endhighlight %}

**7 - Reinicie o servidor Apache**

{% highlight sh linenos=table %}
sudo service apache2 restart
{% endhighlight %}

**8 - Crie uma pasta para servir seus projetos e adicione um link para ela:**

{% highlight sh linenos=table %}
mkdir -p ~/www
cd /var/www && sudo rm -r html && sudo ln -s ~/www html
{% endhighlight %}

**10 - Verifique se está tudo certo:**

Crie um arquivo chamado `index.php` e chame a função `phpinfo` para verificar se a extensão do MongoDB foi instalada corretamente:

{% highlight sh linenos=table %}
cd ~/www && touch index.php && echo "<?php phpinfo();" >> index.php
{% endhighlight %}

Abra o seu navegador em `http://localhost` e procure com `ctrl + f` a extensão `mongo`.

Pronto. É só isso.
---
layout: post
title:  "Configurando um server block no NGINX com PHP5-FPM"
description: "..."
date:   2015-08-13 00:00:00
categories: ['php', 'nginx']
---

No NGIX, um *server block* é exatamente igual a um VHOST no servidor Apache. Vamos direto ao assunto:


###1 - Instalação NGINX e PHP
Primeiro instalamos o PHP5-FPM e o servidor NGINX:

{% highlight sh linenos=table %}
sudo apt-get install php5-fpm nginx
{% endhighlight %}

###2 - Configurando *server blocks* do NGINX
Após terminar a instalação, acesse a pasta onde as configurações dos *server blocks* do NGINX ficam guardadas:

{% highlight sh linenos=table %}
cd /etc/nginx/sites-available
{% endhighlight %}

Dentro dessa pasta há um arquivo `default`, que contém as configurações padrão. Crie uma cópia desse arquivo com o nome que você quiser (de preferência que seja parecido com a URL do seu *server block* para manter um padrão):

{% highlight sh linenos=table %}
sudo cp default meusite.com
{% endhighlight %}

Agora abra o arquivo copiado com seu editor de texto preferido e remova os comentários `#` das seguintes linhas:

{% highlight sh linenos=table %}
location ~ \.php$ {
	fastcgi_split_path_info ^(.+\.php)(/.+)$;
	# NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini

	# With php5-cgi alone:
#	fastcgi_pass 127.0.0.1:9000;
	# With php5-fpm:
	fastcgi_pass unix:/var/run/php5-fpm.sock;
	fastcgi_index index.php;
	include fastcgi_params;
}
{% endhighlight %}

Agora, no mesmo arquivo, configure as variáveis `root` e `server_name` como achar melhor. `root` é a pasta a partir da qual os arquivos serão servidos (vamos usar `/var/www/meusite.com/public`) e `server_name` é a URL para acessar seu servidor (vamos usar `meusite.com`, que permitirá acesso pela URL `http://meusite.com`):

{% highlight sh linenos=table %}
root /var/www/meusite.com/public;

# ...

server_name meusite.com;
{% endhighlight %}

Salve o arquivo de configuração.

###3 - Habilitando o arquivo de configuração

É necessário que esse arquivo de configuração também esteja disponível dentro da pasta `sites-enabled`, que fica no mesmo nível que a pasta `sites-available`. Para isso, criamos um link simbólico para não replicar o documento de configuração já criado:

{% highlight sh linenos=table %}
cd /etc/nginx/sites-enabled
sudo ln -s ../sites-available/meusite.com
{% endhighlight %}

###4 - Criando a pasta para servir os arquivos

O próximo passo é criar a pasta de onde os arquivos serão servidos. Use `mkdir -p` para que os arquivos pai também sejam criados:

{% highlight sh linenos=table %}
sudo mkdir -p /var/www/meusite.com/public
{% endhighlight %}

###5 - Criando uma entrada nos *hosts* do Linux

Abra o arquivo `hosts` do Linux, que fica dentro da pasta `/etc`, e adicione mais uma linha com os valores `127.0.0.1 meusite.com`.

###6 - Finalizando

Reinicie o serviço do PHP-FPM e o servidor NGINX:

{% highlight sh linenos=table %}
sudo service nginx restart
sudo service php5-fpm restart
{% endhighlight %}

Pronto! Agora você pode acessar seu novo *server block* a partir da URL `http://meusite.com`:

{% highlight sh linenos=table %}
curl meusite.com
{% endhighlight %}

###E se não funcionar?
O primeiro motivo pode ser que a configuração padrão esteja atrapalhando. Tente executar esses comandos:

{% highlight sh linenos=table %}
sudo rm /etc/nginx/sites-enabled/default
sudo service nginx restart
sudo service php5-fpm restart
{% endhighlight %}

Outro motivo pode ser a permissão da pasta do projeto. Tente modificá-la da seguinte forma:

{% highlight sh linenos=table %}
sudo chmod -R /var/www/meusite.com/public
{% endhighlight %}

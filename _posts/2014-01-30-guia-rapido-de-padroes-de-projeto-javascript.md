---
layout: post
title:  "Guia rápido de padrões de projeto Javascript"
description: "Javascript é uma das linguagens mais mais difíceis de ser compreendida, justamente por
ser multiparadigma, ou seja, podemos trabalhar com o paradigma orientado a objetos, procedural e 
imperativo, tudo ao mesmo tempo. Isso pode dificultar a manutenção do código e, também, limitar o
programador iniciante que não conhece as 'armadilhas' dessa linguagem."
date:   2015-01-30 14:00:00
categories: ['javascript']
---

Javascript é uma linguagem difícil de ser compreendida, justamente por
ser multiparadigma, ou seja, podemos trabalhar com o paradigma orientado a objetos, procedural e 
imperativo, tudo ao mesmo tempo. Isso pode dificultar a manutenção do código e, também, limitar o
programador iniciante que não conhece as características dessa linguagem.

Nesse texto serão mostradas algumas formas de organizar o código usando padrões de projeto eficazes, que 
podem melhorar a legibilidade, segurança e, também a eficiência quanto ao uso de memória e processamento
de blocos de código complexos em Javascript.

Todo o conteúdo apresentado nesse artigo foi feito com base no livro 
[Learning Javascript Design Patterns](http://addyosmani.com/resources/essentialjsdesignpatterns/book/), que foi escrito por [@Addy Osmani](https://twitter.com/addyosmani) sob a licença [CC BY-NC-ND 3.0](http://creativecommons.org/licenses/by-nc-nd/3.0/).

# Construtor com prototype
Como mostrado em meu artigo [Coding standard rápido para javascript](http://jordankobellarz.github.io/javascript/2015/01/16/coding-standards-rapido-para-javascript.html), é possível criar um método construtor da seguinte forma:

{% highlight javascript linenos=table %}
function Carro( modelo, ano, km ) {

    this.modelo = modelo;
    this.ano = ano;
    this.km = km;
 
    this.toString = function () {
        return this.modelo + " andou " + this.km + " km desde " + this.ano;
    };
}
 
// instanciando Carro
var opala = new Carro( "Chevrolet Opala", 1979, 130000 );
var belina = new Carro( "Ford Belina", 1982, 200000 );

// chamando o método toString() de cada Carro
console.log( opala.toString() );
console.log( belina.toString() );
{% endhighlight %}

Essa é uma forma muito ineficiente de se criar um método construtor, visto que toda vez que 
`Carro` for instanciado o método `toString()` será replicado, consumindo mais memória. Além do mais,
usando esse padrão será muito mais difícil criar herança para essa classe.

Para evitar esses problemas, todos os objetos do Javascript possuem um objeto chamado **prototype**, que
serve para adicionar **propriedades** e **métodos** a um objeto. No exemplo a seguir, apenas um método
`toString()` será criado e disponibilizado no escopo para toda vez que um novo `Carro` for instanciado:

{% highlight javascript linenos=table %}
function Carro( modelo, ano, km ) {

    this.modelo = modelo;
    this.ano = ano;
    this.km = km;

}

// usando protype para definir um método
Carro.prototype.toString = function () {
    return this.modelo + " andou " + this.km + " km desde " + this.ano;
};

// instanciando Carro
var opala = new Carro( "Chevrolet Opala", 1979, 130000 );
var belina = new Carro( "Ford Belina", 1982, 200000 );

// chamando o método toString() de cada Carro
console.log( opala.toString() );
console.log( belina.toString() );
{% endhighlight %}

# Module pattern
Projetos grandes precisam ser organizados em unidades coesas para manter a organização. Existem várias
formas de se fazer isso, mas a mais aceitável é o **module pattern**.

Basicamente para se criar um módulo precisamos de uma função anônima que é imediatamente executada 
após a sua declaração:

{% highlight javascript linenos=table %}
var nomeDoModulo = (function () {

    /* TODO */

})(); // executa imediatamente após a declaração
{% endhighlight %}

O module pattern usa o conceito de [object literals](http://rmurphey.com/blog/2009/10/15/using-objects-to-organize-your-code/) para organizar o código e, dessa forma, usando [closures](http://en.wikipedia.org/wiki/Closure_%28computer_programming%29) (funções encadeadas) é possível 
criar o encapsulamento de propriedades e métodos.

{% highlight javascript linenos=table %}
var moduloEstacionamento = (function () {

    // objetos privados
    var patio = [];
    var deletaCarro = function (carro) {
        index = patio.indexOf(carro);
        if (index > -1) {
            patio.splice(index, 1);
        }
    };
 
    // objetos públicos
    return {
        lista: patio,
        entrar: function (carro) {
            patio.push(carro);
        },
        sair: function (carro) {
            deletaCarro(carro);
        }
    };
})();
{% endhighlight %}

No module pattern, como mostrado no exemplo acima, deixamos todos os objetos **públicos** dentro 
do `return`, fazendo com que tudo o que esteja fora dele seja **privado**. Além do mais, criamos um 
**alias** para a propriedade privada `patio`, que pode ser acessada publicamente pelo nome `lista`.

Dessa forma podemos usar o módulo `moduloEstacionamento` do seguinte modo:

{% highlight javascript linenos=table %}
belina = {
    "modelo" : "belina",
    "placa" : "abc-1234"
};

// mostra os objetos públicos do moduloEstacionamento
console.log( moduloEstacionamento );

// estaciono a belina
moduloEstacionamento.entrar( belina );

// mostra [{ "modelo" : "belina", "placa" : "abc-1234" }]
console.log( moduloEstacionamento.lista );

// removo a belina
moduloEstacionamento.sair( belina );

// mostra []
console.log( moduloEstacionamento.lista );

// mostra undefined
console.log( moduloEstacionamento.patio );
{% endhighlight %}

##### As desvantagens de se usar o Module Pattern são:

* é um pouco desgastante mudar a privacidade de um objeto
* não é possível realizar testes unitários em objetos privados

#### Module exports (variação do Module Pattern)

Para não deixar o objeto de retorno um emaranhado de objetos encadeados, podemos usar uma variação do
module pattern chamada de **exports**. Estéticamente essa forma é bem mais bonita que a primeira:

{% highlight javascript linenos=table %}
var moduloEstacionamento = (function () {

    // objetos privados
    var export = {};
    var patio = [];
    var deletaCarro = function (carro) {
        index = patio.indexOf(carro);
        if (index > -1) {
            patio.splice(index, 1);
        }
    };
    
    // objetos que serão publicados
    export.lista = patio;
    export.entrar = function (carro) {
        patio.push(carro);
    };
    export.sair = function (carro) {
        deletaCarro(carro);
    };
 
    return export;
})();
{% endhighlight %}

#### Revealing Module Pattern  (variação do Module Pattern)
Essa é outra variação do Module Pattern, que permite maior consistência na sintaxe do módulo, pois
todos os métodos são mantidos privados até o momento em que são explicitamente expostos:

{% highlight javascript linenos=table %}
var moduloContador = (function () {

    // objetos privados
    var contador = 0;
    function incrementa(){
        contador ++;
    }
    
    // aponta os objetos públicos para os objetos privados
    function incrementa(){
        incrementa();
    }
    function getContador(){
        return contador;
    }
    
    // publica os métodos que devem ser expostos publicamente
    return {
        add: incrementa,
        contador: getContador
    };
})();
{% endhighlight %}

#### Mixins 

Podemos passar globais como parâmetros de um módulo para facilitar o uso delas dentro de seu escopo.
Um bom exemplo é passarmos o objeto `JQuery` para usarmos dentro do módulo sob nome `$`:

{% highlight javascript linenos=table %}
var moduloPaint = (function ($) {

    return {
        pintar: function (e, cor) {
            $(e).css('background-color', cor);
        }
    }
    
})(Jquery);

// muda o background de H1 para preto
moduloPaint.pintar('h1', '#000');
{% endhighlight %}

# Singleton pattern
Singletons são são classes que só podem ser instanciadas uma vez, ou seja, a classe retorna uma
referência ao seu próprio objeto. Dessa forma toda vez que formos usar essa classe, estaremos usando o 
mesmo objeto.

Esse pattern possui a característica de **inicialização preguiçosa**, ou seja, o singleton somente estará
disponível em memória a partir do momento de sua inicialização.

Ele pode ser útil quando queremos manter um único ponto de acesso a uma funcionalidade do sistema, por
exemplo, o acesso a configuração do programa ou aos recursos de uma API RESTFul.

Nesse exemplo é mostrado como criar um singleton usando o module pattern:

{% highlight javascript linenos=table %}
var singletonEstacionamento = (function () {
 
    // armazena uma referência ao singleton
    var instance;
 
    function init(){
    
        // métodos e propriedades do singleton vão aqui
 
        // objetos privados
        var patio = [];
        var deletaCarro = function (carro) {
            index = patio.indexOf(carro);
            if (index > -1) {
                patio.splice(index, 1);
            }
        };

        // objetos públicos
        return {
            lista: patio,
            entrar: function (carro) {
                patio.push(carro);
            },
            sair: function (carro) {
                deletaCarro(carro);
            }
        };
    };

    // retorna sempre a mesma instância dessa classe
    return {
 
        // se a instância existir, retorna ela, senão cria uma nova
        getInstance: function () {
 
            if ( !instance ) {
                instance = init();
            }
 
            return instance;
        }
    };
})();
{% endhighlight %}

<hr/>

###Resumo
* Ao criarmos o construtor de um objeto, devemos usar o objeto prototype
* Projetos grandes precisam usar **módulos** e **namespaces** e isso é possível com o Module pattern, 
que também permite o encapsulamento de objetos
* Para distribuir os mesmos recursos de uma classe a toda a aplicação, deve-se usar o Singleton pattern
que provê um ponto de acesso único e global à instância de uma única classe.

Por hora é isso. Esse artigo contemplará mais patterns em breve. Por favor, dêem feedback quanto à 
didática usada em meus artigos, assim posso melhorá-los cada vez mais. Dúvidas e sugestões são sempre
bem-vindas!
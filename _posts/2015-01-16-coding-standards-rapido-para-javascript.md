---
layout: post
title:  "Coding standard rápido para javascript"
description: "Manter um estilo consistente ao escrever código pode ser um fator decisivo para a manutenibilidade de um sistema por uma grande equipe."
date:   2015-01-16 05:30:00
categories: ['javascript']
---

Manter um estilo consistente ao escrever código pode ser um fator decisivo para a manutenibilidade de um sistema por uma grande equipe, principalmente se o projeto for Open Source. Nesse artigo vou mostrar o padrão usado em meus projetos.

Esse coding standard foi inspirado em alguns scripts de código aberto que utilizei, assim como diretrizes de [Douglas Crockford](http://javascript.crockford.com/code.html), [Google](https://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml), [W3Schools](http://www.w3schools.com/js/js_conventions.asp). Se for segui-lo, recomendo que comece em seu próximo
projeto ou, caso inicie agora mesmo, pense em refator seu projeto atual para evitar colisão 
entre os estilos.

##Nomeação

* variáveis, funções, propriedades, métodos e namespaces com lowerCamelCase: **nomeDaCoisa**
* classes com UpperCammelCase:  **NomeDaClasse**
* callback inicia com *on*: **onMeuCallback**
* arquivos: **nomedoarquivo.js**

####Exemplo de nomeação

`nomedomeuarquivo.js`
{% highlight javascript linenos=table %}
var nomeDoMeu.novoNamespace; // namespace

var nomeDaMinhaVariavel; // variável

// função
function nomeDaMinhaFuncao() { 
    // ...
} 

// classe
function NomeDaMinhaClasse() {
    var nomeDaMinhaPropriedadePrivada; // propriedade privada
    this.nomeDaMinhaPropriedadePublica; // propriedade pública
    
    // método privado
    var nomeDoMeuMetodoPrivado = function () {
        // ...
    }
    
    // método público
    this.nomeDoMeuMetodoPublico = function () {
        // ...
    }
}

// handler de evento
function onNomeDoMeuCallback() {
    // ...
}
{% endhighlight %}  

##Declaração

* sempre declarar uma variável ou função antes de usá-la
* tentar declarar uma variável próxima do contexto em que será usada
* sempre declarar variáveis não-globais depois de **var**, até mesmo quando estiver no escopo global

####Exemplo de declaração

{% highlight javascript linenos=table %}
var animal = 'Cavalo';
var corre = function () {
    console.log(animal + ' correndo');
}
{% endhighlight %}

##Estruturas de controle

* sempre usar um espaço entre os parênteses
* sempre colocar a(s) chave(s) na mesma linha da declaração
* sempre alinhar a(s) chave(s) de fechamento com a declaração

####Exemplo de estruturas de controle

{% highlight javascript linenos=table %}
if (condicao) {
    // ...
} else if (condicao) {
    // ... 
} else {
    // ...
}

switch (expressao) {
    case n:
        // ...
        break;
    case n2:
        // ...
        break;
    default:
        // ...
}

for (inicializacao; condicao; atualizacao) {
    // ...
}

while (condicao) {
    // ...
}

do {
    // ...
} while (condicao);

try {
    // ...
} catch (var) {
    // ...
} finally {
    // ...
}
{% endhighlight %}

##Formatação

* sempre usar 4 espaços para indentar
* tentar não ultrapassar 80 caracteres em uma linha. Se ultrapassar, pule para a próxima linha
inserindo 4 espaços de indentação
* jamais usar parênteses em operadores unários, como `delete`, `typeof` e `void` ou depois de 
keywords, como `return`, `throw`, `case`, `new`.
* sempre usar aspas simples
* sempre agrupar blocos de código relacionados, separando-os por uma linha em branco
* sempre terminar uma expressão com `;`
* objetos e arrays maiores que 3 elementos ou que ultrapassem os 80 caracteres devem ter um valor por linha
* funções e métodos anônimos devem ter os parênteses separados do nome e da abertura de chaves

###Whitespace

* sempre colocar operadores binários `+, -, *, /, %`, etc. entre espaços
* nunca colocar operadores `.` unários entre espaços 

####Exemplo de formatação

{% highlight javascript linenos=table %}
function MeusAlimentos(){
    var foo = 'fooFruta';
    var fooArray = ['maçã', 'banana', 'tomate'];
    
    var bar = 'barVegetais';
    var barArray = [ 
        'pimentão', 
        'brócoli', 
        'pepino', 
        'alface',
        'acelga'
    ];
    
    var fooObjeto = {
        'nome': 'X-Salada Gorduroso',
        'ingredientes': [
            {'nome': fooArray[2], 'unidade': 'rodela', 'quantidade': 2},
            {'nome': barArray[4], 'unidade': 'folha', 'quantidade': 1},
            {'nome': 'hamburguer', 'unidade': 'und.', 'quantidade': 1},
            {'nome': 'queijo', 'unidade': 'g', 'quantidade': 80},
        ],
        'preco': 3.50,
        'lucroPercentual': 80,
        'vendasTotais': 1495
    }
    
    this.comidaSaudavel = function () {
        return fooArray.join(barArray);
    }
    
    this.junkieFood = function () {
        return fooObjeto;
    }
}
{% endhighlight %}

## Arquivos

* sempre usar a extensão **.js**
* importar com `<script src='nomedomeuarquivo.js'>` sempre que possível no fim do *body* e sem usar o 
atributo `type`, pois quem define o MIME Type é o backend. Esse atributo apenas é usado para o navegador 
saber se pode manipular o arquivo requisitado antes mesmo de fazer uma requisição ao servidor.

## Comentários
Somente use comentários para informações relevantes. Tente ser breve e explicativo, sempre! Pense
em frases que você consiga entender, mesmo lendo daqui há 10 anos. Um leve toque de humor
pode fazer bem, a não ser que você esteja comentando um *hard real time system* (o que talvez
provavelmente não irá acontecer com javascript).

## Bônus
* sempre inicie objetos com `{}` e arrays com `[]`
* sempre use `===` e `!==` para evitar problemas com tipos

Por hora, esses são meus coding standards. Com o tempo poderei mudar ou incluir novos standards,
caso veja que que esses não sejam sustentáveis. Aproveitem para dar feedback e também compartilhar
seus coding standards.

fontes:

* [Douglas Crockford](http://javascript.crockford.com/code.html)
* [Google](https://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml)
* [W3Schools](http://www.w3schools.com/js/js_conventions.asp)
---
layout: post
title:  "Sua informação não tem valor!"
description: "Nenhum dado tem valor até o momento em que ele se torna informação. Pouca informação tem valor até que se torne conhecimento. Algum conhecimento tem valor até que..."
date:   2015-02-20 09:01:00
categories: ['informacao']
---

Estamos aqui trabalhando com níveis de abstração distintos: um **dado** passa a ser **informação**, que, por sua vez, passa a ser **conhecimento**. Mas será que cada um desses níveis possui algum valor significativo? 

>> Nenhum dado tem valor até o momento em que ele se torna informação. Pouca informação tem valor até que se torne conhecimento. Algum conhecimento tem valor.

Antes de começar devemos analisar o que é um dado, uma informação e um conhecimento para que possamos valorá-los, vamos lá:

## Dados, metadados e objetos

**Dados** são meros grãos de areia esperando por uma ação externa para levá-los à condição de informação. Estamos cercados deles por toda parte, produzimos dados a cada milésimo de segundo, respiramos dados, vivemos sobre dados.

Isso muda algo em nossa vida? Absolutamente nada! Esses grãos de areia estão aí apenas por que algum dia uma rocha foi sedimentada. Nesse caso, uma maneira um pouco mais elegante de visualizar um grão de areia seria pintando ele de vermelho, para mostrar que é um grão diferente dos outros. O mesmo pode ser feito com um dado, conferindo-lhe um **rótulo**. Dessa forma, acabamos de criar um **[metadado](http://pt.wikipedia.org/wiki/Metadados)**.

Esse procedimento de transformar o dado em metadado não faz com que ele tenha mais valor, no entanto, sua relevância é consideravelmente aumentada em relação ao dado puro. Por exemplo: o dado **"coelho"**. Sem um rótulo pode significar um tipo de animal ou o sobrenome de uma pessoa. Podemos então criar um rótulo **"animal"** para "coelho". Agora ficou mais fácil saber que "coelho" refere-se a um "animal", não? 

Um metadado não é nada mais que uma **tupla**, ou seja, um dado do tipo chave-valor, no qual o primeiro parâmetro é o rótulo e o segundo é o dado em si. Usamos a notação **(animal : "coelho")** para isso, ou seja, **(chave : "valor")**. 

Bom... até aqui sei que coelho é um animal, mas será que estou me referindo à um animal da família dos leporídeos ou à um cachorro chamado carinhosamente de Coelho? O metadado não é capaz de nos informar isso, por essa razão precisamos partir para um nível de abstração mais alto.

Passamos agora ao nível de abstração de objeto: um **objeto** é uma entidade que possui vários metadados. Nesse caso podemos modelar melhor o tal "coelho". Supondo que "coelho" é um cachorro, da raça Pastor Alemão com nome Coelho e que gosta de latir. Podemos usar a notação **Animal(nome : "Coelho", tipo : "cachorro", raça: "Pastor Alemão", gosta : "latir")**.

Melhorou um pouco mais o entendimento do que é "coelho", mas ainda não existe nenhum valor nesse dado. Precisamos melhorar um pouco mais o que entendemos sobre esse cãozinho. 

## Informação
Supondo que eu adotei o cão Coelho, preciso saber que cuidados especiais devo ter com ele. Não quero que ele faça cocô em locais impróprios ou fuja de casa pulando a cerca do quintal. Nesse caso, precisarei de  algumas informações sobre ele.

Supondo que o Coelho faz cocô por toda a casa, pisa por cima e espalha mais um pouco. Temos aí um grande **problema** e o surgimento de uma pergunta: 

*- Como faço para o Coelho fazer cocô somente em um lugar e não pisar em cima?* 

Agora, podemos mapear essa pergunta para uma **necessidade**: 

*- Preciso que o Coelho faça cocô apenas no jardim de casa.* 

Uma **informação** pode ser obtida quando faço perguntas que podem ser respondidas através da **organização, manipulação e processamento** dos dados que eu tenho em mãos. Sabendo disso, agora é necessário realizar anotações diárias sobre o comportamento do Coelho, para que eu possa **tomar uma decisão** sobre o que eu faço com ele.

Ao fim de um mês junto com ele obtive as seguintes informações:

* Coelho faz seu primeiro cocô às 7h da manhã na frente de casa
* Coelho costuma comer grama e ervas daninhas
* Coelho não come ração, apenas comida de gente
* Coelho não possui fezes consistentes
* Coelho dorme muito depois que alimento ele

Cada ítem dessa lista é uma informação específica sobre eventos que acontecem sobre o objeto em questão. Se antes tinhamos grãos de areia (dados), podemos dizer que agora fizemos blocos de concreto usando os grãos de areia (informação). Com esses blocos será possível construir uma casa. 

Caramba! Fiz uma lista de coisas que acontecem com o Coelho e parece que nada sana meus problemas... Essas informações, infelizmente, não possuem nenhum valor para que eu possa tomar uma decisão agora. Preciso passar a um nível maior de abstração. 

## Conhecimento
O **conhecimento** é gerado da interligação entre informações indiretamente correlacionadas a fim de que seja possível criar implicações lógicas coerentes ao contexto ao qual estamos nos referindo. O conhecimento, em suma, é gerado pela identificação de informações que possuem algum grau de ligação.

Com as informações que tenho sobre o cotidiano do Coelho, posso adquirir mais informações relacionadas indiretamente a ele, mas que podem interferir em seu mau comportamento. Há alguns dias, conversei com meu amigo veterinário, o qual contou algumas coisas sobre o lugar em que os animais fazem suas necessidades fisiológicas e mais algumas curiosidades:

* Cães que comem comida de gente tendem a ter diarreia com mais frequência
* Cães comem grama quando estão cor dor de barriga
* Cães estressados podem ter problemas gastrointestinais
* Cães de grande porte devem ficar em espaços em que seja possível correr livremente
* Cães sedentários tendem a ser mais estressados

Para o caso do Coelho tenho mais algumas informações, que podem auxiliar na formação de conhecimento:

* Minha casa tem um jardim de apenas 10m²
* Passo muito tempo fora de casa

A partir do momento que temos informações distintas, podemos correlacioná-las para gerar o conhecimento que pode resolver os problemas gerados pelo Coelho:

* Coelho passa muito tempo sozinho, logo pode ficar estressado
* Coelho é um cão de grande porte e não tem espaço para correr, logo pode ficar mais estressado
* Coelho dorme muito, logo pode estar estressado
* Coelho come grama, logo pode ter dor de barriga

Essas quatro implicações lógicas ainda não são passíveis de resolver o problema. Que tal juntá-las? Se Coelho fica sozinho e não tem espaço para correr, há grandes chances dele estar estressado, por isso tem fortes dores de barriga e acaba comendo grama para tentar melhorar sua flora intestinal.

Agora sim! Temos um profundo nível de conhecimento vinculado às informações do Coelho. Posso dizer com quase 100% de certeza que é possível resolver seu problema eliminando os fatores que causam estresse nele, como, por exemplo, aumentar o tamanho do meu jardim e dar mais atenção a ele quando estou em casa.

Até aqui não há nenhum valor explícito no conhecimento. Apenas juntamos os blocos de concreto (informação), que foram feitos a partir dos grãos de areia (dados) e construimos uma casa com eles (conhecimento). Quem sabe alguém possa morar nela agora?

## Valor
Após mudar meu cotidiano, passei a ter mais tempo com o Coelho. Além do mais comprei uma casa que tem um jardim um pouco maior. Percebi uma melhora significativa na vida de Coelho, que até perdeu alguns kilos e parece estar mais atlético.

Todo dia que chego em casa ele vem correndo para eu afagá-lo, além de ficar todo saltitante em volta de mim quando vou da garagem até a porta de casa. O melhor de tudo é que ele só faz suas necessidades fisiológicas em um cantinho reservado para isso no jardim.

Esse é o real valor percebido no conhecimento que gerei a partir das informações, que, por sua vez, foram geradas através dos dados. Esse valor no entanto foi dado por mim. Essa mesma informação não teria o mesmo valor para uma pessoa que tem um peixe de estimação, por exemplo.

O **valor** é estimado de acordo com a solução que é dada para os problemas de uma pessoa ou empresa. Se minha irmã tivesse um clone do Coelho e conseguisse os mesmos resultados que eu obti após solucionar os problemas gastrointestinais dele, ela não daria o mesmo valor que eu dei.

Se eu construí uma casa de conhecimento com os blocos de informação, que foram feitos com grãos de dados, agora podemos obter valor morando dentro da casa. Esse é o meu objetivo final, que foi atingido partindo do micro (dado), passando por um estágio intermediário (informação) até chegar ao macro (conhecimento), que, por sua vez, gerou valor (morar na casa construída). 

## Qual é o valor do nosso conhecimento?
Agora podemos demonstrar de onde vem o valor de um dado, informação e conhecimento. 

// falar sobre BI, Watson e Google Knowledge Graph
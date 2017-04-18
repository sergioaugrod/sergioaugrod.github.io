---
layout: post
title:  "Entendendo o processo de compilação"
date:   2017-02-27 18:01:00
categories: compilers
---

Você sabe como o compilador compila um código fonte? Gostaria de entender um pouco mais sobre o assunto? Neste artigo irei explicar de uma forma bem resumida o processo de compilação.

#### O que é um compilador?

O compilador transforma o código fonte em código objeto (comumente chamado de linguagem de máquina). Mas o trabalho que ele realiza não é tão simples, o processo de compilação é divido em várias etapas. Iremos conhecer as principais, e entender o básico deste processo.

#### Compilação

Podemos dividir as fases de compilação em duas áreas, sendo *front-end* e *back-end*. O *front-end* faz a análise do código, agrupa os símbolos e verifica se o código fonte faz sentido. Já o *back-end* realiza todas as ações para transformar a estrutura gerada no *front-end* em um código otimizado que a máquina que irá entender.

Diagrama demonstrando todo o fluxo de compilação:

![Estrutura do Compilador](/assets/images/compilers_structure.png)

##### Front-end

O *front-end* é também dividido em algumas fases, sendo elas, **Análise Léxica**, **Análise Sintática**, **Análise Semântica** e **Geração de Código Intermediário**. Vamos falar sucintamente de cada uma delas logo abaixo.

###### Análise Léxica

O Analisador Léxico também conhecido como *scanner*, é o responsável por dividir o código em símbolos (*tokens*), estes são as unidades do código, como números, operações, atribuições, etc. É também nesta fase que é retirado partes do código que não fazem sentido para a máquina, como comentários e espaços em branco.

Exemplo de uma análise léxica de uma estrutura de controle `if(a > 3)`:

![Análise Léxica - Tabela de Símbolos](/assets/images/compilers_analysis_lexical.png)

Como podemos notar, esta é a fase que dá sentido a cada um dos símbolos do nosso código. O resultado final desta etapa é a tabela de símbolos, exemplificada pela imagem acima.

###### Análise Sintática

Nesta fase é utilizada o resultado produzido pela Análise Léxica, e esta entrada é transformada em uma a árvore, a chamada *syntax tree*, nesta árvore é representada a hierarquia de todos os símbolos produzidos pela fase anterior.

Para representar a expressão:

{% highlight c %}
(5 + 7) - 2
{% endhighlight %}

Teríamos a seguinte árvore:

![Análise Sintática - Abstract Syntax Tree](/assets/images/compilers_syntax_tree.png)

Como podemos notar, primeiro é executado a soma de `5` mais `7` e logo em seguida é feito a subtração de `12` menos `2`.

###### Análise Semântica

A Análise Semântica utiliza a tabela de símbolos gerada na Análise Léxica e a árvore sintática retornada na Análise Sintática. Nesta fase é validado se as expressões estão acordo com a definição da linguagem, fluxos de controle, etc. É nesta etapa que é capturado erros como:

* Divisão por zero;
* Erros de conversão de tipos;
* Variáveis/nomes não existentes;
* Declarações redundantes;
* Acesso de variáveis fora do escopo.

###### Geração de Código Intermediário

É a ponte entre o *front-end* e a *back-end* do compilador, independe da estrutura da máquina alvo, diferentemente das etapas posteriores, que já irão considerar as variáveis da máquina alvo. Esta etapa pode ser representada por várias maneiras, no caso da linguagem Java como *Byte Code*, ou quando for independente de linguagens, como *Three-Address Code* (representado por quadruplas ou triplas).

A fim de exemplificar uma das funções desta fase, considere a seguinte expressão:

{% highlight c %}
a = x + y * z
{% endhighlight %}

Nesta etapa, o código acima seria divido em sub-expressões (*Three-Address Code*):

{% highlight c %}
r1 = y * z
r2 = x + r1
a = r2
{% endhighlight %}

A principal função desta fase é aproximar todo o resultado das etapas anteriores para o código que será executado na máquina.

##### Back-end

O *back-end* é divido entre **Otimização de Código** e **Geração de Código**. Abaixo iremos demonstrar cada uma delas.

###### Otimização de Código

Nesta fase é otimizado o código intermediário gerado no *front-end*. Será utilizado algumas técnicas para que o código fique mais eficiente em relação ao uso de CPU, memória, etc. Blocos de códigos serão modificados afim de se tornarem mais eficientes.

Considere o seguinte código escrito em C:

{% highlight c %}
int value = 10;

while(value < 100) {
  int tax = 10;
  value = value + tax;
}
{% endhighlight %}

Otimizando este código (extraindo a constante `tax`):

{% highlight c %}
int value = 10;
int tax = 10;

while(value < 100) {
  value = value + tax;
}
{% endhighlight %}

Apesar de todas as técnicas utilizadas, o código otimizado nunca deverá ter o significado diferente do código de entrada.

###### Geração de Código

É a fase final da compilação, nesta fase é considerado todas as premissas da máquina alvo para a geração do código. O resultado final é a linguagem de máquina.

#### Concluindo

Espero que este artigo tenha esclarecido sobre como é o processo de compilação, tentei resumir ao máximo todos os tópicos mantendo os principais conceitos. Para maior detalhamento deste assunto, recomendo a leitura do livro *Compilers: Principles, Techniques, and Tools*.

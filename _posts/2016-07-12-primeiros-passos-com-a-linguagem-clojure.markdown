---
layout: post
title:  "Primeiros passos com a linguagem Clojure"
date:   2016-07-12 22:49:00
disqus: true
categories: clojure getting-started
---

**Clojure** é uma linguagem bem legal, muda bastante a forma de pensar no código, principalmente quando você vem de uma linguagem orientada a objetos, como **Ruby**. A linguagem presa sempre pela imutabilidade de valores, não existem variáveis, e nem estado mutável, característica que facilita bastante a criação de *software* concorrente. Outra coisa bem interessante, é que você faz muita coisa, com pouco código, graças a excelente *standard library*. A linguagem é totalmente extensível por *macros*, a comunicação com o **Java** é bastante simples, com isso podemos facilmente utilizar *libs* criadas para o **Java** no **Clojure**.

**Resumão da linguagem:**

* Paradigma funcional;
* Dialeto de LISP;
* Projetada para concorrência;
* Fácil integração com o Java;
* Tipagem forte e dinâmica;
* Estruturas de dados imutável;
* Roda por meio da JVM ou no V8 (Clojurescript);
* O código é uma estrutura de dados.

### Coding

A primeira coisa que se deve fazer é instalar o [**Leiningen**](http://leiningen.org/), gerenciador de projetos da linguagem. Para quem é do **Ruby**, ele é meio que uma mistura de *bundler* com *rake*.

Ubuntu:

{% highlight shell_session %}
$ apt-get install leiningen
{% endhighlight %}

OSX:

{% highlight shell_session %}
$ brew install leiningen
{% endhighlight %}

Com o **Leiningen** instalado já teremos de brinde a linguagem **Clojure** e o seu **READ EVAL PRINT LOOP** (console) para brincarmos um pouco.

Antes de iniciar o **REPL**, vale destacar que o **Clojure** possui os seguintes tipos:

* **Number**: *100*
* **String**: *"Hello World"*
* **Vector**: *[100 "Hello World"]*
* **Keyword**: *:hello-world*
* **Map**: {:blog "sergioaugrod" :posts 5}
* **Set**: #{150 160 "Hello World"}

Em seguida, vamos iniciar o **REPL** com o seguinte comando:

{% highlight shell_session %}
$ lein repl
{% endhighlight %}

Já com o *console* do **Clojure** aberto, poderemos brincar um pouco com a sua *syntax*.

Vamos somar 3 números:

{% highlight clojure %}
(+ 1 2 3)
=> 6
{% endhighlight %}

*O primeiro elemento é o operador, e os seguintes são os operandos.*

Definir um nome para algum valor:

{% highlight clojure %}
(def number 10)
{% endhighlight %}

*Não existe variável na linguagem, você somente dá nome a coisas, sem a possibilidade a alteração do valor nomeado.*

Criando uma função que soma um valor com o número 10:

{% highlight clojure %}
(defn sum-10
  [number]
  (+ number 10))
{% endhighlight %}

Executando a função **sum-10**:

{% highlight clojure %}
(sum-10 10)
=> 20
{% endhighlight %}

Função anônima:

{% highlight clojure %}
(fn [x] (+ x 1 2 3))
{% endhighlight %}

Utilizando o método **map** com uma função anônima:

{% highlight clojure %}
(def numbers [1 2 3 4])
(map (fn [x] (inc x)) numbers)
=> (2 3 4 5)
{% endhighlight %}

Criando um **Map** e acessando suas propriedades:

{% highlight clojure %}
(def city {:name "São Paulo" :age 462})

(:name city)
=> "São Paulo"
{% endhighlight %}

Criando uma uma estrutura **Record** e acessando suas propriedades:

{% highlight clojure %}
(defrecord Car [name year])

(def car-x (Car. "Car X" "2015"))

(:name car-x)
=> "Car X"
{% endhighlight %}

### Como estudar?

Recomendo fortemente o livro [Clojure for the Brave and True](http://www.braveclojure.com/), é possível ler o mesmo gratuitamente pelo site do livro. Outra boa maneira de estudos é a resolução de exercícios de **Clojure** no [exercism](http://exercism.io/) (plataforma de resolução de exercícios).

Quando houver dúvidas sobre a linguagem, um bom lugar para se tirar dúvidas sobre código e funções é o [ClojureDocs](http://clojuredocs.org/), onde é possível obter informações detalhadas sobre determinadas funções, com exemplos de códigos.

O [TryClojure](http://www.tryclj.com/) é uma boa forma de se ter o primeiro contato com a linguagem, pois é possível testar a linguagem pelo *browser*.

### Editor de Texto

Os melhores editores de texto para se programar na linguagem é o *Emacs* e o *IntelliJ* com o [*Cursive*](https://cursive-ide.com/), porém nada o impede de usar outros, eu por exemplo utilizo o *vim* com o *plugin* *vim-fireplace*.

A grande vantagem do *Emacs* e *IntelliJ (Cursive)* é que você consegue facilmente ir avaliando as expressões enquanto desenvolve, onde o *REPL* é iniciado dentro do próprio editor. O *vim* com o *vim-fireplace* tenta fazer um pouco disso, mas não chega nem perto da usabilidade dos outros dois editores.

### Concluindo

Uma das melhores coisas que fiz nestes últimos meses foi estudar **Clojure**, a linguagem é sensacional, muito fácil de se aprender e bastante poderosa. Lendo um pouco pude notar que ela se destaca bastante em tarefas que exigem alto nível de concorrência, ainda não tive a oportunidade de utilizar a mesma em produção, mas estou a procura de um novo desafio para utilizá-la.

Tentei dar um *overview* da linguagem, apresentando conceitos básicos, para ajudar em um primeiro contato com a linguagem. Pretendo em um próximo *post* escrever um pouco sobre concorrência em **Clojure**, que é uma das coisas mais legais da linguagem.

Obrigado.

### Referências e Recomendações

* <http://clojure.org>
* <http://clojure.org/api/cheatsheet>
* <https://github.com/clojure/clojure>
* <http://www.braveclojure.com>
* <http://leiningen.org>
* <http://clojuredocs.org>
* <http://www.tryclj.com>
* <https://github.com/razum2um/awesome-clojure>

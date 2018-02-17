---
title: Criando uma linguagem de programação com o ANTLR4
layout: post
date: '2017-10-08 16:00:00'
disqus: 'true'
categories: compilers
---

Neste artigo, vamos aprender como criar uma linguagem de programação simples com o **ANTLR4**, que é uma ferramenta em que você define uma gramática, e por meio do resultado de sua gramática você cria os comportamentos de sua linguagem. Recomendo a leitura do artigo [Entendendo o Processo de Compilação](https://www.sergioaugrod.com/compilers/2017/02/27/entendendo-o-processo-de-compilacao.html) que tem uma explicação um pouco mais detalhada sobre o que é Análise Léxica e Análise Sintática, que é o resultado gerado pelo ANTLR4 por meio de sua gramática.

#### Instalando o ANTLR4 no Linux

- Instale o Java com versão acima da 1.6;
- Execute os passos abaixo para baixar e instalar o binário no Linux:

{% highlight shell_session %}
$ cd /usr/local/lib
$ curl -O http://www.antlr.org/download/antlr-4.5.3-complete.jar
{% endhighlight %}

- Exporte o CLASSPATH do ANTLR4, se possível já adicione em seu .bash_profile ou .zshrc:

{% highlight shell_session %}
$ export CLASSPATH=".:/usr/local/lib/antlr-4.5.3-complete.jar:$CLASSPATH"
{% endhighlight %}

Para mais detalhes sobre a instalação visite o [Getting Started](https://github.com/antlr/antlr4/blob/master/doc/getting-started.md) do ANTLR4, que também possui os passos para instalação em outros sitemas operacionais.

#### Criando a gramática de nossa linguagem

A gramática da nossa linguagem é onde será definida a *syntax*, e tudo que é permitido se ter como instrução. Vamos chamar nossa linguagem de **Zerolang**. Para isso vamos criar nosso arquivo de gramática Zerolang.g4 (g4 é a extensão do ANTLR).

Já editando o arquivo Zerolang.g4, no início vamos definir o nome da gramática (mesmo  nome do arquivo):

```
grammar Zerolang;
```

Agora vamos criar a chave inicial da nossa gramática, vamos dar o nome de *program*, podendo ser qualquer outro nome de sua escolha:

```
program
    : 'begin' statement+ 'end';
```

Repare que logo abaixo da chave há uma outra declaração, nela queremos dizer, que nosso código deve iniciar com *begin*, ter um ou mais *statement* e terminar com *end*.  Agora vamos definir o que é a chave *statement*:

```
statement
    : assign
    | print ;
```

Como descrito acima, essa chave pode ser outra duas chaves, *assign* ou *print*. Vamos começar definindo o que é a chave *assign*, que é a responsável pela declaração de variáveis:

```
assign
    : 'var' ID '=>' (NUMBER | ID) ;
```

Nossa declaração de variável na Zerolang será da seguinte forma: `var variavel => 10`. O valor da variável pode ser um número(NUMBER) ou um identificador(ID), que vamos deixar para definir no final. Agora vamos para definição da chave *print*:

```
print
    : 'print' (NUMBER | ID) ;
```

O print em nossa linguagem poderá ser de um número(NUMBER) ou uma variável(ID). Por exemplo: `print 25`.

E no final da nossa gramática vamos definir o que é um número, um identificador e o que ignorar no código:

```
ID     : [a-z]+ ;
NUMBER : [0-9]+ ;
WS     : [ \n\t]+ -> skip;
```

O resultado final deste arquivo pode ser visualizado no [Github](https://github.com/sergioaugrod/zerolang/blob/master/src/Zerolang.g4).

#### Gerando o Parser e Lexer da linguagem

O ANTLR4 é uma ferramenta bem interessante, e com ela somente é necessário escrever a gramática da linguagem, sem se preocupar com o resultado do lexer/parser, tudo será gerado pela ferramenta. O código gerado pelo **ANTLR4** por padrão é para a linguagem Java, mas tem *outputs* para Python, Javascript entre outras linguagens. Para gerar os arquivos por meio do arquivo Zerolang.g4 siga os passos abaixo:

{% highlight shell_session %}
$ antlr4 Zerolang.g4
$ javac Zerolang*.java
{% endhighlight %}

#### Testando nossa gramática

Executando o comando abaixo você conseguirá testar a gramática criada e visualizar a AST gerada pelo seu código fonte:

{% highlight shell_session %}
$ grun Zerolang program -gui
begin
a => 10
print a
end
{% endhighlight %}

#### Navegando pela árvore gerada pelo parser do ANTLR

Agora devemos navegar pela AST e assim estabelecer os comportamentos de nossa linguagem. Devemos criar um *Listener*, abaixo criei uma classe chamada *MyListener* e herdei da classe *ZerolangBaseListener* (gerada pelo ANTLR4). Implementei utilizando a linguagem Kotlin, fiquem a vontade para utilizar o Java.

{% highlight kotlin %}
class MyListener : ZerolangBaseListener() {
    val variables : MutableMap<String, Int?> = mutableMapOf()

    override fun exitAssign(ctx: ZerolangParser.AssignContext) {
        val variableName = ctx.ID(0).text
        val variableValue = Integer.parseInt(ctx.NUMBER().text)

        variables.put(variableName, variableValue)
    }

    override fun exitPrint(ctx: ZerolangParser.PrintContext) {
        val output = if (ctx.ID() == null)  ctx.NUMBER().text else variables[ctx.ID().text]
        println(output)
    }
}
{% endhighlight %}

Na classe acima sobrescrevi dois métodos: *exitAssign* e *exitPrint*. O primeiro será executado após uma instrução de atribuição de variável (assign) e o segundo após uma instrução de print. É aqui que você deverá implementar os comportamentos de sua linguagem. No caso do *exitAssign* estamos "criando" as variáveis e adicionando em um *HashMap*. O *exitPrint*  "printa" um valor ou uma varíavel.

#### Executando o código

Vamos agora criar a classe principal da nossa linguagem, responsável por executar nosso código fonte:

{% highlight kotlin %}
import org.antlr.v4.runtime.ANTLRInputStream
import org.antlr.v4.runtime.CommonTokenStream
import java.io.FileInputStream

fun main(args: Array<String>) {
    val sourceCode = "code.zl"
    val input = ANTLRInputStream(FileInputStream(sourceCode))

    val lexer = ZerolangLexer(input)
    val parser = ZerolangParser(CommonTokenStream(lexer))
    parser.addParseListener(MyListener())

    parser.program()
}
{% endhighlight %}

Nesta classe estamos lendo o código fonte, criando o lexer, parser e adicionando o nosso o *Listener* criado anteriormente.

Vamos agora criar o arquivo com o nome de `code.zl` (nome definido na classe anterior), que conterá o código fonte de nossa linguagem:

```
begin
    var a => 10
    print a
end
```

Vamos testar nossa linguagem criada:

{% highlight shell_session %}
$ java Main
{% endhighlight %}

#### Links

- https://github.com/antlr/antlr4
- https://github.com/antlr/antlr4/blob/master/doc/getting-started.md
- https://github.com/sergioaugrod/zerolang
- https://www.sergioaugrod.com.br/compilers/2017/02/27/entendendo-o-processo-de-compilacao.html
- https://kotlinlang.org

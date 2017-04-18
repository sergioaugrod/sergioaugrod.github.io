---
layout: post
title:  "A importância da utilização do X-Request-ID nas aplicações"
date:   2016-08-21 22:21:00
disqus: true
---

O **Request-ID** é um identificador aleatório que identifica uma requisição, ele é importante para rastrear e mapear o fluxo de ações realizadas a partir de uma requisição na aplicação.

Quando uma requisição é realizada, o cliente pode enviar o seu próprio **Request-ID** no *header* da requisição, por exemplo:

{% highlight shell_session %}
$ curl http://localhost:3000/posts -H "X-Request-ID: 94653b24-1776-4936-aabe-98610f0e042f"
{% endhighlight %}

Com isso na aplicação *web* podemos mapear este valor para identificar todas as ações na aplicação oriundas desta requisição.

No *Rails*, a partir da versão *3.2*, é possível facilmente mapear o **Request-ID** no log, adicionando a seguinte linha em `config/application.rb`:

{% highlight ruby %}
config.log_tags = [:uuid]
{% endhighlight %}

Com isso, no caso da requisição mostrada anteriormente, teríamos o seguinte *log*:

```
[94653b24-1776-4936-aabe-98610f0e042f] Processing by PostsController#index as JSON
```

Podemos notar que a *tag* `[94653b24-1776-4936-aabe-98610f0e042f]` tem o mesmo valor enviado na chave **X-Request-ID**.

*Obs: Caso o cliente não envie este identificador, o mesmo é gerado aleatoriamente na aplicação.*

###### Quando utilizar o Request-ID?

Caso tenha um sistema separado em micro serviços e queira analisar de ponta-a-ponta todas as requisições, você somente deverá repassar o primeiro **Request-ID** gerado em todas as requisições.

É importante também para facilitar o mapeamento de erros ocorridos nos clientes. Sempre retornar o **Request-ID** nas páginas de erros, facilitará o mapeamento das causas do erro na aplicação.

###### Finalizando
É bastante simples o uso do **Request-ID**, e não tem porque não usar. Ele é bastante útil no rastreamento de requisições. Neste *post* dei o exemplo de como configurá-lo no log do *Rails*, mas em outros *frameworks* *web* também é bastante simples a configuração.

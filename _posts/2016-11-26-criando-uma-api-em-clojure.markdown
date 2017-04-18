---
layout: post
title:  "Criando uma API em Clojure"
date:   2016-11-26 18:28:00
disqus: true
categories: clojure
---

Olá pessoal, neste artigo vamos criar uma **API** com a linguagem **Clojure**, onde vamos expor alguns **endpoints**, armazenar e recuperar dados. Iremos construir as rotas da nossa aplicação, definir o modelo dos dados e salvar no banco. Preparados? Vamos lá.

### Criando nossa aplicação

Para este tutorial, iremos utilizar a *library* [Compojure](https://github.com/weavejester/compojure), que facilita a criação de rotas. O **Compojure** é construído em cima do [Ring](https://github.com/ring-clojure/ring), que é uma abstração para lidar com **HTTP** no **Clojure**, com ideias similares ao **Rack** do **Ruby**. Lembrando que o **Compojure** não é um *framework* *web*, e não tem características similares ao **Rails** ou **Django**, ele apenas facilita a criação das rotas da nossa aplicação.

Agora que já sabe o que é o **Compojure**, podemos criar nossa aplicação, e para a nossa sorte, já existe um *template* do [Leiningen](https://github.com/technomancy/leiningen) para criar a estrutura de *app*:

{% highlight shell_session %}
$ lein new compojure compojure-started-api
{% endhighlight %}

Após a execução deste comando, será criado o projeto *compojure-started-api* com os seguintes diretórios:

```
project.clj - Informações sobre o projeto, dependências
src - Código fonte do projeto
test - Testes
```

Dentro do diretório *src*, teremos:

```
compojure_started_api - Namespace raiz do projeto
  handler.clj - Bootstrap da aplicação, onde irá conter as rotas
```

No arquivo *handler.clj* teremos uma rota padrão `/` definida, que somente retorna `Hello World`:

{% highlight clojure %}
(ns compojure-started-api.handler
  (:require [compojure.core :refer :all]
            [compojure.route :as route]
            [ring.middleware.defaults :refer [wrap-defaults site-defaults]]))

(defroutes app-routes
  (GET "/" [] "Hello World")
  (route/not-found "Not Found"))

(def app
  (wrap-defaults app-routes site-defaults))
{% endhighlight %}

Para testar essa rota, devemos subir a aplicação:

{% highlight shell_session %}
$ lein ring server
{% endhighlight %}

Será aberto o *browser* com a rota padrão, com o texto `Hello World`.

A estrutura inicial da nossa aplicação foi criada, agora devemos construir o nosso *resource* de produtos, onde iremos, armazenar e listar.

### Criando os Recursos

Vamos criar as rotas para listar e criar produtos, em seguida o modelo, e por último, iremos salvar e listar este produto do banco de dados.

Primeiros vamos alterar o *middleware* do *ring*, para retornar sempre a resposta em `json` e receber `json-params`:

{% highlight clojure %}
(ns compojure-started-api.handler
  (:require [compojure.core :refer :all]
            [compojure.route :as route]
            [ring.middleware.defaults :refer [wrap-defaults site-defaults]]))

(defroutes app-routes
  (GET "/" [] "Hello World")
  (route/not-found "Not Found"))

(def app
  (wrap-defaults app-routes site-defaults))
{% endhighlight %}

É necessário adicionar uma nova dependência no *projects.clj*:

```
[ring/ring-json "0.4.0"]
```

Agora vamos criar a rota para armazenar o produto `POST /products`:

{% highlight clojure %}
(POST "/products" request
      (let [description (get-in request [:body "description"])
            price (get-in request [:body "price"])]
        (str "Desc: " description ", Price: " price)))
{% endhighlight %}

Em seguida criaremos a rota para listar os produtos `GET /products`:

{% highlight clojure %}
(GET "/products" [] [])
{% endhighlight %}

Insira as rotas criadas logo abaixo da rota padrão `GET /`.

Por enquanto nossa rota de criação simplesmente retorna os valores enviados, vamos testá-la:

{% highlight shell_session %}
$ curl -X POST http://localhost:3000/products -H "Content-Type: application/json" -d '{"description":"Product", "price":"10.55"}'
{% endhighlight %}

O retorno deverá ser:

```
Desc: Product, Price: 10.5
```

A nossa rota criada logo acima não está salvando nada, vamos agora criar nosso modelo `Product`. Para não ficar muita informação no mesmo artigo, não iremos abordar migrações, então é necessário que você crie a tabela `product` em seu banco de dados:

{% highlight sql %}
CREATE TABLE products(
  id SERIAL PRIMARY KEY NOT NULL,
  description varchar(50) NOT NULL,
  price float8 NOT NULL
);
{% endhighlight %}

*Obs: Para criar migrações, recomendo o uso da lib [migratus](https://github.com/yogthos/migratus).*

Agora vamos usar a excelente lib [Korma](http://sqlkorma.com/) para facilitar a manipulação da nossa tabela do banco de dados. Adicione ela no seu arquivo *project.clj*, no *vector* de *dependencies*:

```
[korma "0.4.3"]
```

Também adicione a dependência do *driver* do postgres:

```
[postgresql "9.3-1102.jdbc41"]
```

Agora vamos definir nossas entidades, crie o arquivo *compojure-started-api/entities.clj*:

{% highlight clojure %}
(ns compojure-started-api.entities
  (:use [korma.core]
        [korma.db]))

(defdb db (postgres {:db "compojure-started-api"
                     :user "compojure-started-api"
                     :password ""}))

(defentity products)
{% endhighlight %}

Altere as configurações do banco de dados com suas credenciais. Assim já podemos manipular os produtos.

Não vamos deixar nossa rota conter lógicas de negócio, sendo assim vamos criar um novo *namespace* para gerenciar os produtos, e iremos chamar funções deste namespace em nossas rotas.

Crie o arquivo `compojure-started-api/product.clj`, contendo funções para listar todos os produtos, e inserir novo produto:

{% highlight clojure %}
(ns compojure-started-api.product
  (:require [korma.db :refer :all]
            [korma.core :refer :all]
            [compojure-started-api.entities :refer :all]))

(defn find-all
  []
  (select products))

(defn create
  [description price]
  (insert products
          (values {:description description :price (read-string price)})))
{% endhighlight %}

Agora vamos usar as funções criadas em nossos `endpoints` no arquivo `compojure-started-api/handler.clj`:

{% highlight clojure %}
(ns compojure-started-api.handler
  (:require [compojure.core :refer :all]
            [compojure.route :as route]
            [ring.middleware.json :refer [wrap-json-response wrap-json-body]]
            [compojure-started-api.product :as product]))

(defroutes app-routes
  (GET "/" [] "Hello World")
  (GET "/products" [] (product/find-all))
  (POST "/products" request
        (let [description (get-in request [:body "description"])
              price (get-in request [:body "price"])]
          (product/create description price)))
  (route/not-found "Not Found"))

(def app
  (-> app-routes
      wrap-json-response
      wrap-json-body))
{% endhighlight %}

Criando nosso primeiro produto:

{% highlight shell_session %}
$ curl -X POST http://localhost:3000/products -H "Content-Type: application/json" -d '{"description":"Product", "price":"10.55"}'
{% endhighlight %}

Obtendo a lista de produtos cadastrados:

{% highlight shell_session %}
$ curl http://localhost:3000/products
{% endhighlight %}

Resposta:

{% highlight json %}
[{"id":5,"description":"Product","price":10.55}]
{% endhighlight %}

### Github

O projeto que construímos acima está disponível no *Github*:

* <https://github.com/sergioaugrod/compojure-started-api>

### Concluindo

Vimos alguns exempos de como criar `endpoints` com a lib *compojure* e como armazenar e listar dados de um banco de dados, espero que possa ser útil para vocês de alguma forma. Abraço e até a próxima.

### Refêrencias e Links

* <http://leiningen.org>
* <https://github.com/ring-clojure/ring>
* <https://github.com/weavejester/compojure>
* <https://github.com/ring-clojure/ring-json>
* <https://www.postgresql.org>
* <http://sqlkorma.com>

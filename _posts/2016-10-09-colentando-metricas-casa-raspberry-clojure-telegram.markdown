---
layout: post
title:  "Home Instinct: Coletando métricas da minha casa"
date:   2016-10-09 14:27:00
categories: iot clojure
---

**Home Instinct** é uma ideia de coletar métricas e dados da minha casa, e me alertar caso algum evento importante acontença. Toda a ideia se resume a alguns **Sensores**, uma placa **Raspberry**, *código* em **Python**, **Clojure** e **Javascript**.

O projeto consiste em **3 subprojetos**:

* [Fulgore (Python)](https://github.com/sergioaugrod/fulgore)
* [Cinder (Clojure)](https://github.com/sergioaugrod/cinder)
* [Jago (Javascript)](https://github.com/sergioaugrod/jago)

Vou falar mais sobre eles no decorrer do artigo.

### Projetos:

Vou tentar dar um resumo sobre cada projeto, mas se você é como eu, e gosta de entender o projeto pelo código, sinta-se a vontade para visualizar o código no *github*.

#### Fulgore

Comunica com o **Raspberry**, este conectado com vários sensores em seus pinos **GPIO**, lê seus pinos e envia os dados para tópicos de um *broker* **MQTT**.

#### Cinder

Escuta todos os tópicos do **MQTT** que o *Fulgore* envia, pega os dados recebidos e envia para o **InfluxDB**, e também dispara sinais *WebSockets*. Outra coisa legal é a integração com o **Telegram**, existem alguns poucos comandos mapeados como:

* **get_temperature** - Retorna o último valor da temperatura;
* **get_humidity** - Retorna o último valor da umidade;
* **get_presence** - Retorna o último valor do sensor de presença;
* **get_luminosity** - Retorna o último valor do sensor de luminosidade.

Nele também está mapeado alguns eventos, como: *"Se alguém passar pelo sensor de presença"*, notifique uma determinada sala do **Telegram**.

#### Jago

É um *dashboard* bem simples construído com **React**, que reflete na tela todos os dados vindos via *WebSockets* da aplicação *Cinder*.

### Visualização:

O *Jago* é uma aplicação para visualização de dados, mas como estamos lidando com dados temporais e armazenando no **InfluxDB**, utilizamos também o **Grafana** para visualizar as métricas dos sensores.

### Diagrama:

Abaixo está o diagrama de relacionamento entre sensores, placa e os sistemas:

![Home Instinct Diagram](/assets/images/home_instinct_diagram.svg)

### Sketch:

![Home Instinct Sketch](/assets/images/home_instinct_sketch.jpg)

Neste primeiro momento o *sketch* é bem simples, com poucos sensores, pois foquei mais na arquitetura em si, de como os sistemas vão se comunicar, para depois ir adicionando mais sensores, alertas e comandos.

### Executando em sua casa:

O que você precisa para executar em sua casa:

* **Raspberry Pi**;
* Sensores de temperatura, umidade, luminosidade e presença;
* Executar o **MQTT**, **InfluxDB** e **Grafana**;
* Executar o *Fulgore* no **Raspberry Pi**, atentando para numeração dos pinos utilizados nos sensores;
* Executar o *Cinder* conectado com o mesmo **MQTT** que o *Fulgore* envia os dados;
* Executar o *Jago* caso queira visualizar os dados.

Cada projeto possuí uma documentação simples para ajudar na execução, recomendo entrar em cada projeto no *github* para maior detalhamento.

* <https://github.com/sergioaugrod/fulgore>
* <https://github.com/sergioaugrod/cinder>
* <https://github.com/sergioaugrod/jago>

### O que vem pela frente:

* Outros microcontroladores espalhados pela casa (Node MCU);
* Mais sensores;
* Criação de novos comandos para o **Telegram** no *Cinder*;
* Criação de mais alertas;
* Melhorar documentação de uso.

Neste artigo fui bem resumido em relação ao uso das tecnologias, no decorrer do mês irei criar mais *posts* para ir explicando cada passo... Obrigado.

### Links:

* <https://mosquitto.org>
* <https://www.influxdata.com/time-series-platform/influxdb>
* <http://grafana.org>

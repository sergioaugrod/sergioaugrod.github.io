---
layout: post
title:  "IoT: Troca de mensagens com o MQTT"
date:   2016-06-27 16:32:00
categories: iot mqtt
---

Este é o primeiro *post* do blog, e o primeiro também da série de **IoT**. Vou explicar o que é **MQTT** e como utilizá-lo para troca de mensagens entre dispositivos.

O **MQTT** (*Message Queue Telemetry Transport*) é um protocolo leve para troca de mensagens, onde basicamente você envia mensagens para **filas**, e estas podem ser **escutadas** por outros.

Com ele você pode projetar sua arquitetura de dispositivos para se comunicar por meio de troca de mensagens. Assim poderíamos ter vários dispositivos enviando valores de sensores, e outros obtendo estes valores e realizando algum tipo de processamento.

### Caso prático de uso:

Temos dois serviços: *Sensor* e *Dashboard*.

O serviço *Sensor* quer a todo momento enviar o valor atual da sua temperatura para quem quiser escutar.

O serviço *Dashboard* quer ter sempre o valor atualizado da temperatura do *Sensor*.

Como resolveríamos este problema utilizando **MQTT**?

* Definiremos que o serviço *Sensor* irá enviar sempre sua temperatura por meio do tópico **temperatura**.
* O serviço *Dashboard* irá ficar escutando o tópico **temperatura**, com isto irá obter a cada atualização, o valor atual de temperatura.

### Lado técnico da coisa:

**Broker e Client**:

Entenda o MQTT como um protocolo, e este protocolo deve ser implementado tanto para servers quanto para clients. O serviço que provê a troca da mensagens é chamado de **Broker** e os clientes que escutam e enviam mensagens são os **Clients**.

Existem várias implementações de **Broker**, o *Mosquitto* é um exemplo de  implementação *open source*. Em relação aos **Clients** existem implementações para várias linguagens, como Java, Ruby, Clojure, Python, C, etc.

Neste *post* vamos utilizar o [Mosquitto](http://mosquitto.org) como **broker** e **client**.

**Instalação do Mosquitto (Broker)**:

Mac OSX:

{% highlight shell_session %}
$ brew install mosquitto
{% endhighlight %}

Ubuntu:

{% highlight shell_session %}
$ apt-get install mosquitto
{% endhighlight %}

**Execução do Broker**:

Mac OSX:

{% highlight shell_session %}
$ /usr/local/sbin/mosquitto -c /usr/local/etc/mosquitto/mosquitto.conf
{% endhighlight %}

Ubuntu:

{% highlight shell_session %}
$ mosquitto -c /etc/mosquitto/mosquitto.conf
{% endhighlight %}

**Interagindo com o Broker**:

Enviando uma mensagem:

Enviando a mensagem **36** para o tópico **temperatura**.

{% highlight shell_session %}
$ mosquitto_pub -t 'temperatura' -m 36
{% endhighlight %}

Recebendo mensagens:

Escutando todas as mensagens do tópico **temperatura**.

{% highlight shell_session %}
$ mosquitto_sub -t 'temperatura'
{% endhighlight %}

*Uma coisa que pode se notar, é que o **MQTT** não é persistente, então os valores vão se perdendo caso nenhum dispositivo esteja escutando as filas.*

Pessoal espero ter conseguido explicar o básico do **MQTT**, e facilitar um possível *Getting Started* de como utilizá-lo.

Obrigado!

---
layout: post
title:  "IoT: Arduino, Serial e MQTT"
date:   2016-07-08 01:09:00
disqus: true
categories: iot mqtt arduino
---

Continuando com a série de *posts* sobre **Internet of Things**, vou demonstrar como publicar os dados de um **Arduino** em um *broker* **MQTT**. Nesta explicação, a comunicação entre a placa e a Internet será por meio da porta serial, mas esta comunicação pode ser direta com o **MQTT**, caso o **Arduino** possua uma placa de rede sem fio.

#### Serial Port

O **Arduino** possui uma *serial port* que pode ser conectada a um computador via *USB*. Essa porta é muito importante para troca de dados, principalmente quando o **Arduino** não possui uma placa de rede, e não tem autonomia de envio de dados.

Umas das primeiras coisas que geralmente são demonstradas em tutoriais de **Arduino**, é o envio de dados de sensores para a porta a *serial*. Sendo que estes dados são exibidos no *Serial Monitor* na própria *IDE* da placa. Neste *post* vamos enviar estes dados também para a porta *serial*, e depois estes dados serão capturados por uma *app* em *Ruby*, e em seguida serão postados para o *MQTT*.

### Ruby e Serial

Neste exemplo vamos utilizar a gem [rubyserial](https://github.com/hybridgroup/rubyserial), para receber os dados da placa.

Para estabelecer a conexão serial, somente é necessário o código *Ruby* abaixo:

{% highlight ruby %}
require 'rubyserial'

# Porta serial e baud rate.
conn = Serial.new('/dev/ttyACM0', 9600)
{% endhighlight %}

Após a conexão é possível receber dados com o método `gets` e enviar dados com o método `write`:

{% highlight ruby %}
conn.gets
{% endhighlight %}

{% highlight ruby %}
conn.write('Hello World')
{% endhighlight %}

### Ruby e MQTT

Iremos utilizar a lib [ruby-mqtt](https://github.com/njh/ruby-mqtt) para interagir com o **MQTT**. A conexão com o *broker* é bastante simples:

{% highlight ruby %}
mqtt = MQTT::Client.connect('localhost')
{% endhighlight %}

Enviando dados (deve-se passar o tópico e o valor):

{% highlight ruby %}
mqtt.publish('temperatura', 30)
{% endhighlight %}

Recebendo dados:

{% highlight ruby %}
mqtt.subscribe('#')
mqtt.get do |topic, value|
  puts "#{topic} = #{value}"
end
{% endhighlight %}

Neste caso estamos escutando todas as filas, e abaixo estamos "printando" na saída padrão o tópico e o valor.

### Arduino e Modelo de dados

Devemos definir um modelo de dados, para que possamos saber qual o tipo de dado está sendo enviado da placa, por exemplo: Temperatura, Luminosidade, Umidade, etc.

Neste tutorial optei por utilizar o formato *JSON* para enviar e receber dados. E agora vocês devem estar se perguntando, porque *JSON*? Não é complicado "parsear" o *JSON* na placa em *C*? A resposta é não. Existe uma excelente lib [ArduinoJson](https://github.com/bblanchon/ArduinoJson) que faz todo este trabalho.

Primeiramente incluímos a lib `ArduinoJson`, e vamos iniciar a comunicação serial com o `baud rate` com o valor `9600`:

{% highlight cpp %}
#include <ArduinoJson.h>

void setup()
{
  Serial.begin(9600);
}
{% endhighlight %}

Agora vamos enviar o valor do sensor de luminosidade, que está no pino 0 da porta analógica (A0):

{% highlight cpp %}
void loop()
{
  double luminosity = analogRead(A0);

  sendMessage("luminosity", luminosity);
}

void sendMessage(String topic, double value) {
  StaticJsonBuffer<200> jsonBuffer;

  JsonObject& message = jsonBuffer.createObject();
  message["topic"] = topic;
  message["value"] = value;

  message.printTo(Serial);
  Serial.println();
}
{% endhighlight %}

Temos a função `loop` da placa, que lê a todo momento o valor do sensor na porta analógica `A0` da placa, e temos também a funcão auxiliar `sendMessage` que transforma o dado em `JSON` e envia este dado para a porta `serial`.

Se considerarmos que a nossa luminosidade tem o valor de `30`, enviaremos o seguinte *JSON* para nossa *serial port*:

{% highlight json %}
{
  "topic": "luminosity",
  "value": "30"
}
{% endhighlight %}

### Recebendo o JSON da placa e enviando para o MQTT

O código abaixo tem um loop que recebe os dados do **Arduino**, "parseia" o `json` em um `hash` e publica o dado no respectivo tópico do **MQTT**.

{% highlight ruby %}
require 'json'
require 'mqtt'
require 'rubyserial'

class Publish
  attr_reader :serialport, :mqtt

  def initialize(serialport, mqtt)
    @serialport = serialport
    @mqtt = mqtt
  end

  # Recebe os dados da porta serial.
  def execute
    loop do
      parse_and_publish(serialport.gets)
    end
  end

  private

  # Transforma o json em hash.
  def parse_and_publish(message)
    data = JSON.parse(message)
    publish(data['topic'], data['value'])
  rescue
    nil
  end

  # Pública um dado em algum tópico do MQTT.
  def publish(topic, value)
    mqtt.publish(topic, value)
  end
end

serialport = Serial.new('/dev/tty.usbmodem1421', 9600)
mqtt = MQTT::Client.connect('localhost')

Publish.new(serialport, mqtt).execute
{% endhighlight %}

### Recebendo dados do MQTT e enviando para o Arduino

Agora vamos fazer o sentido contrário, vamos receber os dados do **MQTT** e vamos enviar para o **Arduino** via *serial*.

{% highlight ruby %}
class Receive
  attr_reader :serialport, :mqtt

  def initialize(serialport, mqtt)
    @serialport = serialport
    @mqtt = mqtt
  end

  # Escuta todos os tópicos do MQTT.
  def subscribe
    mqtt.subscribe('#')
    mqtt.get do |topic, value|
      write(topic, value)
    end
  end

  private

  # Escreve os dados na porta serial da placa.
  def write(topic, value)
    message = { topic: topic, value: value }.to_json
    serialport.write(message)
  end
end

serialport = Serial.new('/dev/tty.usbmodem1421', 9600)
mqtt = MQTT::Client.connect('localhost')

Receive.new(serialport, mqtt).subscribe
{% endhighlight %}

### Finalizando

Agora que sabemos enviar/receber dados de uma placa **Arduino** e ainda utilizar o **MQTT** para receber estes dados, podemos criar diversas aplicações que escutam os tópicos do **MQTT** e trabalhem estes dados de alguma forma.

Nos exemplos acima, utilizei pequenos pedaços de códigos para explicar tanto o envio, quanto o recebimento de dados. Tenho em meu github um projeto chamado [Ultron](https://github.com/sergioaugrod/ultron), onde se pode fazer tudo isso que foi explicado, com o uso bem simplicado, você só precisará baixar o projeto e instanciar a aplicação com as configurações de conexões do **MQTT** e da *serial port*.

Espero que eu tenha conseguido explicar um pouco dos conceitos básicos sobre comunicação de dados com Arduino e MQTT, e aguardo vocês no próximo post.

Obrigado.

### Referências

* <https://www.arduino.cc>
* <https://mosquitto.org>
* <https://github.com/hybridgroup/rubyserial>
* <https://github.com/njh/ruby-mqtt>
* <https://github.com/sergioaugrod/ultron>

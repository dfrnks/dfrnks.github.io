---
published: true
title: Comunicação RPI com o Arduino via RF 433 mhz 
layout: post
Examplos: https://guides.github.com/features/mastering-markdown/
---

Para poder se comunicar tanto arduino com o arduino via RF ou Raspberry com arduino ou vice-versa, até que é um pouquinho simples. É necessário algumas coisas como o Módulo de RF transmissor e receptor 433 MHZ, como os da imagens abaixo.

![pi3 gpio](/img/posts/rf_433mhz.jpg)

Um arduino e um RPI, obviámente. 

Para esse exemplo irei utilizar um RPI 3 Model B e ou Arduino UNO. Python e Arduino IDE

A comunicação se dará do RPI para o arduino

### Código RPI do transmissor
Aqui utilizaremos um lib externa para poder comunicar com o transmissor, [piVirtualWire](https://github.com/DzikuVx/piVirtualWire)
```python
# necessário importar a lib piVirtualWire
import piVirtualWire.piVirtualWire as piVirtualWire 
# Necessário importar a lib pigpio também, pois a piVirtualWire utiliza ela para se comunicar
import pigpio

pi = pigpio.pi()

# Primeiro parâmetro é a instancia do pigpio, segundo é a porta do GPIO que irá utilizar e o terceiro é a taxa de comunicação, necessário informar a mesma que o receptor
tx = piVirtualWire.tx(pi, 18, 2000)

while True:
    msg = raw_input('Digite uma mensagem: ') # Espera pela mensagem no terminal
    tx.put(msg) # Envia a mensagem
    tx.waitForReady() # Esperar enviar toda ela

tx.cancel()
pi.stop()
```

### Esquema RPI transmissor
A ligação é extremamente simples, pois o bichinho tem só 3 pernas. 

![transmissor_rf](/img/posts/2018-07-08/transmissor_rf.jpg)

Se a distancia for pequena, tipo alguns metros não é necessário antena para comunicar. 

Primeira coisa, ligue o GND (Terra) do transmissor ao pino Ground do RPI (Pino 6 por exemplo), após ligue  o pino do 3.3v fo RPI (Pino 1) ao pino VCC do trasmissor. 

Então é so ligar o pino DATA do transmissor ao pino do GPIO, no nosso caso ao Pino GPIO18 (Ou pino 12). 

Pronto, a ligação está feita. 

![esquema_rpi_transmissor_rf](/img/posts/2018-07-08/esquema_rpi_transmissor_rf.jpg)

Caso tenha duvidas das pinagens do RPI pode verificar [aqui](/2018/07/08/esquemas-portas-rpi.html){:target="_blank"}


### Código arduino do receptor

No arduino receberemos a mensagem e iremos escrever a mesma via comunicação serial, você poderia exibir num display por exemplo, ou talvez executar algum comando, como ligar um led, acionar um motor, etc.
Aqui para podermos nos comunicar será utilizar a lib externa [VirtualWire](http://www.airspayce.com/mikem/arduino/VirtualWire/VirtualWire-1.20.zip){:target="_blank"}
```c
#include <VirtualWire.h>

byte message[VW_MAX_MESSAGE_LEN];    // Armazena as mensagens recebidas
byte msgLength = VW_MAX_MESSAGE_LEN; // Armazena o tamanho das mensagens

void setup()
{
  Serial.begin(9600);   // Inicializa a comunicação serial informando a taxa de comunicação
  vw_set_rx_pin(5);     // Porta que vai comunicar com o receptor
  vw_setup(2000);       // Taxa de comunicação, tem que ser a mesma do RPI
  vw_rx_start();        // Inicializa o receptor
  
  Serial.println("Esperando o sinal..."); //Envia uma mensagem para a serial

  pinMode(LED_BUILTIN, OUTPUT); // Iremos acionar o led interno toda vez que o arduino receber alguma comunicação via RF
}

void acendeLed(){
    digitalWrite(LED_BUILTIN, HIGH);
    delay(50);
    digitalWrite(LED_BUILTIN, LOW);
    delay(50);
    digitalWrite(LED_BUILTIN, HIGH);
    delay(300);
    digitalWrite(LED_BUILTIN, LOW);
}

void loop()
{
  uint8_t message[VW_MAX_MESSAGE_LEN];      // Cria um array de caracteres tipo uint8_t
  uint8_t msgLength = VW_MAX_MESSAGE_LEN;

  if (vw_get_message(message, &msgLength)) // Essa função do VirtualWire verifica se foi recebido alguma mensagem, se sim ele set a mensagem na primeira variável passada por parâmetro.
  {
    acendeLed(); // Acende o led, com uma piscadela reconhecivel
    
    Serial.print("Recebido: "); //Joga para a serial dizendo que recebeu alguma coisa
    
    for (int i = 0; i < msgLength; i++)
    {
      Serial.write(message[i]); // Escreve caracter por caracter na serial
    }
    
    Serial.println(); // Quebra a linha
  }
}
```
Você pode encontrar mais funções para utilizar na documentação do VirtualWire [aqui](https://arduino-info.wikispaces.com/file/view/VirtualWire.pdf){:target="_blank"}

### Esquema arduino recptor

Aqui também a conexão é bem simples, o bichino do recpetor tem 4 pernas, mas só 3 funções, já que duas pernas tem a mesma função. 

![receptor_rf_2.jpg](/img/posts/2018-07-08/receptor_rf_2.jpg)
![receptor_rf_1.jpg](/img/posts/2018-07-08/receptor_rf_1.jpg)

O GND (Terra) ligue no GND do arduino, e o VCC ligue nos 5v (5 volts) do arduino e qualquer um dos dos DATA ligue no DP (Digital port) do Arduino, no nosso caso foi na porta 5. 

![esquema_arduino_receptor_rf.jpg](/img/posts/2018-07-08/esquema_arduino_receptor_rf.jpg)

Para ver a saida na serial, utilize o monitor serial do Arduino IDE. 


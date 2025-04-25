# Projeto IOT- RASPBERRY

O protocolo MQTT (**Transporte de Telemetria de Enfileiramento de Mensagens) ou (Message Queuing Telemetry Transport)** é um protocolo de mensagens baseados em pub/sub e o Broker é um servidor que permite a comunicação entre dispositivos sem eles precisarem conhecer um ao outro diretamente,  ele recebe dados de dispositivos que publicam informações e os encaminha para outros dispositivos ou sistemas que se inscreveram para receber esses dados em **tópicos** específicos. Então o Broker MQTT é um servidor capaz de receber mensagens de um publicador (pub) e entregar aos assinantes (sub) usando como base as assinaturas dos tópicos.

Neste projeto o objetivo é montar um servidor broker MQTT, afim de criar uma rede que irá se conectar a outras máquinas locais que não se conhecem, utilizando dos conhecimentos já adquiridos em aula. 

![image.png](attachment:f67e38d5-7fb2-4c10-b8ed-8acc8a7a9096:image.png)

## O que é pub/sub ?

É um sistema de mensagens amplamente muito utilizado sendo um sistema de comunicação assíncrona. Ele permite que diferentes partes de um sistema (ou sistemas separados) se comuniquem sem que precisem conhecer a identidade umas das outras.

- **Publishers (Publicadores):** São as partes que enviam mensagens sobre um determinado tópico ou evento. Eles não sabem quem ou quantos assinantes existem.
- **Subscribers (Assinantes):** São as partes que expressam interesse em receber mensagens sobre um ou mais tópicos específicos. Eles não precisam saber quem são os publicadores.

### Exemplo:

Notificações de redes sociais, como o Youtube por exemplo, quando o criador de conteúdo publica um vídeo e a pessoa se inscreveu no canal desse criador, permite que você possa receber esse vídeo.

---

## **Os materiais utilizados são:**

1. Raspberry Pi ( é o broker utilizando do protocolo mqtt para realizar a comunicação entre os equipamentos).
2. ESP32 (aqui é será um equipamento que irá se comunicar, sendo o sub)
3. Notebook ( aqui será um equipamento que irá se comunicar, sendo o pub)

---

## Passo a passo da construção do broker

1. Baixar a imagem ISO

- Baixar o app Raspberry Pi
- Ao baixar, configurar o ISO:
    - Definir o dispositivo, sendo o “No filtering”;
    - Em seguida, você define o sistema operacional(Operating system), sendo o “Raspberry Pi-(64 bits)”;
    - Após isso, selecione o SSD embutido no computador;
    - Feito isso, é preciso customizar o ISO, definindo o usuário e a sua senha;
    - Detalhe, não é necessário alterar outras informações além do Username e Password;
    - Clique em save;
    - Clique em baixar, espera pela instalação e verificação do ISO;
    - Após isso, coloque o SSD com o ISO no Raspberry Pi, para inicializar o sistema operacional.

*`O SSD deve ter no mínimo 16GB.`*

---

## Configuração do MQTT

Com o SSD inserido na Raspberry, iniciará o sistema operacional, aparecendo primeiramente a página inicial, apenas configure algumas coisa como data, hora e local, logo em seguida abra o terminal para a instalação do Mosquitto. 

1. Atualize os pacotes.

```powershell
sudo apt update
```

Atualiza pra versão atual.

```powershell
sudo apt upgrade  
```

1.  Para baixar o mosquitto, precisa colocar o seguinte comando:

```powershell
sudo apt-get install mosquitto mosquitto-clients
```

1. Execute o comando para iniciar o servidor:

```powershell
sudo systemctlstart mosquitto
```

1. Ativação e configuração do serviço. Para garantir que o serviço Mosquitto inicie automaticamente:

```bash
sudo systemctl enable mosquitto

```

1. Para permitir conexões de outros dispositivos da rede, abrimos o arquivo onde está as configurações do servidor:

```bash
sudo nano /etc/mosquitto/mosquitto.conf

```

1. Adicione as seguintes linhas ao final do arquivo:

```bash
listener 1883
allow_anonymous true

```

`listener 1883` → Específica a porta utilizada para executar as conexões de clientes MQTT.

`allow_anonymous true` → Habilita a comunicação de usuários anônimos, sem exigir usuário e senha.

1. Em seguida, reinicie o serviço:

```bash
sudo systemctl restart mosquitto
```

1. Para publicar uma mensagem, você pode usar a `mosquitto_pub`ferramenta de linha de comando que vem com o pacote Mosquitto. A sintaxe básica é:

```bash
mosquitto_pub -h<nome do host > -t<tópico > -m<mensagem >
```

- A `h`opção especifica o nome do host do broker MQTT.
- A `t`opção especifica o tópico no qual a mensagem deve ser publicada.
- A `m`opção especifica a mensagem a ser publicada.

1. Assinando um tópico

```
mosquitto_sub -h<nome do host > -t<tópico >
```

Assinar um tópico é igualmente fácil. Você pode usar a `mosquitto_sub`ferramenta de linha de comando, com a sintaxe básica sendo:

- A `h`opção especifica o nome do host do broker MQTT.
- A `t`opção especifica o tópico no qual você deseja se inscrever.

---

## Tópicos e Conexão com equipamento

Após concluirmos isso precisamos descobrir o endereço de ip.

Para isso vamos usar o seguinte comando:

```bash
hostname -I  # o i deve ser maiusculo obrigatoriamente
```

O comando devolverá o endereço de ip da máquina.

---

## Teste de comunicação

Agora podemos entrar no tópico e executar a função de pub e sub para isso iremos utilizar os seguintes comandos.

### Subscrição:

```bash
mosquitto_sub -h [localhost](http://localhost) -t "007/chat"
```

### Publicação:

```bash
mosquitto_pub -h [localhost](http://localhost) -t "007/chat" -m "Mensagem do broker"
```

1. Em um dos Raspberries (por exemplo, do Grupo 01), execute o comando de **assinatura** (`mosquitto_sub`).
2. Em outro dispositivo (por exemplo, do Grupo 02), execute o comando de **publicação** (`mosquitto_pub`).
3. A mensagem deverá aparecer no terminal do assinante.

---

## Realizando a comunicação com outro grupo

### Código para o Sub:

```powershell
#include <WiFi.h>
#include <PubSubClient.h>
const char* ssid = "iot";
const char* password = "iotsenai502";
const char* mqtt_server = "192.168.0.175";  // IP da Raspberry Pi
WiFiClient espClient;
PubSubClient client(espClient);
#define LED_PIN 4
void setup_wifi() {
delay(10);
Serial.println("Conectando ao WiFi...");
WiFi.begin(ssid, password);
while (WiFi.status() != WL_CONNECTED) {
delay(500);
Serial.print(".");
}
Serial.println("\nConectado ao WiFi");
}
void callback(char* topic, byte* payload, unsigned int length) {
Serial.print("Mensagem recebida em [");
Serial.print(topic);
Serial.print("]: ");
String mensagem;
for (int i = 0; i < length; i++) {
mensagem += (char)payload[i];
}
Serial.println(mensagem);
if (mensagem == "ligar_led") {
digitalWrite(LED_PIN, HIGH);
Serial.println("LED LIGADO");
delay(1000);  // Acende por 1 segundo
digitalWrite(LED_PIN, LOW);
Serial.println("LED DESLIGADO");
}
}
void reconnect() {
while (!client.connected()) {
Serial.print("Tentando conectar ao MQTT...");
if (client.connect("ESP32Sub")) {
Serial.println("Conectado ao broker MQTT");
client.subscribe("grupo7/chat");
} else {
Serial.print("Falhou, rc=");
Serial.print(client.state());
Serial.println(" tentando novamente em 1 segundo");
delay(1000);
}
}
}
void setup() {
pinMode(LED_PIN, OUTPUT);
Serial.begin(115200);
setup_wifi();
client.setServer(mqtt_server, 1883);
client.setCallback(callback);
}
void loop() {
if (!client.connected()) {
reconnect();
}
client.loop();
}
```

### Código para Pub:

```powershell
#include <WiFi.h>
#include <PubSubClient.h>
// Wi-Fi
const char* ssid = "iot";
const char* password = "iotsenai502";
// MQTT Broker
const char* mqtt_server = "192.168.0.175";  // IP da Raspberry Pi
WiFiClient espClient;
PubSubClient client(espClient);
void setup_wifi() {
delay(10);
Serial.println("Conectando ao WiFi...");
WiFi.begin(ssid, password);
while (WiFi.status() != WL_CONNECTED) {
delay(500);
Serial.print(".");
}
Serial.println("\nConectado ao WiFi");
}
void reconnect() {
while (!client.connected()) {
Serial.print("Tentando conectar ao MQTT...");
if (client.connect("ESP32Pub")) {
Serial.println("Conectado ao broker MQTT");
} else {
Serial.print("Falhou, rc=");
Serial.print(client.state());
Serial.println(" tentando novamente em 1 segundo");
delay(1000);
}
}
}
void setup() {
Serial.begin(115200);
setup_wifi();
client.setServer(mqtt_server, 1883);
}
void loop() {
if (!client.connected()) {
reconnect();
}
client.loop();
static unsigned long lastMsg = 0;
if (millis() - lastMsg > 5000) {
lastMsg = millis();
String msg = "ligar_led"; // Comando a ser enviado
client.publish("grupo7/chat", msg.c_str());
Serial.println("Mensagem publicada: ligar_led");
}
}
```

---

![ecadbeae-f648-485b-acfe-6d5f45e32344.jpg](attachment:4e1afa95-4463-4f12-aa5b-d869eaf1c50f:ecadbeae-f648-485b-acfe-6d5f45e32344.jpg)

                                                *Imagem da Montagem Arduino + ESP*

![690069b9-f671-48b1-b68b-0f6a742c8c3c.jpg](attachment:8a60e80b-2888-4ce0-8692-ae0ba568387b:690069b9-f671-48b1-b68b-0f6a742c8c3c.jpg)

                                           *Resposta através no Arduino IDE (Plataforma):*

---

## REFERÊNCIAS:
https://github.com/MrFMach/Esp32-MQTT-PubSub/blob/main/media/Image.jpeg
https://www.emqx.com/en/blog/esp32-connects-to-the-free-public-mqtt-broker#introduction
https://forums.raspberrypi.com/viewtopic.php?t=260851
https://randomnerdtutorials.com/how-to-install-mosquitto-broker-on-raspberry-pi/
https://randomnerdtutorials.com/esp32-mqtt-publish-subscribe-arduino-ide/
